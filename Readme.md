# 数据结构与算法  课程大作业：WFC算法实现 

## 摘要

本项目是2019年春季数据结构与算法课程大作业，由孟凡强，孙亮，蔡斌共同完成。在这个项目中，我们使用**Python**实现了波函数塌缩算法(***WaveFunctionCollapse***, 以下简称***WFC***),  并将其实现为可供扩展调用的模块。在本文中我们将按照课程要求对我们的项目进行介绍，主要包括：


- 需求分析
- 算法实现
- 示例
- 组员分工
- 总结与评价




## 需求分析

如果我们需要根据一张小小的图来生成一张具有相同特征的大大的图，应该怎么做呢？直接把输入的图像排起来似乎就可以满足这个要求，但这样的结果未免有些平庸。实际上，这样的需求并不罕见。比如在游戏中随机生成一张城市的地图。这时，既有一定的随机性，又能够保持原来图样风格的输出才是我们更感兴趣的，就像下方图示这样：

<img src="doc\example1.png" style="zoom:18%" />


怎么来获得与输入风格相似的输出呢？当然，我们或许可以采用机器学习来完成这件事，比如识别输入图中的特点，然后再以此生成输出。但在目前的情况下，输入的图像的大小十分有限，缺乏足够的样本来训练我们的代码。那么非机器学习的方式要如何呈现呢？我们首先可以试着把在输入中出现的像素块记录下来，然后按它们出现的频次来生成输出：

<img src="doc\fail1.png" style="zoom:20%" />

显然，这样的结果过于混乱，因为我们并没有限制像素块之间能否互相连接。为了更多地记录输入中的信息，我们应该以输入中出现的小图案为单位来分析，而不仅仅只是像素块。同时，还应该记录图案之间相互连接的规则。这样就有可能使我们的输出在局域上相似于输入。再调控各种图案出现的概率使其在整体上的分布与输入近乎一致。下一个问题则是应该按怎样的顺序来分别确定每一个格点上的图样。不难想象，如果每一次都按照相同的顺序来选择图样的话（比如从左上到右下），很容易就会使得后面的格点没有选择的余地。为了不让程序这么脆弱，应该每次都去寻找已经比较确定的格点，或者说，选项比较少的格点。事实上，这就是所谓的波函数塌缩算法(***WFC***)的主要思想。



### *WFC* 算法简介

***WaveFunctionCollapse*** 算法是由Maxim Gumin等人于2016年提出的，是一种非回溯的贪心搜索算法。最早呈现于一个`GitHub`项目[WFC](<https://github.com/mxgmn/WaveFunctionCollapse>),  是采用 **C#** 语言进行实现的。算法的构想借鉴于量子力学中的波函数塌缩过程，用于随机生成与输入位图风格相似的位图。随着项目的发布，人们很快就用了各种编程语言实现了 ***WFC*** 算法，并开发了一些简单有趣的[小游戏](<<https://marian42.itch.io/wfc>>)作为应用，也有人详细讨论了其作为一种[解决约束问题的算法](<https://adamsmith.as/papers/wfc_is_constraint_solving_in_the_wild.pdf>)的特性。

那么什么是所谓的“风格相似”的位图呢？Gumin 将其归结为两个条件：

>1. 输出中每一种大小为$N \times N$ 的图案(pattern)都应至少在输入中出现一次。
>2. (弱条件)  输入中$N\times N$ 图案的分布应类似于足够大量输出上的$N\times N$模式的分布。 换言之，输出中某个图案出现的概率应接近输入中该模式出现的频次。
>(强条件)  输出中某个图案出现的概率的极限应收敛于输入中该模式出现的频次。

由此可见，$N\times N$ 大小的图案将是算法采集和分析输入位图局部特征的最小单元。也就是说，$N$ 将决定算法的输出在多大程度上模仿输入的局部特征。可以想象的是，$N$ 太小时，输出受到的约束很少，结果将比较混乱，难以完整地再现输入的特征；$N$ 太大时，输出将大块大块的复制输入而缺少随机性。所以参数 $N$ 的选择对于算法的运行十分重要，之后我们将对此再次讨论。另一个需要指出的是，***WFC*** 算法只能满足第二个条件的弱化版本，强条件的需要采用其他方式比如马尔柯夫链来满足。

下图(图片取自网络)是一个简单的样例，可以看到输出中的各种模式与输入之间的关系。

<img src="doc\wfc-patterns.png" style="zoom:95%" />

类比于量子力学中体系波函数和叠加态的概念，***WFC*** 算法首先将待输出的位图初始化一个未被观测过的波函数。其每一个格点的像素值都是输入位图中的像素值的叠加（例如，若输入是黑色块与白色块，那么尚未观测的像素点就是灰色的）。不同于量子力学的是，这里的叠加系数都是实数而非一般的复数。接下来，我们进入观测-传播循环：

- 挑选信息熵最小的未观测格点，将这个格点的状态随机塌缩为其状态空间中的某一个态。随机塌缩过程的权重则依赖于相应的图案在输入中出现的频次。
- 格点塌缩后，根据图案之间相邻近的约束，将塌缩的影响向四周的格点传播。

不断循环观测的过程直至所有格点都完成塌缩之后，整个体系的状态就被确定下来了。当然，算法运行过程中有可能出现某个格点已经没有了可取的状态，此时接下来的过程将无法继续进行。考虑到生成一个满足前述条件的非平凡位图是一个NP问题，并不存在既高效又稳定的方法，故Gumin的解决方案是放弃之前的结果，让程序重新开始运行直至出现合适的结果。有趣的是，在实际的运用过程中，极少出现这样崩溃的情况。

在整个观测过程中，每次选择信息熵最小的点进行塌缩是一个十分精妙的构想，这使得每一次塌缩都从最接近确定的格点开始，而最终获得的输出竟能乱中有序。算法的提出者Gumin在其程序的介绍文件中写道：

>  “ I noticed that when humans draw something they often follow the minimal entropy heuristic themselves. That’s why the algorithm is so enjoyable to watch. ” 

正是这样的选择使得尽管 ***WFC*** 算法的输出不是简单机械地堆叠输入以获得一张更大的图样，而是仿佛有意识般地绘制，以至于能还原出输入图像的风格乃至神韵。



### 我们想做什么？

虽然`GItHub`上已有了许多实现 ***WFC*** 算法的项目，其中也包括采用 **Python** 语言的。但我们小组仍希望能够自己实现这个有趣好玩的算法，并希望加入一个简单的图形界面以方便使用。同时，虽然算法的初衷是为了求解开放性的、非唯一解的约束问题，我们还想将这个算法用在其他解唯一的约束问题中（比如数独问题）。此时，回溯就变得十分重要了。简言之，我们想做的事情有：

- 实现 ***WFC*** 算法，将之一般化为一个可以求解二维约束问题的模块
- 加入回溯操作
- 编写图形界面

下面，我们就来分析本项目的具体细节。



## 程序实现

基于之前的分析，我们的项目自然地分为两个部分，即核心算法的实现和图形界面的编写。其中图形界面的编写比较常规，主要是对官方模块`tkinter` 的运用。程序中的主要运算过程就是扫描输入和塌缩-传播循环，其大致思想如前文所述，在代码文件中 *wfc.py* 也有详细的注释。下面简要概述一下程序采用的数据结构。

首先，我们需要提取输入中的信息，对于图片来说，就是各种小的图案、图案出现的权重和图案间的相邻关系。其次，由于***WFC*** 算法是基于对格点依熵的次序塌缩工作的。故每一个格点都应该要存储其目前可取的状态及相应的权，同时最好还要有相应于这个态空间的信息熵（[香农熵](<https://en.wikipedia.org/wiki/Entropy_(information_theory)>)）. 对于某个格点而言，第 $i$ 个可取的状态的权为 $\omega_i$ ,则该格点状态空间的熵就定义为
$$
S=-\sum_i \left(p_i \ln p_i\right)=-\sum_i \frac{w_i}{\sum w_i} \ln{\frac{w_i}{\sum w_i}}
$$

构造一个格点类来记录这些数据将是一个方便的选择。最后，需要用一个包含所有格点的二维矩阵来作为体系的整体波函数。为了方便，我们把从输入中提取的信息作为波函数的系数，和体系波函数一起用一个波函数类来存放。另外，为了将程序一般化，可以把扫描输入得到的模式(pattern)单独存放并编号，在后续的运算中只处理其编号，在输出时再还原为相应的模式。这样也可以使算法结构更清楚。于是，算法中采用的主要数据结构可以总结如下

- Knot 类：
    space： 描述该点的状态空间，是一个以可选值为key，以其频率为value的字典；
    entroy：该点的香农熵，算法见上文。
    
- Wave类：

  - 提取信息部分：
    
    patterns：记录所有出现过的图案，是一个以整数为key，以对应图案为value的字典；
    weights：记录每种图案出项频率，是一个以图案对应的整数为下标，该图案出现频率为值的list；
    rules: 记录扫描得到的规则，是一个以图案对应的编号为第一坐标，该图案周围的某一方向为第二坐标（上下左右分别对应0，1，2，3），该图案在该方向上可以相邻的图案组成的集合为值的二维list；
    options: 记录各种功能选项的字典。
    
  - 波函数部分：
    
    wait_to_collapse：记录还未坍缩的点的集合，塌缩过程中不断从中选取熵最小的点。集合为空作为坍缩完成的判据；
    Stack：一个用来记录坍缩过程的栈，回溯时从其中读取上一步的信息；
    wave：体系的整体波函数，是一个由格点类对象组成的二维矩阵。



## 项目文件说明

文件夹中一共包括三个程序文件：

- wfc.py  波函数塌缩模块
- ImageWFC.py     处理图像的图形界面
- SudokuWFC.py  单独编写的用WFC算法计算数独的程序

其中 *wfc.py* 是可拓展的模块，输入其他2维数组即可用以处理不同的问题。在模块内部便是用一个字符矩阵作为示例的。*ImageWFC.py* 是调用了 *wfc.py* 以及 `matplotlib`、`tkinter` 等模块编写的图形界面程序。其操作简单而完善，可以直接使用。运行之后直接读取图片，再分别设置识别模式的大小`N`、输出图片的宽度和高度，再点击 `WFC!` 即可。下方的选项中可以提供各种功能，列举如下：

| 选项名          | 功能                                                         |
| :-------------- | ------------------------------------------------------------ |
| All Rule        | 匹配所有图案之间的邻近规则，这将可能出现输入中不存在的连接方式 |
| Surveil         | 监控塌缩过程，若不选则直接输出结果                           |
| Periodic Input  | 设置输入周期化，即将输入分别在两个维度上首尾相连             |
| Periodic Output | 设置输出周期化，即要求输出分别在两个维度上首尾相连           |
| Rotate          | 允许输入分别进行90°、180°、270°旋转                          |
| Reflect         | 允许输入分别进行水平反射、垂直反射、中心反演                 |

而 *SudokuWFC.py* 则是计算数独的程序，因为是作为展示用来在具有较强约束下的 ***WFC*** 的小样编写的，故只支持在程序内部输入，并未编写用户接口。



## 组员分工

本项目是由小组三人共同讨论、选题和一起完成的。小组三人的姓名学号分别为：

孟凡强：1700011323；孙亮：1700011430；蔡斌：1500012145。

在项目编写初期，由孟凡强负责简单实现波函数塌缩过程，由孙亮负责实现对输入的扫描以及与图像的接口。基本运行成功后开始优化和调试代码，其中孟凡强优化了塌缩传播过程的运算，并添加了对称操作和周期性边界操作；孙亮单独编写了用有回溯的 ***WFC*** 算法求解数独的程序，在调试成功后为主程序添加了回溯操作。在项目编写的后期，蔡斌初步编写了用户输入输出接口，最终由孙亮和孟凡强共同设计并完善了图形界面。另外，本文档由孟凡强完成。

整个项目中孟凡强和孙亮共同完成了95%以上的工作，蔡斌在项目编写过程中参与了讨论，提出了一些建议，并做了用户界面编写的尝试。



## 总结

在这次项目的编写中，我们增进了对 ***Python*** 编程能力，加深了对数据结构的认识。事实上，在实现波函数塌缩算法的过程中，为了能使得代码更简洁、高效，我们分析并尝试过几种不同的结构来存储从输入中采集的信息，并最终选择现在的实现方式。在使用处理图像的库时，我们曾采用过`PIL`来编写，但由于其不支持实时刷新输出，不能方便的展示塌缩过程，我们最终采用`matplotlib`来处理图像。另外，为了方便小组协作，我们采用了 `Git` 进行源代码管理，并将全部代码同步于 `GitHub` [网站](<https://github.com/PhyM73/WaveFunctionCollapse_DASC_project>)上。

不无遗憾的是，我们的项目尚有一些功能没有实现。首先，如前所述，***WFC*** 算法本身就无法使得输出中图案分布的频率与输入完全一致。其次，虽然我们希望可以将算法延拓为一般的情形，但暂时还只为其拓展了处理图像的方法。最后，对于算法的运算速度，我们虽然不是很满意，但暂时也难以再为其优化和加速了。

最后，非常感谢老师和助教一个学期的辛勤付出，祝生活愉快！