# JVM(Java Virtual Machine) 汇总：
------

## **监控命令**

### jps  
- 查看Java进程列表：`jps -l`

### jinfo
- 启动参数：`jinfo pid`
- 虚拟机启动参数： `jinfo -flags pid`
- Java启动参数：`jinfo -sysprops pid`

### jmap
- 生成内存堆 Dump 信息：`jmap -dump:live,format=b,file=heap.hprof pid`

### jhat
- 查看内存堆 Dump 信息(启动Http服务)：`jhat heap.hprof`

### jstack
- 生成线程栈 Dump 信息：`jstack [-F] pid > current.tdump`


## **监控工具**

### jconsole

- 连接
	- 本地进程
	- 远程进程(jmx)
- 监控
	- 堆内存使用量
	- 线程数
	- 类加载情况
	- CPU占用率
	- VM概要
	- MBean值查看

### jvisualvm(JavaVisualVM)
- 支持插件安装
  - Visual GC
- 导入(推荐)
  - 线程 Dump(*.tdump)
  - 堆 Dump(*.hprof, *.*)
		
### jmc(JavaMissionControl)
- 支持插件安装
- 可导入堆 Dump(*.hprof, *.*)
- 飞行记录器(HotSpotJVM)
	- 需加程序启动参数：`-XX:+UnlockCommercialFeatures -XX:+FlightRecorder -XX:FlightRecorderOptions=defaultrecording=true`

	
## JAVA 内存模型

- JMM(Java Memory Model)
- 线程 <-> 本地内存（共享变量的副本） <-> 主内存（共享变量）
- Java内存模型 与 Java 并发编程 有关
- 并发通常需要解决的问题
	- 原子性
		- synchronized
		- Lock
		- java.concurrent.Atomic.* 包
	
	- 可见性
		- volatile
		- synchronized
		- Lock
	- 有序性（重排序）
		- 编辑器优化
		- 指令重排，不存在依赖的数据，改变执行顺序
		- 内存系统重排序，发生在缓冲区

## JVM 运行时数据区

- 方法区（元空间）
	- 线程共享
	- 存放被虚拟机加载的类型信息、常量、静态变量、即时编译后的代码缓存等数据

- JAVA堆（Heap）
	- 线程共享
	- 存放对象实例

- 程序计数器（PC寄存器）
	- 线程私有
	- 指向下一条指令地址
	- 该区域是整个jvm内存区域中唯一的没有OOM的区域


- 本地方法栈
	- 线程私有
	- 储存局部变量表、操作数栈、动态链接、方法出口、对象指针等信息
	
- JAVA虚拟机栈
	- 线程私有


## JVM 垃圾回收

### 垃圾确认算法
- 引用计数法
- 可达性分析法

### 垃圾收集算法
- 标记清除法
- 标记整理法
- 复制算法

### 垃圾收集器

- Serial
- ParNew
- Parallel Scavenge
- Serial Old
- Parallel Old
- CMS
- G1
- ZGC

## RMI(Remote Method Invocation)
- 启动参数：
`-Dcom.sun.management.jmxremote -Djava.rmi.server.hostname=172.30.64.110 -Dcom.sun.management.jmxremote.port=9999 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false`


------

## 常用参数说明

### 设置最大、最小堆内存
	-server -Xms4g -Xmx4g

### 遇到 OutOfMemoryExceptions ，内存溢出堆转储到文件
	-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=data

### 遇到内存溢出执行shell命令或脚本
	-XX:OnOutOfMemoryError=kill -9 %p
	-XX:ErrorFile=logs/hs_err_pid%p.log

### 垃圾收集器
- CMS  
  `-XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly` 
- G1  
  `-XX:+UseG1GC -XX:MaxGCPauseMillis=50`
- ZGC（JDK11+）

### GC 日志  
- 保存 GC 日志  `-XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCTimeStamps -XX:+PrintTenuringDistribution -XX:+PrintGCApplicationStoppedTime -XX:+PrintGCApplicationConcurrentTime -XX:+PrintHeapAtGC -Xloggc:logs/gc.log`
- 回滚 GC 日志文件  `-XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=32 -XX:GCLogFileSize=64m`
- 控制台输出 GC 日志  `-verbose:gc`

### 禁用通过 System.gc() 显式执行 Full GC
	-XX:+DisableExplicitGC

### 打印系统变量信息
	-XshowSettings:all
	-XshowSettings:properties


