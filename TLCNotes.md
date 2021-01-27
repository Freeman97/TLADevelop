# TLC Source Code Analysis
## TLCGlobals.java
* 一个全是静态字段和静态方法的类。记录当前TLC运行的具体信息。（例如当前使用的ModelChecker是BFS还是DFS，当前使用了多少个worker线程等）

## TLC.java
* `main()`
  * 调用`handleParameters()`方法来处理TLC的运行参数。
  * 调用`tlc.process()`方法启动TLC。
* `process()`
  * 根据输入的参数和选项启动TLC
  * 在执行模型检验之前，先通过```TLCStandardMBean modelCheckerMXWrapper = TLCStandardMBean.getNullTLCStandardMBean();```创建一个MBean交给JMX进行管理，从而可以通过该MBean观察TLC的运行状态以及对运行中的TLC进行操作。
    * 然而实际上，`ModelCheckerMXWrapper`类才真正能够暴露TLC运行信息和对TLC进行操作的方法。只有在BFS模式中，才会初始化`ModelCheckerMXWrapper`。只有在以BFS模式运行TLC时，TLC的MBean服务器才能查询到与TLC有关的运行信息（`tlc2.tool`，`tlc2.tool.fp`）。
  * 如果是simulate模式，则实例化`Simulator`。如果是BFS模式，则实例化`ModelChecker`。如果是DFS模式，则实例化`DFIDModelChecker`。后两者为`tlc2.tool.AbstractChecker`的子类。
  * 调用`Simulator`的`simulate()`或`AbstractChecker`的`modelCheck()`(实际上由子类的`modelCheckImpl()`来完成具体工作)来进行模型检验。
  * `ModelCheckerMXWrapper`持有`TLC`和`ModelChecker`的实例。但是只有`SpecName`和`ModelName`这两个属性的获取和`TLC`的实例有关。其它属性的获得以及方法的实现都需要使用`ModelChecker`。

## AbstractChecker
* `IWorker[] workers`: 线程集合
* `ITool tool`: 处理spec的工具类
* `modelCheck()`: 运行model checker的主方法
* `modelCheckImpl()`: 由子类实现的model checker的具体运行方法
* `runTLC(int depth)`: TLC启动的主方法。该方法会被子类实现的`modelCheckImpl()`调用。
  * 启动用于模型检验的线程`startWorkers(AbstractChecker checker, int checkIndex)`(该方法为抽象方法，由具体的子类实现，`checkIndex`其实就是`depth`，即深度，在BFS模式下不需要考虑该参数)。
  * 调用`doPeriodicWork()`、`runTLCContinueDoing(int count, int depth)`方法
    * 建立检查点(checkpoint)
    * 进行周期性的liveness checking
  * 在上述模型检验工作完成后，退出循环，等待所有worker线程停止(调用`join()`)

## ModelChecker
* **因为目前的TLC Runtime API只在BFS模式中提供，因此暂时只考虑BFS模式下Runtime API的实现方式**
* `AbstractChecker`的子类，实现BFS模式
* 在该类的构造函数处完成对`Workers`的初始化。
  * 从`TLCGlobals.getNumWorkers()`获得应该初始化的worker线程数
  * 初始化`ConcurrentTLCTrace`。相比于它的超类`TLCTrace`，它可以让多个worker线程并发地向Trace添加状态，而在`TLCTrace`中这样的操作是阻塞的。
  * 初始化`theFPSet`。
* `modelCheckImpl()`: 实现model checker的具体方法
  * 所有访问的状态都保存在`theFPSet`中，所有状态的未访问的后继状态都保存在`theStateQueue`中。
  * 首先调用`recover()`，试图从检查点恢复。如果没有检查点，则需要初始化state queue和state set。
  * 调用`doInit()`对model checker进行初始化。
    * `doInit()`负责生成所有的初始状态。但尚未清楚`doInit()`及其之后的调用链上是否对`theStateQueue`和`theFPSet`进行了修改，如果按照`ModelChecker`的代码来看，`doInit()`会将所有的初始状态加入`theStateQueue`和`theFPSet`中，但是暂不明确这一操作的具体位置。
  * 调用`runTLC(Integer.MAX_VALUE)`启动TLC（参数`depth`应该是用于在DFS中限制最大的状态深度）
    * 调用`startWorkers(AbstractChecker checker, int checkIndex)`。
      * `ModelChecker`持有所有的worker线程类`Worker`的实例数组（`IWorker[] workers`）。`Worker`实现了`IWorker`，继承了类`IdThread`。是`Thread`的子类（非直接）。在该方法中直接调用`Thread`的`start()`方法来启动线程。
      * 其中的`incWorkers(int num)`的作用暂不明确。`incWorkers(int num)`在fingerprint set的公共抽象超类`FPSet`中定义，是一个可被覆写的具体方法，提供一个什么事情都不做的默认实现。只有`MultiFPSet`和`OffHeapDiskFPSet`需要用到这一方法。`OffHeapDiskFPSet`的内部类`OffHeapSynchronizer`实现了该方法。暂时不明白`OffHeapDiskFPSet`使用的同步机制。
    * 循环检查条件`this.done`，不满足该条件则继续运行
      * 调用`doPeriodicWork()`，该方法负责两件事：检查liveness属性、创建checkpoint（包含3个数据结构：state set、state queue、state trace）
        * 只有在`doPeriodicWork()`中才会调用`checkpoint()`创建checkpoint，因此就算通过JMX调用`forceChkpt()`，也要等到"next time possible"（下一个循环）才会创建checkpoint。
        * 创建检查点前需要调用`this.theStateQueue.suspendAll()`，创建完成后需要调用`this.theStateQueue.resumeAll()`。
      * 调用`runTLCContinueDoing(int count, int depth)`，主要用于统计进度和打印信息。
    * `runTLC()`返回时，所有的worker线程都会被停止(调用`join`方法)。
  * 

## Worker
* worker线程类，封装了worker线程所需要持有的字段和需要进行的操作。
* `run()`：`Worker`的主方法
  * 从当前的状态队列中取出一个状态。如果取出的状态为空则说明模型检查即将结束，进行将`ModelChecker`的`done`字段设置为true并进行post condition check。（post condition check只需要进行一次，但这个操作在worker线程类中进行，因此需要获取`ModelChecker`的对象锁）
    * 问题：为什么`currentState`需要使用`ThreadLocal`为不同线程创建副本？还有哪些线程会使用worker线程的这一字段？
  * 获取当前状态的后续状态(交由`FastTool`处理)，然后在`addElement(final TLCState curState, final Action action, final TLCState succState)`中检查这个后继状态是否已经检查过(`isSeenState(TLCState curState, TLCState succState, Action action)`，当该后继状态不在当前FPSet中时，会将该后继状态添加到FPSet和trace中)、是否满足不变量(`doNextCheckInvariants(TLCState curState, TLCState succState)`)，最后调用`IStateQueue`的`sEnqueue(TLCState state)`方法将该状态加入队列。

## tlc2.tool.fp.FPSet
* 实现用于判断某个状态是否被检查过的fingerprint set的基类。大致可以被分为`DiskFPSet`和`MemFPSet`两种类型，以及将不同fingerprint set组合起来的`MultiFPSet`和为空操作设计的`NoopFPSet`。共同点是需要保证它们的方法都是线程安全的。一般情况下正常使用都使用`DiskFPSet`，`MemFPSet`只用于测试代码。
* 提供了`addThread`方法

## tlc2.tool.fp.DiskFPSet
* 使用有限的内存，当fingerprint的内存占用超过限制时，fingerprint会被写到硬盘上的文件中。
* 这一实现使用了排序文件。
* 为每个worker线程都创建一个单独的`BufferedRandomAccessFile`对象（但都表示同一个文件）。该实现使用了`ReadersWriterLock`进行同步，因此在硬盘上进行查找是可以同步进行的。`BufferedRandomAccessFile`是对`java.io.RandomAccessFile`的继承，表示一个带有缓冲区的随机访问文件。
* 使用最高有效位(MSB)来表示一个fingerprint是否被从内存中写回硬盘。
* 调用`init(int numThreads, String aMetadir, String filename)`时会为每个线程创建一个`BufferedRandomAccessFile`并保存在`this.braf`中。

## tlc2.tool.fp.HeapBasedDiskFPSet
* `put`方法：首先查找fingerprint，查找到时直接返回`true`，否则需要将fingerprint插入内存缓冲区并返回`false`。内存缓冲区会在适当的时机将数据写到硬盘的文件上。


## TLC2.tool.ConcurrentTLCTrace
* 维护Trace文件的类。持有当前的所有worker的实例，多个worker可以并发地向这个trace增加状态，基类为`TLCTrace`。worker需要维护Trace文件，模型检查出现错误的时候需要重构error trace。每个worker都有他们专属的文件，而这些worker持有的trace文件可能是不完整的("trace fragment")，当worker需要创建一个反例时，需要对所有的trace文件进行整合。每个worker的trace fragment在硬盘上的文件可能是不一致的(`Worker.java` line 202-204: The on-disk file of each worker's trace fragment is potentially inconsistent because the worker's cache in BufferedRandomAccessFile hasn't been flushed out.)，猜测大概是`BufferedRandomAccessFile`在内存中的缓冲区没有写回硬盘导致的不一致。