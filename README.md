# proj35-diagnose-tools
### 项目名称
Linux服务器集群故障分析工具。

### 项目描述

与单机故障诊断有所不同，集群内的偶发故障更难于捕获。

Linux服务器常见的故障表现是：

1. Load 高；
2. 应用程序 RT 抖动；
3. 网络/IO 抖动。

如何正确的识别异常信息并及时产生集群告警，存在如下难点：

1. 集群内个别机器偶发异常，这要求在集群内大规模的部署工具。对工具的稳定性、性能损耗都有严格的要求。

2. 发生异常的时间点并不确定，要求工具能在集群内常态化的运行。

3. 故障往往只在生产环境中才能出现，而生产环境的压力较大。例如单机网络达到10Gb，或者单机存在数百块磁盘，每秒IO量达到数GB。在这些机器中运行工具跟踪网络/IO流量，不可避免会对业务造成影响。

举一个简单的例子，在单机中识别Load高的原因，可以简单的编写一个Shell脚本，遍历/pro目录下所有线程的内核堆栈，并使用pstack/gdb命令获取其用户态堆栈。这在生产系统中并不可行。因为：

1. 通过读取/proc目录的方法获取堆栈，可能消耗长达一分钟的CPU时间。这会加重系统的抖动。

2. pstack/gdb命令获取单个线程的堆栈可能会消耗6秒的时间，而系统中的抖动可能只持续1秒，这导致无法抓到正确的堆栈。

3. pstack/gdb命令的实现原因导致：它不能正确的抓到Linux内核引起的问题。

diagnose-tools是阿里生产环境中大量使用的故障分析工具，该工具通过内核模块的方式在操作系统中挂接钩子，探测系统中的异常。2020年9月在云栖大会上正式开源。

当前项目实现源码位于：https://github.com/alibaba/diagnose-tools

### 所属赛道

2021全国大学生操作系统比赛的“OS功能设计”赛道

### 参赛要求

- 以小组为单位参赛，最多三人一个小组，且小组成员是来自同一所高校的本科生（2021年春季学期或之后本科毕业的大一~大四的学生）
- 如学生参加了多个项目，参赛学生选择一个自己参加的项目参与评奖
- 请遵循“2021全国大学生操作系统比赛”的章程和技术方案要求



### 项目导师

谢宝友

* github
* gitee https://gitee.com/xiebaoyou/
* email [baoyou.xie@alibaba-inc.com](mailto:baoyou.xie@alibaba-inc.com)

### 难度

中等

### 特征

- 兼容2.6 / 3.x / 4.x / 5.x不同Linux版本
- 兼容centos / ubuntu / debian等发行版
- 以纯内核模块的方式实现
- 尽量采用无锁编程，极致的性能

### 文档

- https://github.com/alibaba/diagnose-tools/tree/master/documents
- https://gitee.com/xiebaoyou/documents

### License

[GPL-3.0-only](https://opensource.org/licenses/GPL-3.0)

### 平台实现注意事项

- 对稳定性有很强的要求，特别是要注意无锁编程的正确性、稳定性。
- 需要对Linux内核有深入的理解。

## 预期目标

### 注意：下面的内容是建议内容，不要求必须全部完成。选择本项目的同学也可与导师联系，提出自己的新想法，如导师认可，可加入预期目标

### 第一题：完善diagnose-tools的功能

- diagnose-tools在网络/IO方面的功能还较少
- diagnose-tools增加对python调用链解析
- 如何实现对shell脚本进行调用链回溯

### 第二题：业务进程画像支持

- 在混部系统中，对不同的业务进程进行画像，是很有意义的事情。例如，不同的业务进程可能是CPU密集型/内存访问密集型/IO密集型。其中内存访问密集型进程可能频繁的破坏系统缓存，极端情况下可能导致相邻CPU的业务进程CPU占用率增加一倍，带来业务进程的抖动甚至故障。
- 可以在diagnose-tools中增加对PMU性能寄存器的采样，并将这些采样数据与进程调度/CGROUP信息结合起来，实现对业务进程画像的目的。
- 快速识别异常业务进程，并与生产系统的集群调度关联起来。

### 第三题：工程化

- 目前，diagnose-tools更多的是一款单机诊断工具，如何实现集群内的异常识别，并及时启动diagnose-tools工具抓取异常现场？