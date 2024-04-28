# UC Berkeley EECS 是如何培养计算机学生的

加州大学伯克利分校电子工程和计算机科学系(EECS)是世界知名的院系，计算机领域在[2020 USNews 排名第一](https://www.usnews.com/best-graduate-schools/top-engineering-schools/computer-engineering-rankings)。EECS 的使命是`教育`、`创新`和`服务社会`。自创建以来，为社会培养了大批人才，诞生 7 位图灵奖得主。EECS 认为，其成功的背后，是**强大的合作传统**、**与工业界紧密联系**和**互助的文化**。

像这样的顶尖大学，本文无法面面俱到，而是从计算机专业培养入手，结合`课程`、`研究中心实验室`等角度总结其培养学生的特点，为 CS 领域或想转行 CS 的小伙伴提供可借鉴的方法和参考。

## 课程

### 命名约定

课程采用`编号+课程名`表示，比如很出名的`CS 61A: The Structure & Interpretation of Computer Programs`，61A 是课程的编号，字母 A 表示系列（下面会介绍），后面跟着名字。关于编号有如下约定：

- 0xy (e.g., 16, 61, 70) - lower-division courses，默认 0 是省略的，表示核心课程
- 1xy (e.g. 105) - upper-division courses，高阶课程
- 15x - Computer Architecture，计算机体系结构类课程
- 16x - Software，软件类课程
- 17x - CS Theory，计算机理论类课程
- 18x - CS Applications，计算机应用类课程
- 11x - Electromagnetics/Optics，电磁学或光学类课程
- 12x - Information Processing and Communication，信息处理和通信类课程
- 13x - Physical Electronics，物理电子类课程
- 14x - Integrated Circuits and Embedded Systems，集成电路和嵌入式系统类课程
- 19x - Special Topics, Directed Studies，特殊主题课程，指导学习

不同编号序列按照如下约定：

- 1xy，1 开头为本科课程
- 2xyA is the mezzanine-level course room-shared with 1xy，和 1xy 课程共享教室，但内容层次更高
- 2xyB，研究生课程
- 2xyC, 2xyD...，2xy 后面跟 C、D...表示后续课程

从上面可以总结出，0xy 是核心课程，1xy 是本科课程，2xy 是研究生课程。总体分为 EE 和 CS 两大类，层次分明。

### 0xy 核心课程

核心课程：

- CS61A 计算机程序的构造和解释
- CS61B 数据结构
- CS61C 计算机结构
- CS70 离散数学和概率论

0xy 系列课程重点培养学生的计算机基础、计算机科学素养和数学能力。

### 1xy 本科生课程

本科生 CS 课程：

- CS 161 计算机安全
- CS 162 操作系统与系统编程
- CS 164 编程语言与编译器
- CS 169 软件工程
- CS 170 高效算法与难题
- CS 172 可计算性与复杂性
- CS 174 组合数学与离散概率
- CS 182 设计、可视化和理解深度神经网络
- CS 186 数据库系统概论
- CS 188 人工智能导论
- CS 189 机器学习导论
- CS C191 量子信息科学与技术

面向本科生的 1xy 系列课程和 0xy 系列核心课程有明显区别，0xy 是核心基础课，1xy 则针对 CS 不同方向开课。

### 2xy 研究生课程

研究生 CS 课程：

- CS 252 研究生计算机体系结构
- CS 261 计算机系统安全
- CS 261A 因特网与网络安全
- CS 262A 计算机系统高级主题
- CS 262B 计算机系统高级主题，对 262A 的延续，高级话题一门课讲不完...
- CS 263 编程语言设计
- CS 264 编程语言实现
- CS 265 编译器优化与代码生成
- CS 268 计算机网络
- CS 270 组合算法与数据结构
- CS 285 Deep Reinforcement Learning, Decision Making, and Control
- CS 286A 数据库系统导论
- CS 286B 数据库系统实现
- CS 288 自然语言处理
- CS 289A 机器学习导论
- CS 294-162 机器学习系统
- CS 298-015 BAIR First-year Proseminar
- CS 299 个人研究

2xy 系列是面向研究生的课程，部分课程名和本科课程相同，为了区别会在前面加上`Graduate`（研究生）或`Advanced Topics`（高级主题），比如`CS 252. Graduate Computer Architecture`。为了培养研究生论文阅读、交流讨论等能力，还开设了研讨会课程（CS 298-015 BAIR First-year Proseminar）。

### 课程总结

下图是笔者总结的 CS 课程架构图，可以看到特点鲜明：分类合理、层次分明、层层递进。红色部分是核心课程，也是其它课程的基础，然后将课程分为软件、硬件、理论、应用等方向。有些课程本科和研究生阶段都会开设，但研究生课程偏向高级主题、注重研究。

![ucb cs课程体系](https://raw.githubusercontent.com/alwqx/picx-images-hosting/master/blog/2019/07/ucb_eecs_courses.jpg)

EECS 信息化程度很高，上文提到的课程都有对应的官网资源，读者可以浏览[EECS Course WEB Sites](http://www-inst.eecs.berkeley.edu/classes-eecs.html)选择自己感兴趣的课程。

## 培养方案

### 本科

伯克利是公立大学，招收的学生数量比哈佛、斯坦福要多很多，因此对于本科生来说有三个特点：宽进严出、竞争激烈和丰富多彩。在学位上，CS 方向为本科生主要提供两个选择：

- 由文理学院（Letters & Science）提供的 CS 专业项目，毕业后授予 Bachelor of Arts
- 由工学院（College of Engineering）提供的 EECS 专业项目，毕业后授予 Bachelor of Science

两个学位 CS 课程内容是一样的，不同点在于选修课上，文理学院提供更丰富的课程选择，工学院的选修课侧重 EECS 领域。想修双学位可以选择前者，希望专注于计算机、STEAM 选择后者，详情参考[EECS/CS Program Comparison Chart](https://eecs.berkeley.edu/academics/undergraduate/eecs-cs-comparison-chart)。

### 研究生

伯克利的研究生有两种，面向工业(Industry-Oriented Degree Programs)和面向研究(Research-Oriented Degree Programs)，类似国内的专业/工程硕士和学术硕士。

专业硕士有两个培养方案：

1. [Master of Engineering(M.Eng.)](https://eecs.berkeley.edu/academics/graduate/industry-programs/meng)：工程硕士，教授毕业后进入工业界所需的技能，重点培养学生的技术基础和领导力。主要计算机相关课程是[Data Science and Systems](https://eecs.berkeley.edu/node/370)和[Visual Computing and Computer Graphics](https://eecs.berkeley.edu/node/389)，学制为 1 年。
2. [5th Year M.S. Program](https://eecs.berkeley.edu/academics/graduate/industry-programs/5yrms)：和 M.Eng.类似，不过该项目只面向 UC Berkeley 自己的 CS 本科生，和国内的本硕连读类似。

研究型硕士有三个培养方案：

1. [Master of Science](https://eecs.berkeley.edu/academics/graduate/research-programs)：学制为 2 年，是比较小的研究型项目，读完后可以申请博士或者进入工业界做研发。
2. [Doctor of Philosophy](https://eecs.berkeley.edu/academics/graduate/research-programs)：博士项目，培养学术人才，学制本科生要读 5-6 年，硕士读要 3-5 年。
3. [Both (M.S./Ph.D.)](https://eecs.berkeley.edu/academics/graduate/research-programs)：硕博连读，学制 5-6 年。

研究型项目着重培养学生的**研究能力**和**教学能力**，页面[Graduate Research Program Admissions](https://eecs.berkeley.edu/academics/graduate/research-programs/admissions)详细列出了研究型 CS(Computer Science)专业主要研究方向：

- [Artificial Intelligence (AI)](https://www2.eecs.berkeley.edu/Research/Areas/AI/)
- [Operating Systems & Networking (OSNT)](https://www2.eecs.berkeley.edu/Research/Areas/OSNT/)
- [Database Management Systems (DBMS)](https://www2.eecs.berkeley.edu/Research/Areas/DBMS/)
- [Graphics (GR)](https://www2.eecs.berkeley.edu/Research/Areas/GR/)
- [Human-Computer Interaction (HCI)](https://www2.eecs.berkeley.edu/Research/Areas/HCI/)
- [Programming Systems (PS)](https://www2.eecs.berkeley.edu/Research/Areas/PS/)
- [Scientific Computing (SCI)](https://www2.eecs.berkeley.edu/Research/Areas/SCI/)
- [Security (SEC)](https://www2.eecs.berkeley.edu/Research/Areas/SEC/)
- [Theory (THY)](https://www2.eecs.berkeley.edu/Research/Areas/THY/)
- [Education (EDUC)](https://www2.eecs.berkeley.edu/Research/Areas/EDUC/)

每个研究领域都有详细的介绍，并且列出了主要研究者、开设的课程等，方便学生了解和选择。

## 硬件条件

### 地理位置

![](https://raw.githubusercontent.com/alwqx/picx-images-hosting/master/blog/2019/07/ucb_map.png)

UCB 地理位置优越，地处湾区（Bay Aarea），附近比较出名的城市还有旧金山，奥克兰，硅谷等。硅谷大量的创业公司和科技公司为学生实习提供的便利。也为伯克利与工业界紧密合作提供了条件。

### 图书馆

伯克利图书馆系统包括三个主要图书馆、18 个学科专业图书馆和 11 个具有特殊藏书的附属图书馆，藏书超过 1000 万册，北美地区排名第四。在校园里，平均每步行 5 分钟，都会遇到一个图书馆。

### 研究中心和实验室

由于 EECS 是两个体系，所以研究中心和实验室大方向上包括 EE 方向和 CS 方向，这里主要讲 CS 方向，说两个业界出名的实验室。

[伯克利人工智能研究实验室（BAIR）](http://bair.berkeley.edu/)，开发[Caffe](http://caffe.berkeleyvision.org/)和[PyTorch](https://pytorch.org/)的大神贾扬清就来自这个实验室。该研究中心包含约 30 名教师和超过 200 名研究生、博士后。研究领域包括`计算机视觉`、`机器学习`、`自然语言处理`、`规划`和`机器人`。该中心开发了很多业界出名的工具，包括大数据处理框架 Spark 中的[GraphX](http://spark.apache.org/graphx/)、[CycleGAN](https://junyanz.github.io/CycleGAN/)等，详细页面见[BAIR SOFTWARE](https://bair.berkeley.edu/software.html)。

[RISELab](https://rise.cs.berkeley.edu/)，该实验室前身是[AMPLab](https://amplab.cs.berkeley.edu/)，两者都是为期 5 年的针对特定主题的实验室。AMPLab 研究周期为 2011-2016，为业界贡献了[Mesos](http://mesos.apache.org/)集群管理框架、[Spark](http://spark.apache.org/)大数据引擎、[Alluxio](https://github.com/Alluxio/alluxio)虚拟分布式存储等知名框架，**为行业作出卓越贡献**。新的 5 年研究项目由 RISELab 承担，RISELab 实验室研究领域聚焦在**Real-time Intelligence with Secure Explainable decisions**，即开发**实时、智能、安全、可解释的决策系统**，围绕这一理念，RISELab 已经研究开发了[Ray](https://github.com/ray-project/ray)、[Clipper](https://github.com/ucbrise/clipper)、[confluo](https://github.com/ucbrise/confluo)等软件，并开源的。Ray 已经在很多公司开始使用，国内知名科技金融公司蚂蚁金服就在使用。

伯克利 EECS 的研究中心非常多，这里只列举两个，详细信息请浏览[EECS-Research Centers and Labs](https://www2.eecs.berkeley.edu/Research/Areas/Centers/)。从这两个实验室可以一窥伯克利 CS 研究生的培养方式：学生会在导师所在实验室做研究，和工业界保持紧密的联系与合作。好的软件会和业界分享，在 GitHub 上开放源代码，与各界共同开发。

## 总结

在本科阶段，伯克利 EECS 很重视学生的基础知识，课程理论与实践相结合，课程的 Project 有难度需要编写很多代码，以培养学生的计算机科学思维为主，编程能力为辅。研究生阶段，重点培养学生的研究能力、教学能力，学生通过参加学术会议、开源项目等和学术界、工业界保持紧密的联系。

伯克利对于学生的培养方案是值得我们借鉴和学习的，但因人而异，以笔者个人经验来说，给出以下不成熟的建议：

0. 在 CS 领域，**计算机基础知识至关重要**！
1. 对于想转专业到计算机的本科学生：
   1. 可以学习 CS 的核心课程 CS 61A 61B 打基础
   2. 然后学习一门语言课（推荐 Python，适合初学者）
   3. 再学习 OS、Network 等课程
   4. 最后要**重视实践**，多编写代码和参与开源项目
   5. 接着去找实习在公司中锻炼自己，最后应该可以找到一个满意的工作
2. 本身就是 CS 专业的本科生：CS 专业应该具备相关知识了，这时可以根据自己感兴趣的 CS 课程
3. 工作 1-3 年的程序员：工作和实际编程中可以发现自己的不足，针对不足选择相关课程重点加强
4. 其它情况超出笔者的能力和经验了，可以结合本文自行决定

## References

- [伯克利加州大学计算机专业课程简介](https://www.zhihu.com/question/23372616)
- [转专业在 Berkeley 上的 CS 课](https://www.1point3acres.com/bbs/thread-345582-1-1.html)
- [加州伯克利大学计算机系是如何培养计算机人才的？](http://www.sohu.com/a/308174076_453160)
- [在加州大学伯克利分校（UC Berkeley）学习计算机是怎样的体验？](https://www.zhihu.com/question/41533899)
- [2020 最新世界大学排名](https://www.forwardpathway.com/worldranking)
- [《大学之路》 吴军](https://book.douban.com/subject/27199584/)
