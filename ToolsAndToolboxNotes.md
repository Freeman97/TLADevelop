- [TLA+ Tools and Toolbox学习笔记](#tla-tools-and-toolbox学习笔记)
  - [资料收集](#资料收集)
  - [TLC是怎么运行的?](#tlc是怎么运行的)
    - [使用Toolbox运行TLC](#使用toolbox运行tlc)
    - [单独运行TLC](#单独运行tlc)
  - [在Distributed/Cloud模式下运行TLC](#在distributedcloud模式下运行tlc)
    - [Ad Hoc模式下的TLC](#ad-hoc模式下的tlc)
      - [运行Master](#运行master)
      - [运行Slave](#运行slave)
      - [Distributed模式/Ad Hoc模式下的局限性](#distributed模式ad-hoc模式下的局限性)
    - [Cloud模式下的TLC](#cloud模式下的tlc)
      - [用法](#用法)
  - [Model Checking TLA+ Specfications](#model-checking-tla-specfications)
    - [Introduction](#introduction)
    - [TLA+](#tla)
    - [Checking Models of a TLA+ Specification](#checking-models-of-a-tla-specification)
    - [How TLC Works](#how-tlc-works)
    - [Representing States](#representing-states)
    - [Using Disk](#using-disk)
  - [The TLA+ Toolbox](#the-tla-toolbox)
    - [Toolbox Features](#toolbox-features)
      - [Model](#model)
      - [CloudTLC](#cloudtlc)
      - [Results](#results)
      - [Profiler](#profiler)
      - [Trace Explorer](#trace-explorer)
    - [Toolbox Architecture](#toolbox-architecture)
      - [Toolbox to Back-end](#toolbox-to-back-end)
      - [Back-end to Toolbox](#back-end-to-toolbox)
      - [CloudTLC Back-end](#cloudtlc-back-end)
      - [Testing](#testing)
    - [Conclusion](#conclusion)
    - [Future Work](#future-work)
  - [个人总结](#个人总结)
    - [TLC Runtime API](#tlc-runtime-api)
      - [需要增强的功能](#需要增强的功能)


# TLA+ Tools and Toolbox学习笔记
## 资料收集
* 文档链接大概是 https://tla.msr-inria.inria.fr/tlatoolbox/doc/ 不过文档似乎并未完成。
* 如何单独运行（不通过Toolbox）TLA+ Tools：http://lamport.azurewebsites.net/tla/standalone-tools.html?back-link=tools.html
* 一本全面了解TLA+及其相关工具（除了PlusCal translator和TLAPS）的书：Specifying Systems https://github.com/jackfoxy/SpecifyingSystemsWithContents/blob/master/Specifying%20Systems%2C%20TLA%2B.pdf 书中关于language和tool的描述和最新版本有不同。具体的不同在 http://lamport.azurewebsites.net/tla/current-tools.pdf 中有描述。language层面的不同则在 http://lamport.azurewebsites.net/tla/tla2.html 中有描述。
* TLC的命令行参数：http://lamport.azurewebsites.net/tla/tlc-options.html?back-link=tools.html
* TLC的主类：tlaplus/tlatools/org.lamport.tlatools/src/tlc2/TLC.java
* 一篇关于Toolbox的论文：The TLA+ Toolbox https://arxiv.org/abs/1912.10633

## TLC是怎么运行的?
### 使用Toolbox运行TLC
* TLA+ Toolbox在它根据模型构建的module`MC`上运行TLC。当对model进行验证时（点击run或validate时）Toolbox会对module文件`MC.tla`和TLC配置文件`MC.cfg`进行写入。如果spec的名字是`SpecName`，根据这个spec建立的model名字是`ModName`，这两个文件会写在目录`SpecName.toolbox/ModName`中。
  * 自己跑过的demo里看了一下，`MC.cfg`大概长这个样子，大概是用配置文件的方式记录了Model Overview选项卡中的一些model的配置信息
    ```
        \* CONSTANT definitions
        CONSTANT
        RM <- const_16075082780244000
        \* INIT definition
        INIT
        TCInit
        \* NEXT definition
        NEXT
        TCNext
        \* INVARIANT definition
        INVARIANT
        TCTypeOK
        \* Generated on Wed Dec 09 18:04:38 GMT+08:00 2020
    ```
  * 但是怎么编写这个配置文件似乎没有找到文档
* TLC Console View：TLC运行的标准输出。
  * 几乎包括了Model Checking Results里的所有必要信息
    ```
        TLC2 Version 2.15 of Day Month 20?? (rev: eb3ff99)
        Running breadth-first search Model-Checking with fp 4 and seed -6447362117639996479 with 2 workers on 4 cores       with 1205MB heap and 2711MB offheap memory [pid: 17976] (Windows 10 10.0 amd64, AdoptOpenJDK 14.0.1 x86_64,     OffHeapDiskFPSet, DiskStateQueue).
        Starting SANY...
        Parsing file D:\studyMakesMeHappy\Labwork\tlaplus playground\TCommit.toolbox\Model_1\MC.tla
        Parsing file D:\studyMakesMeHappy\Labwork\tlaplus playground\TCommit.toolbox\Model_1\TCommit.tla
        Parsing file F:\TLAToolbox-1.7.0-win32.win32.x86_64\toolbox\plugins\org.lamport.tlatools_1.0.0.     202004251858\tla2sany\StandardModules\TLC.tla
        Parsing file F:\TLAToolbox-1.7.0-win32.win32.x86_64\toolbox\plugins\org.lamport.tlatools_1.0.0.     202004251858\tla2sany\StandardModules\Naturals.tla
        Parsing file F:\TLAToolbox-1.7.0-win32.win32.x86_64\toolbox\plugins\org.lamport.tlatools_1.0.0.     202004251858\tla2sany\StandardModules\Sequences.tla
        Parsing file F:\TLAToolbox-1.7.0-win32.win32.x86_64\toolbox\plugins\org.lamport.tlatools_1.0.0.     202004251858\tla2sany\StandardModules\FiniteSets.tla
        Semantic processing of module TCommit
        Semantic processing of module Naturals
        Semantic processing of module Sequences
        Semantic processing of module FiniteSets
        Semantic processing of module TLC
        Semantic processing of module MC
        SANY finished.
        Starting... (2020-12-19 16:28:24)
        Computing initial states...
        Finished computing initial states: 1 distinct state generated at 2020-12-19 16:28:26.
        Model checking completed. No error has been found.
          Estimates of the probability that TLC did not check all reachable states
          because two distinct states had the same fingerprint:
          calculated (optimistic):  val = 1.1E-16
        The coverage statistics at 2020-12-19 16:28:26
        <TCInit line 16, col 1 to line 16, col 6 of module TCommit>: 1:1
          line 16, col 13 to line 16, col 46 of module TCommit: 1
        <Prepare line 36, col 1 to line 36, col 10 of module TCommit>: 14:27
          line 36, col 18 to line 36, col 39 of module TCommit: 129
          |line 36, col 18 to line 36, col 27 of module TCommit: 102
          line 37, col 18 to line 37, col 62 of module TCommit: 27
        <Decide line 39, col 1 to line 39, col 9 of module TCommit (39 18 41 66)>: 7:12
          line 39, col 21 to line 39, col 43 of module TCommit: 114
          |line 39, col 21 to line 39, col 30 of module TCommit: 102
          line 21, col 28 to line 21, col 67 of module TCommit: 112
          |line 21, col 28 to line 21, col 37 of module TCommit: 76
          |line 21, col 43 to line 21, col 67 of module TCommit: 76
          line 21, col 23 to line 21, col 24 of module TCommit: 36
          line 41, col 21 to line 41, col 66 of module TCommit: 12
        <Decide line 39, col 1 to line 39, col 9 of module TCommit (42 18 44 64)>: 12:54
          line 42, col 21 to line 42, col 58 of module TCommit: 156
          |line 42, col 21 to line 42, col 30 of module TCommit: 102
          |line 42, col 36 to line 42, col 58 of module TCommit: 102
          line 26, col 31 to line 26, col 54 of module TCommit: 340
          |line 26, col 31 to line 26, col 40 of module TCommit: 178
          line 26, col 26 to line 26, col 27 of module TCommit: 63
          line 44, col 21 to line 44, col 64 of module TCommit: 54
        <TCTypeOK line 10, col 1 to line 10, col 8 of module TCommit>
          line 14, col 3 to line 14, col 69 of module TCommit: 34
        End of statistics.
        Progress(7) at 2020-12-19 16:28:26: 94 states generated (1,778 s/min), 34 distinct states found (643 ds/min),       0 states left on queue.
        94 states generated, 34 distinct states found, 0 states left on queue.
        The depth of the complete state graph search is 7.
        The average outdegree of the complete state graph is 1 (minimum is 0, the maximum 6 and the 95th percentile         is 4).
        Finished in 3297ms at (2020-12-19 16:28:26)
    ```
  * 这些输出以文件的形式进行记录，文件为`SpecName.toolbox/ModName/MC_TE.out`。
* Toolbox和TLC之间的交流方式：
  * Toolbox使用命令行和TLC进行交互（向TLC发送命令
  * Toolbox读取TLC的输出显示在图形界面上
  * TLC通过读取配置文件`SPEC.cfg`以及传入的参数获得每个选项的值
  * 这个cfg文件的所有配置项应该有一个文档才对
  * 不太清楚配置文件分隔每个选项的手段
    * 例如，如果想通过在`SPEC.cfg`里面设置`CHECK_DEADLOCK`参数来让TLC不要检查死锁
      ```
        \* CONSTANT definitions
        CONSTANT
        RM <- const_160846618939547000
        \* INIT definition
        INIT
        TCInit
        \* NEXT definition
        NEXT
        TCNext
        \* INVARIANT definition
        INVARIANT
        TCTypeOK
        \* CHECK_DEADLOCK definition
        CHECK_DEADLOCK
        FALSE
        \* Generated on Sun Dec 20 20:09:49 GMT+08:00 2020
      ```
      这么写会出错，TLC会把CHECK_DEADLOCK这个选项名识别为一个不变量(INVARIANT)
      ```
        \* CONSTANT definitions
        CONSTANT
        RM <- const_160846618939547000
        \* INIT definition
        INIT
        TCInit
        \* NEXT definition
        NEXT
        TCNext
        \* CHECK_DEADLOCK definition
        CHECK_DEADLOCK
        FALSE
        \* INVARIANT definition
        INVARIANT
        TCTypeOK
        \* Generated on Sun Dec 20 20:09:49 GMT+08:00 2020
      ```
      而把`CHECK_DEADLOCK`这个设置放在前面似乎就没问题，大概是因为`NEXT`只能有一个？
      Toolbox生成的配置文件估计是使用TLC的`-deadlock`命令行选项来完成这个配置的。

### 单独运行TLC
* 如果安装了toolbox，tools会在类似`your-installation-path\TLAToolbox-1.7.0-win32.win32.x86_64\toolbox\plugins\org.lamport.tlatools_1.0.0.202004251858`下。可以配置`classpath`来单独运行这些tools。
* 单独运行TLC需要提供: tla文件和用于声明model配置的cfg文件: `java tlc2.TLC YourSpec.tla -config YourConfiguration.cfg`
* 似乎网站上挂出来的TLC命令行选项没有更新。以`-help`里显示的为准？
* 通过toolbox创建的模型，启动的目标model文件应该是`MC.tla`

## 在Distributed/Cloud模式下运行TLC
### Ad Hoc模式下的TLC
* TLC使用一系列的worker线程来实现模型检查。它们计算可达状态的图并对不变量和其它safty属性进行检查。master线程用于协调worker线程。master线程运行于master机上，worker运行于slave机上。
* TLC维护它发现的所有状态的fingerprint，fingerprint用于确定一个新计算的状态是否已经被检查过。当fingerprint太多以至于内存装不下时，TLC会将fingerprint存在硬盘上。（读写硬盘很慢）

#### 运行Master
* 选择IP地址并启动Master之后，Master会进入等待远程worker的状态，并且说明当前有多少worker已经注册到master上。
* 如果slave要负责存储fingerprint，则slave上必须运行一个fingerprint服务器。
* 单独运行Distributed模式的TLC：
  ```
    java -cp YOUR-TOOL-PATH tlc2.tool.distributed.TLCServer MODEL-PATH/MC
  ```

#### 运行Slave
* JRE需要11或以上的版本
* Method2的工作流程：先启动master，然后从`http://master-conputer:10996/files/tla2tools.jar`上获取jar文件，然后从在slave上通过命令行执行jar文件。
  ```shell
  # Get executable jar file
  wget http://master-computer:10996/files/tla2tools.jar

  # To run just worker threads on the slave, execute:
  java -cp tla2tools.jar tlc2.tool.distributed.TLCWorker master-computer

  # To run just a fingerprint server on the slave, execute:
  java -cp tla2tools.jar tlc2.tool.distributed.fp.DistributedFPSet master-computer

  # To run both worker threads and a fingerprint server on the slave, execute:
  java -cp tla2tools.jar tlc2.tool.distributed.fp.TLCWorkerAndFPSet master-computer
  ```
#### Distributed模式/Ad Hoc模式下的局限性
* 不能检查liveness属性
* 不能使用深度优先(depth-first)模式或simulation模式。

### Cloud模式下的TLC
* 要针对不同的云实例类型进行TLC参数的优化
* 执行完成后立刻关闭云实例
  * 除非使用email通知执行结果失败，此时用户必须手动关闭实例。
* 只在一个云实例上运行non-distributed模式
* 并没有看到使用命令行启动Cloud-TLC的方法

#### 用法
* 在启动Toolbox之前，将key和secret设置为系统的环境变量（有点奇怪的做法...为什么不写成配置文件？）
* https://vimeo.com/126244715
* The TLA+ Toolbox中提到: CloudTLC has been implemented directly as part of the Toolbox.

## Model Checking TLA+ Specfications
### Introduction
* Model checker通常是由它能够处理的系统的规模和能够检查的属性类型来评价的。（待检查的）系统一般是通过硬件描述语言或是为model checker定制的语言来描述的。而TLC所检查的specification是通过TLA+来编写的。TLA+是一个具有良好定义语义的语言，它是为了追求强表达性和便于形式推理而不是模型检查来设计的。
* 我们想要验证并发的反应式的系统，例如通信网络和缓存一致性协议。他们的specification通常不是有限状态的，经常包含任意数量的处理器和无限制的消息队列。使用TLC可以很轻易地从这样一个specification中选择一个有限的模型，并且详尽地对其进行检查。
* 对正确性进行证明的关键是找到一个合适的不变量（invariant），即一个在所有可达的状态中都为真的谓词。经验告诉我们验证不变性是最高效的发现错误的技术。
* TLC的某些设计目标让它在速度上有一定妥协，但是TLC希望尽可能地支持更大规模的specification。因此TLC的所有数据存放在硬盘上，使用内存作为缓存。TLC直接地对硬盘访问进行管理来处理状态队列。

### TLA+
（一个使用TLA+编写specification的案例。）

### Checking Models of a TLA+ Specification
* 传统的模型检查都是工作于有限状态的specification上的(with a priori upper bound on the number of reachable states)。而上一节中的model并不是有限状态的：
  * 集合$Proc$可以是任意大的
  * 状态数可以取决于未指明的参数$N$
  * 消息序列$inq[p]$可以任意长
* 使用TLC时，可以通过选择一个model来确定状态的上限。
  * 创建一个新模块继承第二节中编写的模块，并使用新的常量来追加约束条件。
* TLC的输入：
  * module文件
  * 配置文件
    * 初始条件（Init）
    * 下一状态（Next）
    * 约束条件（Constr）
    * 常量的值
    * 不变量
  * Import的module
* Nonexhaustive checking：simulation mode，随机选择behavior，不需要限制条件
* TLC搜索所有可达的状态，试图找到一个状态
  * 不满足不变量
  * 产生死锁：没有下一个可能的状态了
* TLC只有检查完所有满足限制条件的可达状态后才会停止（可达状态无限时TLC可能永不停止）
* TLC允许任何TLA+ module被Java类覆盖（使用Java反射技术），这些类可以实现module中定义的运算符和数据结构

### How TLC Works
* TLC使用明确的（显式的，explicit）状态表示而不是符号表示
  * Explicit state representations seem to work at least as well for the asynchronous systems that interest us
  * A symbolic representation would require additional restrictions on the class of TLA+ specifications TLC could handle.
  * It is difficult to keep a symbolic representation on disk.
* TLC维护两个数据结构
  * $seen$: 一个set保存所有已知的可达状态
  * $sq$: 一个FIFO队列，包含了$seen$中的元素，它们的后继状态都未被检查。
  * $sq$中的元素都是实际的状态，而$seen$中只保存了状态的fingerprint，fingerprint是64位的，probabilistically unique的校验和（所以才会出现hash collision的概率，hash collision可能会导致某些状态没被检测到）
* 对于错误报告，一个$seen$中的entry有一个指针指向它在$seen$中的前驱状态。该指针对于初始状态为null。
* TLC在最开始会生成并检查所有可能的满足初始谓词的状态，并将seen和sq设置为恰好包含这些状态。
* 之后，TLC会将next-state relation重写为尽可能多的subactions的析取式。
* 再之后，TLC会启动一组worker线程，每一个线程都会重复执行以下工作
  * 将队列$sq$的首个元素$s$出队列
  * 对于每个subaction $A$，worker会生成所有可能的下一状态$t$，使得状态对$s, t$能够满足$A$。
  * 对于任意的subaction都不存在可能的下一状态$t$，则报告死锁。
  * 对于每个被生成的下一状态$t$，worker会：
    * 检查$t$是否在$seen$中
    * 如果它不在$seen$中，检查$t$是否满足不变量的要求
    * 如果能满足不变量的要求，那么将$t$和一个指向$s$的指针加入$seen$
    * 如果不满足不变量，则会报错
    * 如果$t$能满足限制条件，那么将其加入队列$sq$。
  * 在报错的情况，TLC会生成一个以$t$为结尾的trace。如果$s$没有任何后继状态，则以$s$结尾。
  * 使用$seen$中的指针能够生成fingerprint的序列。
* TLA+ specification可以包含任何能够通过一阶逻辑和ZF集合论表达的initial谓词和next-state关系，显然TLC不能处理所有这些谓词。TLC大概具有能够处理描述显示系统的specification的能力，但是并不能处理所有的抽象的、高层的specification。
  * （简述了TLC的一些局限性）

### Representing States
* $sq$必须包含实际上的未经检验的状态，而不是它们的fingerprint，$sq$可能会变得非常大，因此需要一个相对紧凑的状态表示法。
* 状态的表示法要求用户编写一个类型不变量（type invariant, 示例中出现过的TypeOK？），包括由每个变量$x$的$x \in T$形式的合取式，其中$T$是一个类型。
  * The types supported by TLC are based on atoms.
  * An atom is an integer, a string, a primitive constant of the model, or any other Java object with an equanlity method.
  * 意思是Atom可以是一个model中的原始的常量，也可以是整数、字符串或者任何一个能判断相等的Java对象
    * `Proc = {p1, p2, p3}`，其中`p1, p2, p3`是primitive的，那么`Proc`就应该不是primitive的？
* 一个类型是建立在一个atom构成的有限集合上的任何一个集合，并且使用常见的集合论中的运算符。
* 状态是对变量的赋值。这个状态的表示法大概是想办法建立一个变量类型到某个自然数集的双射关系，然后想办法用自然数来代表状态。然而这么做的收益没有想象中的大，因此TLC使用一个更简单的方式进行状态表示（但是文中没说到）。
  
### Using Disk
* 作者实现了两个版本的TLC，主要区别是磁盘使用
* 第一个版本是How TLC Works中提到的，使用$seen$和$sq$。
  * $seen$为两个不相交的状态集合的并集(the union of two disjoint sets of states)，一个保存为内存中的哈希表，另一个保存在硬盘上的排序文件上，并且在内存中保留索引。
  * 如果要检查一个状态是否在$seen$中，TLC首先检查内存中的哈希表。如果不在，TLC则利用内存中的索引来查找状态可能所在的硬盘块，读取该硬盘块，并使用二分查找来检查状态是否在该块中。
  * 如果要向$seen$添加一个状态，TLC将其添加到内存哈希表中。当内存哈希表满，它的内容将被排序，并且与硬盘上的文件进行合并，该文件的内存中的索引将会更新。多个worker线程对于硬盘上文件的操作将使用readers-writers lock protocol（读写锁？）进行保护。
  * 队列$sq$实现为一个硬盘上的文件，其开头和结尾的数千个entry会保存在内存中。$sq$是一个FIFO队列，使用一个后台线程来进行预载（将entry读入内存），另一个后台线程用于将entry写入硬盘。处理器时间片会专门分配给这两个后台线程，这总体上能够保证worker线程在访问$sq$的时候不需要等待硬盘I/O。
* 第二个版本的TLC使用三个硬盘上的文件：$old$，$new$，$next$。$old$和$next$最开始都为空，$new$最初为一个最初状态的排序列表。每一轮模型检查都由两步组成：
  1. TLC向$next$文件中追加$new$文件中状态的后继状态，之后对$next$进行排序并移除重复的状态。
  2. TLC将$old$和$new$合并，产生下一轮的$old$文件。在$next$中的状态，如果出现在这个新的$old$中，将会被从$next$中移除。这样产生的$next$文件将会成为下一轮的$new$文件。下一轮的$next$文件将为空。
* 使用这一算法的情况下，在对状态空间进行广度优先遍历时，每个level都需要读写一次$old$文件。
* 为了提高性能，$next$将实现为磁盘文件加上内存缓存，next文件的每个entry都会包含一个$dist$位
  * 一个新生成的状态会加入缓存，当且仅当这个状态不在缓存中，并且$disk$位置为0。
  * 当缓存满时，$disk$位为1的entry会被驱逐(evicted)以腾出空间。当特定比例的entry持有为0的$disk$位时，这些entry会被排序、写到硬盘，并且将他们的$disk$位置为1。（有一点像第二次机会页面交换算法）


## The TLA+ Toolbox
### Toolbox Features
#### Model
* TLAPS可以对具有无限种可能状态的specification进行验证。但是显式状态(sxplicit-state)的model checker完成不了对无限状态空间的验证。TLC的主要目的是让spec不需要进行修改就可以被检测。TLC model通过为spec的参数赋予具体值并声明额外的边界，将spec的状态空间限制为一个有限的可能状态集。用户一般会使用多个独立的model来检测同一个spec以提高可信度。
* model以XML文件的形式保存在文件系统上。
  * 保存为`.launch`文件，文件内容实际上是XML格式

#### CloudTLC
* CloudTLC准备了一组云实例并在上面运行TLC。如果用户选择在多个实例上运行模型检查，则CloudTLC会以distributed模式启动TLC。当模型检查完成时，CloudTLC实例会自行gracefully terminate。如果用户一直打开Toolbox，Toolbox会显示模型检查的进度以及最终结果。Toolbox也有可能被关闭(失去与CloudTLC的连接)，此时CloudTLC会将运行结果发送到用户提供的邮件地址，检查结果可以被导入Toolbox进行查看。换言之，CloudTLC是完全透明的。
* CloudTLC的启动时间主要取决于IaaS提供商启动实例的时间。为了减少启动时间，后续的model checker运行会重用先前准备的实例，除非他们已经被停止。CloudTLC同样可以通过Toolbox的命令行模式进行启动(?)，这对于实现自动操作是非常有用的，例如运行TLC性能测试。

#### Results
* TLC的检查结果包括
  * 经过时间
  * 未能完整地检查状态空间的概率
  * Action statistics: similar to global state-graph statistics except that they are reported at the level of individual TLA+ actions and the diameter is undefined
  * 全局状态图统计信息(global state-graph statistics)
    * 直径: TLC在状态图探索中达到的深度(有向状态图中的最长路径，路径中没有重复的状态)
    * 独立状态(distinct states): 状态图可达顶点集的基数
    * 所有状态(total states): TLC检查的状态数
    * 独立状态与被检查的所有状态的比值约等于状态图的度数。
* 状态图可以被导出并可视化。

#### Profiler
* 对TLA+ model及其spec进行分析(Profiling)可以生成4种不同类型的数据(Action statistics?)
  * 调用计数(invocation count): TLA+表达式被计算的次数
  * 开销(Cost): 要枚举一个集合或多个集合中的元素所需要执行的操作数，如果要计算一个表达式需要枚举某些数据结构(shuold enumeration be required to evaluate an expression)
  * States/Action: 一个特定的TLA+ action所产生的需要进行检查的状态数
  * Distinct States/Action: 一个action所产生的独立状态(distinct state)数
* 前两种数据类型被称为计算统计信息(evaluation statistics)，并在全局层面和调用链层面对它们进行收集。
  * 假设所有表达式都有一个相同的、固定的开销(cost)。在这种情况下，用户可以通过查看调用数来判断模型检查中消耗时间最多的部分。
  * 然而，一些表达式需要model checker去显式地枚举数据结构，而这些表达式的开销是需要定量测量的。
    * 比如这样的一个例子: $\forall s \in \operatorname{SUBSET} S:s \subseteq S$。计算该表达式的开销取决于TLC枚举$\operatorname{SUBSET} S$的所有子集($S$的幂集)所需要的操作数。这一表达式会模型检查中消耗时间较多的部分，即使它的调用数较少。
  * 有了action statistics，用户可以在action层面上准确地定位状态空间迅速扩大(状态空间爆炸, state space explosion)的源头。
* 这个信息有什么用呢?
  * 能表达相同意思(例如，不违背safety property...)的情况下，编写具有更低的total states/distinct states比值的action会让模型检查更高效
  * 有些action产生的total states和distinct states为0，这可能表明spec存在错误，或者说该action是多余的，可以被删去。
* Toolbox会按照一条表达式的计算统计信息生成一维的热力图，并用热力图中的颜色对表达式进行高亮着色。
* 总而言之，Profiler可以凸显模型检查中各种不同类型的低效之处。对于和表达式计算相关的低效之处，model为用户提供了一种方式来使用更高效的变体来覆写TLA+的运算符。一种更极端的优化手段是使用等效的Java函数来覆写TLA+的运算符。Action statistics将支撑谓词的弱点或是spec中的错误暴露出来。
* Profiler不需要对spec或model进行修改就可以收集统计数据。然而分析工作会带来性能开销，检查大型模型的时候应该被禁用。以分布式模式运行TLC时无法进行分析。

#### Trace Explorer
* 如果模型检查发现了违背了safety属性或liveness属性的情况，对应的错误跟踪会在Trace Explorer中显示。错误跟踪是一个状态序列，状态则是一次对变量的赋值。
* Trace Explorer支持trace expression(该表达式会在错误跟踪中的每个状态中计算一次)的计算。Trace expression可以被命名，进而可以实现表达式的组合。Trace expression可以使用root module的所有运算符来进行编写，除此之外还可以使用两个额外的运算符:
  * `_TEPosition`: 当前状态在错误跟踪中的位置(错误跟踪的第几个状态)。
  * `_TETrace`: 一个TLA+状态序列，`_TETrace[_TEPosition]`为错误跟踪中`_TEPosition`位置的状态。（也就是error trace的状态序列）
* 这两个额外的运算符提高了trace expression的表达力，因为通过这两个运算符，trace expression可以使用整个错误跟踪中所有状态中的变量。例如，trace expression可以比较两个或多个任意状态中的变量的值。

### Toolbox Architecture
* Toolbox主要使用了Eclipse Rich Client Platform(RCP)来进行GUI的构建。其他人可以通过贡献扩展和OSGi服务来增加功能。

#### Toolbox to Back-end
* Toolbox依赖RCP框架来提供用户可见的程序运行进度报告以及支持取消操作。为了支持这些功能，toolbox的后端需要实现设置命令行参数的适配器。
* Toolbox将模型的一部分序列化为一个纯文本的配置文件。这个文件包括TLA+的hehavior spec、待检查的不变量、待检查的属性、所有声明了的常量的定义。该配置文件还有一些可选部分，包括运算符覆写以及状态和action的限制。这些配置文件并不是TLC专属的，其它back-end也能使用。

#### Back-end to Toolbox
* Toolbox使用一个连接到Toolbox UI的框架，使用Model-View-Presenter(MVP)设计模式，解析后端的进度和运行结果。
* 一个特定的后端解析器需要阅读一个后端的输出流，这一输出流可能由大小不一的不完整的print语句块组成。基于效率的考量，对输出的解析是基于特殊的token和常量的，这些token和常量会包裹输出流。
* 解析分三个阶段进行
  * 将字符块缓冲到由换行符分隔的行中(buffers chunks of characters into lines separated by a newline character)
  * 通过token识别出属于多行语句的行
  * 通过MVP将多行语句反序列化为Java对象
* 总之，back-end integration在一个足够低的层次，从而不需要提供诸如REST API的高层接口。取而代之的是，后端的基于文本的IO可以被复用。但这种底层集成的灵活性是有代价的: 缺少编译时的验证让back-end的进化更加困难了。

#### CloudTLC Back-end
* CloudTLC构建于Toolbox的框架之上，同时使用了一个multi-cloud的工具箱(toolkit, provides an abstraction from individual IaaS providers)。
* CloudTLC的工作过程大概可以被分为4个阶段:
  * 部署(Deploy): toolbox通过HTTP向IaaS提供商查询带有标记的CloudTLC实例。如果请求结果为空，toolkit会请求IaaS提供商启动指定数量的实例。如果查询结果不为空，toolbox启动返回的实例，并且跳过接下来的准备阶段。
  * 准备(Provision): Toolbox配置示例上的操作系统并安装model checker的依赖。Authentication credentials来自Toolbox的环境变量。准备阶段toolbox会直接通过ssh和每个实例进行通信。
  * 启动(Launch): Toolbox将spec和model复制到实例，并启动model checker。model checker持续地将它的输出流传送到本地Toolbox。TLC同时将结果发送到用户提供的电子邮件地址。
  * 停止(Terminate): 云实例会为后续的连接尝试等待一个宽限期。在这之后，当且仅当上一阶段的邮件成功发送后云实例会自行停止。有些IaaS提供商要求停止请求需要进行认证，在这种情况下需要用到之前提到的credentials。
* 在开发过程中，云实例的硬件配置都是已知的，model checker可以对云实例的部署和配置进行优化。例如，OS和TLC的运行时参数是硬编码的(hard-coded)。
* CloudTLC是为TLC而设计和实现的，但是它可以轻易地支持新的back-end(比如后文提到的Apalache?)

#### Testing
* Toolbox的开发遵循一种测试驱动和先开发后测试(test-last)的方法学。有专为model checker和proof system设计的测试套件。
* 测试是全自动的，编译使用持续集成系统。

### Conclusion
* Toolbox的两大功能是其它IDE通常不提供的:
  * CloudTLC
  * Profiler

### Future Work
* Profiler  
  用户可以使用Java函数来对低效的TLA+表达式进行覆写，但是用户如何断言它们编写的Java函数和待覆写的TLA+表达式具有等价性仍是一个待解决的问题。
* Trace Exploration  
  Trace Explorer对于寻找错误的源头很有用，但是纯文本的表现形式并不是很有利于理解系统的动态性。(Trace Animator)
* Back-ends
  新的model checker Apalache，从该model checker的团队发表的论文TLA+ model checking made symbolic( https://dl.acm.org/doi/10.1145/3360549 )来看，在一些标准测试上性能强于TLC。TLA+团队有使用Apalache作为新的Back-end的意愿，但是目前似乎仍有待解决的问题。

## 个人总结
### TLC Runtime API
#### 需要增强的功能
* 暂停/恢复TLC运行: 该功能在现有版本已经通过JMX中实现，通过JMC或jconsole可以连接到MBean服务器来执行该方法。
* 运行时减少/增加worker线程的数量：尚未实现
* 触发检查点并实现gracefully terminate: 从通过JMX暴露出来的MBean来看，该功能在现有版本中已经实现了触发检查点的方法，但是该方法似乎并不会gracefully terminate。
* 触发liveness checking：从通过JMX暴露出来的MBean来看，该功能在现有版本中已经实现了强制进行liveness checking的方法。
* 能够观察当前生成的状态的检查器(生成的状态数, 当前的trace深度)：该功能在现有版本已经通过JMX中实现，通过JMX可以获取到当前的State，并且可以获得当前在状态图探索中到达的深度
* Metrics overall and for fingerprint set, state queue, trace(?)