# CloudTLC Source Code Analysis
## CloudTLC实现的大致结构
* CloudTLC作为toolbox的一部分被实现为一个Eclipse RCP应用(Eclipse RCP应用的本质是一个Eclipse插件，也就是一个OSGI包)。
* CloudTLC相关的类在`org.lamport.tla.toolbox.jclouds`中。位于`/src/org/lamport/tla/toolbox/jcloud/`下。

## 总入口: Application.java
* 启动CloudTLC的属性`props`，以key-value形式描述启动CloudTLC的属性。
  * 邮箱地址
  * 主类名称
  * MODEL名称
  * SPEC名称
* `initializeFromFile()`: 从`modelDirectory/cloud.properties`中读取启动CloudTLC的配置信息(如果该文件存在)
* `tlcParams`: CloudTLC的运行参数。只有一部分TLC的参数能够在CloudTLC中使用。如果只是增加云服务提供商，构建`tlcParams`的流程应该不需要改
  * `-fp`
  * `-maxSetSize`
  * `-deadlock`
  * `-coverage`
* 根据传入模块的参数等创建`CloudDistributedTLCJob`
* 调用`CloudDistributedTLCJob`的`run`方法。传入`MyProgressMonitor`的实例对`CloudDistributedTLCJob`的运行情况进行监控，响应`CloudDistributedTLCJob`的取消等事件，并进行进度展示。
* 如果返回的`IStatus`表明执行出错，则打印错误信息(stderr)，返回执行不成功的信号(`return 1`)。
* 如果没有出错，则持续接收远程TLC进程的输出(stdout)，并输出到本地的stdout中。

## CloudTLCJobFactory.java
* 一个简单工厂，根据传入的`aName`参数来判断指定了哪个云服务提供商，创建不同的`CloudTLCInstanceParameters`的子类，并用其创建不同的`CloudDistributedTLCJob`对象。

## CloudTLCInstanceParameters
* 持有运行TLC所需要的参数、创建云实例所需要的其他参数、JVM的启动参数
* 其中TLC本身所需要的参数由单个字符串变量(`tlcParams`)表示
* `numbersOfWorkerNodes`: 如果在构造时不提供或传入`1`，则之后将在云实例上调用non-distributed TLC。如果构造时传入大于`1`的值，则会启动distributed TLC。
* `getJavaSystemProperties()`
  * 返回拼接好的JVM启动参数字符串
  * 如果`numbersOfWorkerNodes`为`1`则返回`"-Dtlc2.tool.fp.FPSet.impl=tlc2.tool.fp.OffHeapDiskFPSet"`
  * 如果`numbersOfWorkerNodes`大于`1`则返回`"-Dtlc2.tool.distributed.TLCServer.expectedFPSetCount=" + (numberOfWorkerNodes - 1)`
* `getJavaVMArgs()`: 总之是提供一系列的JVM启动参数字符串。`getJavaWorkerVMArgs()`同理。
* `getTLCParameters()`: 提供TLC启动参数字符串。
* 其余涉及云实例的`getter`方法由子类进行重写。已知继承该类的子类有`EC2CloudTLCInstanceParameters`，`AzureCloudTLCInstanceParameters`，`PacketNetCloudTLCInstanceParameters`

### EC2CloudTLCInstanceParameters
* `getOwnerId()`: 暂不明白，返回固定值`"owner-id=owner-id=099720109477;state=available;image-type=machine"`
* `getCloudProvider()`: 获得云服务提供商名称
* `getIdentity()`: 从系统变量（在启动JVM时用`-Dxxx=yyy`来指定）中获得id。（在EC2中为`AWS_ACCESS_KEY_ID`
* `getCredentials()`: 从系统变量中获得secret。（在EC2中为`AWS_SECRET_ACCESS_KEY`
* `validateCrededentials()`: 通过已经确定的规则验证key和secret的正确性。（比如EC2的id必以`AIKA`开头并总共有20个字符
  * 返回值为`IStatus`类型，是eclipse平台的相关接口。猜测是用于描述组件的状态。如果该方法验证通过，则返回`Status.OK_STATUS`，否则通过`Status.ERROR`和错误信息构造`Status`类。
* `mungeProperties()`: 设置了两个关于`jclouds`的属性。注意到`getOwnerId()`和`getRegion()`都有固定的默认值
* `mungeTemplateOptions()`: 暂不明白，接受`TemplateOptions`（`jclouds`相关类）。猜测是完成了创建子网和复用子网的工作。
* `getHostnameSetup()`: 查找公用ipv4主机名并设置相应的主机名（而不是保留云实例默认的主机名），这可以减少发给用户的邮件被当成垃圾邮件的概率。应该是通过ssh连接到云主机实例之后需要的工作。
* `getImageId()`: 返回指定的linux镜像类型（镜像id）。搭配`getRegion()`进行使用
* `getRegion()`: 返回EC2的所属区域。如果没有在系统变量里设置则固定使用`us-east-1`
* `getHardwareId()`: 设置云实例的类型
* `getOSFilesystemTuning()`: 对云实例的FS进行调整。参见`getOSFilesystemTuningDefault()`
* `getOSFilesystemTuningDefault()`: 一系列的linux命令。大意是用两个实例存储创建一个raid0，优化磁盘读写性能。
* `getJavaVMArgs()`: 获得JVM参数
* `getTLCParameters()`: 获得TLC参数
* `getJavaWorkerVMArgs()`: 暂不明白与`getJavaVMArgs()`的区别

## CloudDistributedTLCJob.java
* 增加云服务提供商主要是增强`CloudDistributedTLCJob`的功能。（同时要实现对应的`CloudTLCInstanceParameters`的子类
* 该类的基类`Job`是一个与Eclipse平台相关的类。属于Eclipse平台的核心组件`org.eclipse.core.*`，但不涉及UI（有没有UI都可以运行）。
  * Jobs are units of runnable work that can be scheduled to be run with the jobmanager. Once a job has completed, it can be scheduled to run again (jobs arereusable).
  * Job有
  * 可能是eclipse平台里一种封装好的任务调度手段？
* `aName`: 云服务提供商
* `nodes`: 节点数量（worker数量）
* `params`: `CloudTLCInstanceParameters`的子类实例
* `groupNameUUID`: 暂不明确
* `props`: 在`Application.java`中构造的启动`CloudTLC`的一些配置
* `modelPath`: 存放model的路径
* `SHUTDOWN_AFTER`: 默认值为10，应该是CloudTLC关闭前的宽限期(graceful period)。
* `isCLI`: ?
* `doJfr`: 默认值为false，是否开启JFR(Java Flight Recorder)。
* `run()`: 接受`IProgressMonitor`参数
  * 首先调用`this.params.validateCredentials()`来确认提供的`key`和`secret`是否正确。（只是验证符不符合基本格式，并没有验证是否真的有效）
  * 然后启用一个后台线程，使用`PayloadHelper`调整`tla2tools.jar`让其包含需要使用的spec和model。(执行结果是一个和jclouds有关的类实例`org.jclouds.io.Payload`)
  * 调用`CloudTLCInstanceParameters.mungeProperties()`为`properties`设置必要属性(根据不同的云服务提供商各有不同)。
  * 在云实例上建立计算环境，并注入一个ssh实现。主要通过ssh和计算节点进行通信。
  * 构建Context（一个连接到云服务提供商的connection，可以类比为数据库连接）。（`mungeBuilder()`的作用？）
  * 创建或复用已经创建的云实例（line210-233,`findReusableNodes()`（只支持non-distributed CloudTLC）, `provisionNodes()`）
  * 使用ssh将`/tmp/tla2tools.jar`（通过上文提到的后台线程生成）复制到**一台**主机上（复制到master上）
  * 用字符串拼接在master实例上运行的linux命令`tlcMasterCommand`，使用jclouds执行命令运行`tla2tools.jar`(line323-326，使用了jclouds)。
  * 如果`nodes`大于1，则需要对其它的worker节点进行配置
    * 在master节点上，将`tla2tools.jar`里的spec、model和`generated.properties`去掉后复制到web服务器的根目录下。
    * 在所有的worker节点上，通过`wget`将master上web服务器的`tla2tools.jar`下载到本地，然后运行Distributed TLC Slave。
  * 使用linux的`tail`命令读取master上的`MC.out`的内容。
  * 返回`CloudStatus`的实例，该类是`Status`的子类，可以表示CloudDistributedTLCJob的执行状态，持有通过`sshClient.execChannel()`得到的`InputStream`和连接到master的`SshClient`实例。

## PayloadHelper.java
* 只有静态方法的工具类
* `checkToolsJar()`: 检查`tla2tools.jar`是否存在。`tla2tools.jar`丢失时直接抛出异常。
  * 使用了`getToolsURL()`来查找`tla2tools.jar`的位置。
* `appendModel2Jar()`: 
  * 首先在默认临时文件目录创建一个`tla2tools.jar`文件，并将通过`getToolsURL()`获取的tla2tools.jar文件内容复制到该临时文件中去。
  * 然后将spec、model和通过`props`构造得到的`generated.properties`复制到临时jar文件的`/model`文件夹下。
  * 通过上述的临时文件获得`InputStream`，构造`InputStreamPayload`并返回。
