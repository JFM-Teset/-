# 系统简介

Azure RTOS ThreadX是专门为嵌入式应用程序设计的高性能实时内核。本章包含对该产品的介绍以及其应用和优点的描述。



## ThreadX数据类型

除了自定义ThreadX控件结构数据类型外，在ThreadX服务调用接口中还使用了一系列特殊数据类型。这些特殊的数据类型直接映射到基础C编译器的数据类型。这样做是为了确保不同C编译器之间的可移植性。在 ***tx_port.h*** 文件中可以找到定义.

以下是ThreadX常用服务调用数据类型及其相关含义的列表：

| UINT  | 基本无符号整数。 此类型必须支持8位无符号数据。 |
| ----- | ---------------------------------------------- |
| ULONG | 无符号长型。 此类型必须支持32位无符号数据。    |
| VOID  | 几乎总是等同于编译器的void类型。               |
| CHAR  | 最常见的是标准的8位字符类型。                  |

[更多支持](azure.com/rtos)



## ThreadX的独特功能

与其他实时内核不同，ThreadX具有通用性——通过使用功能强大的CISC，RISC和DSP处理器的应用程序，可以轻松地在基于小型微控制器的应用程序之间扩展。(Unlike other real-time kernels, ThreadX is designed to be versatile easily scaling among small microcontroller- based applications through those that use powerful CISC, RISC, and DSP processors.)

ThreadX可基于其基础体系结构进行扩展。因为ThreadX服务是作为C库实现的，所以只有应用程序实际使用的那些服务才回运行。因此，ThreadX的实际大小完全由应用程序确定。 对于大多数应用程序，ThreadX的指令映像大小在2 KB至15 KB之间。



### picokernel™架构

ThreadX服务没有像传统的微内核体系结构那样相互叠加，而是直接插入其核心。这导致最快的上下文切换和服务呼叫性能。 我们称这种非分层设计为picokernel体系结构。

### ANSI C源代码

ThreadX主要是用ANSI C编写的。只需少量的汇编语言即可为目标处理器定制内核。这种设计使得可以在很短的时间内（通常在数周之内）将ThreadX移植到新的处理器系列中！

### 先进的技术

以下是ThreadX高级技术的重点：

* 简单的picokernel架构
* 按需加载(占用资源少)
* 确定性处理
* 快速的实时性能
* 抢先式和合作式调度（Preemptive and cooperative scheduling）
* 灵活的线程优先级支持（32-1024）（Flexible thread priority support (32-1024)）
* 动态系统对象创建
* 无限数量的系统对象
* 优化过的中断处理
* 抢占阈值™（Preemption-threshold™）
* 优先继承
* 事件链™
* 快速软件计时器
* 运行时内存管理
* 运行时性能监控
* 运行时堆栈分析
* 内置系统跟踪
* 强大的处理器支持
* 大量的开发工具支持
* 完全中性（Completely endian neutral）

### 不是黑匣子

ThreadX的大多数发行版都包含完整的C源代码以及特定于处理器的汇编语言。这消除了许多商业内核中出现的“黑匣子”问题。 使用ThreadX，应用程序开发人员可以确切地看到内核在做什么——没有秘密可言。

源代码还允许对特定应用程序进行的适配。 尽管不建议这样做，但是如果绝对需要的话，具有修改内核的能力当然是有益的。对于习惯于使用自己的内部内核的开发人员而言，这些功能特别令人欣慰。他们希望拥有源代码和修改内核的能力。 ThreadX是此类开发人员的终极内核

### 符合MISRA C

MISRA C是使用C编程语言的关键系统的一组编程准则。 最初的MISRA C指南主要针对汽车应用。 但是，MISRA C现在被广泛认为适用于任何安全关键型应用。 ThreadX符合MISRA-C：2004和MISRA C：2012的所有“必需”和“强制性”规则。 ThreadX还符合除三个“建议”规则之外的所有规则。 有关更多详细信息，请参考ThreadX_MISRA_Compliance.pdf文档。



## 嵌入式应用

。。。。。。



