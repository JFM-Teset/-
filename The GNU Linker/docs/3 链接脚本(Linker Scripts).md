# 综述

每个链接都由一个链接描述文件控制。 该脚本使用链接器命令语言编写。

链接描述文件的主要目的是描述如何将输入文件中的节(**sections**)映射到输出文件中，并控制输出文件的内存布局。大多数链接描述文件仅此而已。但是，必要时，链接器脚本也可以使用下面描述的命令来指导链接器执行许多其他操作。链接器始终使用链接器脚本。 如果您自己不提供，则链接器将使用编译到链接器可执行文件中的默认脚本。您可以使用“ --verbose”命令行选项显示默认的链接描述文件。 某些命令行选项（例如“ -r”或“ -N”）会影响默认的链接描述文件。

您可以使用“ -T”命令行选项来提供自己的链接描述文件。 当您执行此操作时，您的链接描述文件将替换默认的链接描述文件。

您也可以通过将链接器脚本命名为链接器的输入文件来隐式使用链接器脚本，就像它们是要链接的文件一样。

## 3.1链接描述文件的基本概念

我们需要定义一些基本概念和词汇来描述链接描述文件语言。

链接器将输入文件合并为一个输出文件。输出文件和每个输入文件都采用一种特殊的数据格式，称为目标文件格式。每个文件称为目标文件。 输出文件通常称为可执行文件，但出于我们的目的，我们也将其称为目标文件。除其他事项外，每个目标文件都有节的列表。有时我们将输入文件中的一个部分称为输入节。 类似地，输出文件中的节是输出节。

目标文件中的每个节都有名称和大小。大多数部分还具有关联的数据块，称为节内容。一个节可能被标记为可加载（**loadable**），这意味着在运行输出文件时应将内容加载到内存中。一个没有内容的节可能是可分配的，这意味着应该在内存中预留一个区域，但是在此不应该特别加载任何内容（在某些情况下，必须将该内存清零）。既不可装载也不可分配的部分通常包含某种调试信息。

每个可装入或可分配的输出节都有两个地址。 第一个是VMA(虚拟内存地址 **virtual memory address**)。这是运行输出文件时该节将具有的地址。第二个是LMA(*load memory address*)，即**加载内存地址**。这是将加载该节的地址。 在大多数情况下，这两个地址是相同的。当它们可能不同时，一个例子就是：将数据段加载到ROM中，然后在程序启动时将其复制到RAM中（此技术通常用于在基于ROM的系统中初始化全局变量）。 在这种情况下，ROM地址为LMA，并且
RAM地址为VMA。

您可以将*objdump*程序与“ -h”选项一起使用，以查看目标文件中的各个部分。每个目标文件还具有一个符号列表，称为符号表。 符号可以定义也可以不定义。每个符号都有一个名称，每个定义的符号都有一个地址以及其他信息。如果将C/C ++程序编译为目标文件，则会得到每个已定义函数以及全局或静态变量的已定义符号。 输入文件中引用的每个未定义函数或全局变量都将成为未定义符号。

您可以使用nm程序或带有'-t'选项的objdump程序在目标文件中查看符号。

## 3.2链接描述文件格式

链接描述文件是文本文件。

您将链接脚本编写为一系列命令。每个命令要么是关键字（可能后面跟参数），要么是对符号的分配。您可以使用分号（**semicolons**）分隔命令。 空格通常被忽略。

通常可以直接输入诸如文件名或格式名之类的字符串。如果文件名包含逗号等字符，否则将用于分隔文件名，则可以将文件名用双引号括起来。无法在文件名中使用双引号字符。（*If the file name
contains a character such as a comma which would otherwise serve to separate file names,
you may put the file name in double quotes. There is no way to use a double quote character
in a file name.*）

您可以在链接器脚本中包含注释，就像在C中一样，由“/*”和“*/”分隔。和C一样，注释在语法上等同于空白。



## 3.3简单链接器脚本示例

最简单的链接器脚本只有一个命令：“SECTIONS”。使用“SECTIONS”命令描述输出文件的内存布局。“SECTIONS”命令是一个强大的命令。这里我们将描述它的一个简单用法。假设您的程序仅由代码、初始化数据和未初始化数据组成。它们将分别位于“.text”、“.data”和“.bss”部分。让我们进一步假设这些是出现在输入文件中的唯一部分。

对于这个例子，假设代码应该加载在地址0x10000，数据应该从地址0x8000000开始。下面是一个链接器脚本：

```
SECTIONS
{
    . = 0x10000;
    .text : { *(.text) }
    . = 0x8000000;
    .data : { *(.data) }
    .bss : { *(.bss) }
}
```

您可以将“SECTIONS”命令编写为关键字“SECTIONS”，然后是一系列符号分配和用大括号括起来的输出节说明。上述示例的“SECTIONS”命令中的第一行设置特殊符号“**.**”的值，该符号是位置计数器。如果您没有以其他方式指定输出部分的地址（后面将介绍其他方式），则该地址是从位置计数器的当前值设置的。然后，位置计数器按输出部分的大小递增。在“SECTIONS”命令的开头，位置计数器的值为“0”。

第二行定义了一个输出部分，'.text'。冒号是必需的语法，它可以暂时被忽视。在输出节名称后面的大括号中，列出应放入此输出节的输入节的名称。“*****”是与任何文件名匹配的通配符。表达式“*(.text)”表示所有输入文件中的所有“.text”输入节。

由于定义输出节“.text”时位置计数器为“0x10000”，链接器将输出文件中“.text”节的地址设置为“0x10000”。

其余行定义输出文件中的“.data”和“.bss”部分。链接器将在地址“0x8000000”处放置“.data”输出节。链接器放置“.data”输出节之后，位置计数器的值将为“0x8000000”加上“.data”输出节的大小。其效果是链接器将在内存中紧跟“.data”输出节之后放置“.bss”输出节。

必要时，通过增加位置计数器，链接器将确保每个输出部分具有所需的对齐方式。在此示例中，“.text“和”.data”节的指定地址可能满足任何对齐约束，但链接器可能必须在“.data”和“.bss”节之间创建一个小间隙。

就这样！这是一个简单而完整的链接器脚本。

## 3.4简单的链接器脚本命令

### 3.4.1设置入口点

在程序中执行的第一条指令称为入口点。可以使用ENTRY linker脚本命令设置入口点。参数是符号名：

```
	ENTRY(symbol)
```

有几种方法可以设置入口点。链接器将按顺序尝试以下每个方法并在其中一个方法成功时停止，以设置入口点：

* '-e'输入命令行选项；
* 链接器脚本中的**ENTRY(symbol)**命令；
* 特定于目标的符号的值（如果已定义）；对于许多目标，这是开始的，但是基于PE和BeOS的系统（例如）检查可能的输入符号列表，与找到的第一个匹配。
* “.text”部分第一个字节的地址（如果存在）；
* 地址0。

### 3.4.2处理文件的命令（Commands Dealing with Files）（待续）

**INCLUDE filename**

​	此时包括链接器脚本文件名。将在当前目录和使用'-L'选项指定的任何目录中搜索该文件。您可以嵌套调用以包含多达10个级别的深度。（Include the linker script filename at this point. The file will be searched for in the current directory, and in any directory specified with the ‘-L’ option. You can nest calls to INCLUDE up to 10 levels deep.）

​	您可以将INCLUDE指令放在顶层、MEMORY 或SECTIONS 命令或输出节描述中。

**INPUT(file, file, ...)**

**INPUT(file file ...)**

​	INPUT命令指示链接程序在链接中包括命名文件，就像在命令行上命名文件一样。







### 3.4.3处理目标文件格式的命令（待续）

几个链接描述文件命令处理目标文件格式。

**OUTPUT_FORMAT(bfdname)**

**OUTPUT_FORMAT(default, big, little)**







### 3.4.4为存储区分配别名

用Section3.7介绍的命令创建的存储区是可以添加别名的。每个名称最多对应一个内存区域。

```
	REGION_ALIAS(alias, region)
```

REGION_ALIAS函数为存储区域区域创建别名。这允许将输出节灵活地映射到存储区域。接下来举个例子。假设我们有一个用于带有各种内存存储设备的嵌入式系统的应用程序。它们都具有通用的易失性存储器RAM，可以执行代码或存储数据。有些可能具有只读、非易失性存储器ROM，允许代码执行和只读数据访问。最后一种变型是只读、非易失性存储器ROM2，具有只读数据访问和无代码执行能力。

我们有四个输出节：

	* .text program code;
	* .rodata read-only data;
	* .data read-write initialized data;
	* .bss read-write zero initialized data.

目标是提供一个链接器命令文件，其中包含一个定义输出部分的独立于系统的部分和一个将输出部分映射到系统上可用内存区域的依赖于系统的部分。我们的嵌入式系统有三种不同的内存设置A、B和C：

| Section  | Variant A | Variant B | Variant C |
| -------- | --------- | --------- | --------- |
| \.text   | RAM       | ROM       | ROM       |
| \.rodata | RAM       | ROM       | ROM2      |
| \.data   | RAM       | RAM/ROM   | RAM/ROM2  |
| \.bss    | RAM       | RAM       | RAM       |

符号RAM/ROM或RAM/ROM2表示该部分分别加载到区域ROM或ROM2中。请注意，.data节的加载地址在.rodata节末尾的所有三个变体中开始(Please note that the load address of the .data section starts in all three variants at the end of the .rodata section.)。

接下来是处理输出部分的基本链接描述文件。 它包括系统相关的linkcmds.memory文件，该文件描述了内存布局：

```
INCLUDE linkcmds.memory
SECTIONS
{
	.text :
	{
		*(.text)
	} > REGION_TEXT
	.rodata :
	{
		*(.rodata)
		rodata_end = .;
	} > REGION_RODATA
	.data : AT (rodata_end)
	{
		data_start = .;
		*(.data)
	} > REGION_DATA
	data_size = SIZEOF(.data);
	data_load_start = LOADADDR(.data);
	.bss :
	{
		*(.bss)
	} > REGION_BSS
}
```

现在，我们需要三个不同的linkcmds.memory文件来定义内存区域和别名。 三个变体A，B和C的linkcmds.memory的内容：

```
/*A						这里一切都进入RAM。*/

MEMORY
{
	RAM : ORIGIN = 0, LENGTH = 4M
}
REGION_ALIAS("REGION_TEXT", RAM);
REGION_ALIAS("REGION_RODATA", RAM);
REGION_ALIAS("REGION_DATA", RAM);
REGION_ALIAS("REGION_BSS", RAM);
```

```
/*B	程序代码和只读数据进入ROM。读写数据进入RAM。
初始化数据的映像将被加载到ROM中，并将
在系统启动期间复制到RAM。
*/
MEMORY
{
	ROM : ORIGIN = 0, LENGTH = 3M
	RAM : ORIGIN = 0x10000000, LENGTH = 1M
}
REGION_ALIAS("REGION_TEXT", ROM);
REGION_ALIAS("REGION_RODATA", ROM);
REGION_ALIAS("REGION_DATA", RAM);
REGION_ALIAS("REGION_BSS", RAM);

```

```
/*
C
程序代码进入ROM。 只读数据进入ROM2。 读写数据进入RAM。 初始化数据的镜像被加载到ROM2中，并将在系统启动期间复制到RAM中。
*/
MEMORY
{
    ROM : ORIGIN = 0, LENGTH = 2M
    ROM2 : ORIGIN = 0x10000000, LENGTH = 1M
    RAM : ORIGIN = 0x20000000, LENGTH = 1M
}
REGION_ALIAS("REGION_TEXT", ROM);
REGION_ALIAS("REGION_RODATA", ROM2);
REGION_ALIAS("REGION_DATA", RAM);
REGION_ALIAS("REGION_BSS", RAM);
```

如有必要，可以编写一个通用的系统初始化例程，以将.data节从ROM或ROM2复制到RAM中：

```
#include <string.h>
extern char data_start [];
extern char data_size [];
extern char data_load_start [];
void copy_data(void)
{
	if (data_start != data_load_start)
	{
		memcpy(data_start, data_load_start, (size_t) data_size);
	}
}
```

### 3.4.5其他链接器脚本命令 （待续）





## 3.5给符号赋值

您可以在链接描述文件中为符号分配值。 这将定义符号并将其放入具有全局范围的符号表中。