# 实验环境使用教程

### 工程结构

```
├── abstract-machine
├── am-kernels
├── asm-test
│   ├── Makefile
│   ├── build
│   │   ├── hello.bin
│   │   ├── hello.elf
│   │   ├── hello.o
│   │   ├── hello.s
│   │   └── mem.data
│   └── tests
│       └── hello.S
├── script
│   └── dumphex.py
├── vsrc
│   ├── Core
│   │   ├── ALU.v
│   │   ├── ALUct.v
│   │   ├── Control.v
│   │   ├── Core.v
│   │   ├── IFU.v
│   │   └── RegFile.v
│   ├── Memory
│   │   ├── Memory.v
│   │   ├── Router.v
│   │   └── SimDevice.v
│   ├── SimTop.v
│   └── Top.v
├── Makefile
```

* abstract-machine: 来自NJU，是为裸机程序提供的一系列的运行时环境
* am-kernels：来自NJU，包含着对于CPU与AM外设的测试代码
* asm-test: 提供了基本的测试虚拟串口的Hello.S代码，同时各位同学也可以在其中加入自己的代码通过文件夹下的编译环境进行实验
* script:主要是一些基本的脚本，当前仅包括用于将binary程序转换为readmemh函数可读的文件的python脚本
* vivado: vivado仿真工程脚本文件
* vsrc：CPU的主体代码
  * Core：处理器核的代码，对应处理器核部分实验指导书的内容
  * Memory：内存与外设的代码
  * SimTop.v：仿真文件的顶层
  * Top.v：例化Core与Memory的文件顶层，上板工程会以该文件作为顶层
* Makefile: 主体的Makefile代码

### 实验环境要求

实验推荐在Linux操作系统下进行。处理器的仿真同样也可以在Windows下通过Vivado执行，也可以安装IcarusVerilog进行。但是如果你需要利用实验环境中提供的工具链对于汇编代码进行编译，你需要安装MIPS的交叉编译环境，对于采用APT作为包管理器的Linux系统，可以通过以下命令来执行

```shell
$ sudo apt install gcc-mips-linux-gnu
```

该模块会将交叉编译工具链安装到你的电脑上，运行AM或asm-test下的Makefile脚本，即可编译出相应的程序

### 仿真环境搭建

对于使用Vivado仿真环境的同学，我们提供了基于Vivado的仿真工程，同时也提供了Vivado仿真工程的构建脚本`create_project.tcl`

如果你使用Vivado2018.3版本，你可以直接打开位于vivado/project下的xpr文件，如果你使用其他版本的vivado，你可以选择升级vivado工程或是通过在vivado目录下重新运行create_project.tcl文件，关于tcl脚本的使用，可以直接上网查到。进入vivado工程后，按照正常流程仿真即可，不过需要注意的是，使用vivado仿真器，需要将mem.data放到正确的位置。

对于Linux下使用IcarusVerilog的同学，只要你完成了IcarusVerilog的安装，在实验根目录下运行下面的指令进行仿真。

```shell
$ make
```

mem.data文件直接放在生成的build文件夹下即可

### asm-test使用说明

asm-test是一套不依赖于AM运行时库，与MIPS指令集紧耦合的编译脚本，其中额外提供了hello.S文件，通过下面的命令即可编译得到汇编文件对应的二进制文件和用于初始化仿真器内存的mem.data文件

```shell
$ make NAME=hello
```

如果需要增加自己写的汇编文件，将汇编文件保存在asm-test的test文件夹下，命名为`<your-filename>.S`，在asm-test路径下运行以下的文件即可得到相应的mem.data文件

```
$ make NAME=<your-filename>
```

