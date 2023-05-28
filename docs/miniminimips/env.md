# 实验环境使用教程

### 获取实验工程

实验环境的代码位于[此处](https://github.com/DLD-USTB/minitinymips)
获取实验工程可以通过以下git命令获取

```shell
git clone https://github.com/DLD-USTB/minitinymips.git
```
如果出现网络问题，可以尝试使用ssh方式进行下载
```shell
git clone git@github.com:DLD-USTB/minitinymips.git
```
如果你需要NJU的AM环境, 请使用下面的命令
!> AM环境的使用不在本课程的要求内，但欢迎同学们尝试

```shell
git clone --recursive https://github.com/DLD-USTB/minitinymips.git
```
或者你可以在获取实验工程之后，使用下面的命令，通用可以获得AM环境
```shell
git submodule update --init 
```


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
├── vivado
│   ├── constr/
│   ├── project/
│   └── create_project.tcl
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
│   ├── Simulation
│   │   ├── SimDevice.v // 仿真设备文件
│   │   ├── SimTop.v // 仿真连线顶层
│   │   └── SimEnv.v // 仿真顶层文件
│   └── Top.v //上板用顶层文件
├── ready-to-run
│   ├── hello
│   │   ├── mem.data // 已经编译好的hello程序对应的mem.data文件
│   │   └── hello.s //  hello程序的反汇编代码，你可以通过该代码了解到该程序需要的指令
├── Makefile
```

* abstract-machine: 来自NJU，是为裸机程序提供的一系列的运行时环境
* am-kernels：来自NJU，包含着对于CPU与AM外设的测试代码
* asm-test: 提供了基本的测试虚拟串口的Hello.S代码，同时各位同学也可以在其中加入自己的代码通过文件夹下的编译环境进行实验
* script:主要是一些基本的脚本，当前仅包括用于将binary程序转换为readmemh函数可读的文件的python脚本
* vivado: vivado相关文件
  * constr: EGO1约束，上板用
  * project: vivado工程，内含一个已经准备好的vivado工程
  * create_project.tcl: 用于生成vivado工程的脚本 
* vsrc：CPU的主体代码
  * Core：处理器核的代码，对应处理器核部分实验指导书的内容
  * Memory：内存与外设的代码
  * Simulation: 支持仿真运行的相关文件
  * Top.v：例化Core与Memory的文件顶层，上板工程会以该文件作为顶层
* ready-to-run:为准备编译环境有困难的同学准备的已经完成编译的文件
* Makefile: 主体的Makefile代码

!> 注意，当前的工程尚**不支持**上板，请暂时忽略所有上板用的文件
### 实验环境要求

实验推荐在Linux操作系统下进行。需要以下的工具链

* python
* gcc-mips-linux-gnu
* make
* IcarusVerilog(选择vivado仿真的同学请忽略)

**实验环境一键安装**
```shell 
$ sudo apt install gcc-mips-linux-gnu make python iverilog 
```

实验的仿真

* 仿真在Linux下直接通过IcarusVerilog文件进行
* 处理器的仿真同样也可以在Windows下通过Vivado执行，也可以安装IcarusVerilog进行。

测试代码的编译和运行

* 编译需要Linux的GNU工具链，缺少物理机的同学可以选择机房电脑，Linux虚拟机，WSL，以及其他同学。
* 你需要安装MIPS的交叉编译环境，对于采用APT作为包管理器的Linux系统，可以通过以下命令来执行

```shell
$ sudo apt install gcc-mips-linux-gnu
```

该模块会将交叉编译工具链安装到你的电脑上，运行AM或asm-test下的Makefile脚本，即可编译出相应的程序

### 仿真环境搭建

#### 使用Windows下的Vivado
对于使用Vivado仿真环境的同学，我们提供了基于Vivado的仿真工程，同时也提供了Vivado仿真工程的构建脚本`create_project.tcl`

如果你使用Vivado2018.3版本，你可以直接打开位于vivado/project下的xpr文件，如果你使用其他版本的vivado，你可以选择升级vivado工程或是通过在vivado目录下重新运行create_project.tcl文件，关于tcl脚本的使用，可以直接上网查到。进入vivado工程后，按照正常流程仿真即可，不过需要注意的是，使用vivado仿真器，需要将mem.data放到正确的位置，这一点在[处理器核部分指导书](/miniminimips/old_guide)中已经介绍了。


#### 使用Icarus Verilog
对于Linux下使用IcarusVerilog的同学，只要你完成了IcarusVerilog的安装，在实验根目录下运行下面的指令进行仿真。

```shell
$ make
```

mem.data文件直接放在生成的build文件夹下即可

### asm-test使用说明

asm-test是一套不依赖于AM运行时库，与MIPS指令集紧耦合的编译脚本，其中额外提供了hello.S文件，通过下面的命令即可编译得到汇编文件对应的二进制文件和用于初始化仿真器内存的mem.data文件。

!> 请切换到`<实验环境根目录>/asm-test`后运行以下指令

```shell
$ make NAME=hello
```

如果需要增加自己写的汇编文件，将汇编文件保存在asm-test的test文件夹下，命名为`<your-filename>.S`，在asm-test路径下运行以下的文件即可得到相应的mem.data文件

```
$ make NAME=<your-filename>
```

