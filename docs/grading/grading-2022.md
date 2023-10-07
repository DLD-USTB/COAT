## 任务要求

### 个人任务

#### CG 测评任务 `20分`

1. 完成CG上12 + 8 共计 20 道评测题，需要提交的代码已经在CDE工程中给出。请阅读实验指导书《计算机组成原理课程设计实验指导书 下》，学习TinyMIPS的基本结构。并提交相应代码
2. 20道评测题包括指导书第二章8道题，第三章12道题，在指导书的对应位置有说明该部分代码对应哪一道题目的评测。

校内用户访问网站通过[CG平台](http://202.204.62.165 ':crossorgin')进行访问。校外用户请通过[vpn](https://n.ustb.edu.cn)进行访问

#### 虚拟仿真实验平台`10分`

1. 完成虚拟仿真实验平台上的四个实验。包括模拟器运行和代码运行。
2. 本部分实验重点在于解决流水线的前递与暂停问题，与实验指导书的第二章结尾和第三章结尾有关。请注意体会其原理，在进行乘除法器的扩展过程中，同样需要利用到前递和暂停的知识。

虚拟仿真实验平台通过[虚拟仿真实验平台](https://www.ilab-x.com/details/page?id=6594&isView=true)进行访问

#### 扩展指令任务`20分`

1. 要求选择TinyMIPS CPU中未扩展的22条指令中的若干条进行扩展，具体包含的指令在表A1-1，对于ADD，SUB等涉及异常的指令，暂时可以不用扩展异常处理的内容。
2. 对于每一条指令，都有若干其所属的分组和一个基础分值。
3. 每个指令的基础分值根据扩展该指令时的工作量进行设置，具体见表A1-1
4. 如果选择了多条同分组的指令，自第二条起，其计算分值减少为基础分值的1/2。
5. 要求最终同学们完成计算分值总计10分的指令扩展。多做不加分。
7. 个人任务需要到现场进行验收，演示对指令的测试过程。验收现场会进行提问，提问内容不仅涉及下面的指令扩展的思路过程，也包括Trace机制，测试用例，MIPS指令集架构相关的知识。

*表A1-1*

|序号|指令名称|指令分值|指令分组|
|-|-|-|:-:|
|1|ADD|2|R型运算指令/加法指令|
|2|SUB|2|R型运算指令|
|3|NOR|2|R型运算指令|
|4|ADDI|2|I型运算指令/加法指令|
|5|ANDI|2|I型运算指令|
|6|ORI|2|I型运算指令|
|7|XORI|2|I型运算指令|
|8|SLTI|2|I型运算指令|
|9|SLTIU|2|I型运算指令|
|10|SRL|2|Shamt移位指令|
|11|SRA|2|Shamt移位指令|
|12|J|2|直接跳转指令|
|13|JR|3|直接跳转指令|
|14|LH|3|内存载入指令|
|15|LHU|3|内存载入指令|
|16|SH|3|内存存储指令|
|17|BGEZ|3|条件分支指令/BGE跳转|
|18|BGTZ|3|条件分支指令|
|19|BLEZ|3|条件分支指令|
|20|BLTZ|3|条件分支指令/BLT跳转|
|21|BGEZAL|3|条件分支指令/BGE跳转|
|22|BLTZAL|3|条件分支指令/BLT跳转|

> 例如：如果一个同学选择了ADD指令,ORI指令，ANDI指令,JR指令，LH指令，其总分值为2,2,1,3,3 共计 11分，可以达到验收要求。

#### 实验报告——个人部分`10分`

> 实验报告的具体评分细则待发布

### 团队任务


#### 创新基础部分 `15分`

创新部分基础实现包括《计算机组成原理课程设计指导书补充内容》所介绍的全部内容

* 乘除法指令扩展 `7分`
    * 通过乘除法测试点，包括`n44_div`,`n45_divu`,`n46_mult`,`n47_multu`,`n48_mfhi`,`n49_mflo`,`n50_mthi`,`n51_mflo` 
    * 乘除法模块可以采用IP核实现，自己实现可以获得更高的分数，详见[创新扩展部分](#创新扩展部分)
* 异常相关指令扩展 `8分`
    * 实现特权指令`ERET`,`MTC0`,`MFC0`
    * 自陷指令`BREAK`,`SYSCALL`
    * 实现A02文件中规定的所有CP0寄存器


#### 创新扩展部分 `上限30分`

创新扩展部分包括以下选题，创新扩展部分上限不超过30分，但可以多于15分。
> 换言之，你可以选择用部分创新扩展的分数，填补创新基础部分的分数。

* **乘除法扩展进阶**

    不使用Vivado的IP,自己实现乘除法模块，各种实现具体分值如下
    * 迭代除法器 `5分`
    * 迭代一位Booth乘法器 `3分`
    * 迭代两位Booth乘法器 `5分`
    * 华莱士树乘法器 `10分`
    * 流水化华莱士树乘法器 `15分`
    * 其他实现，请于答辩前至少一周联系助教或老师进行评分。

* **异常相关扩展进阶**

    基础部分并未要求实现所有的异常，所以你可以进一步扩展，扩展所有的异常处理
    * 通过测试用例中所有的异常指令 `5分`

* **缓存模块设计**

    Cache是现代计算机中不可缺少的技术
    * Cache基本实现为一路直接映射Cache，InstCache分值为`8分`，DataCache分值为`10分`，二者都实现时取DataCache成绩，其他实现在此基础上增加
    * Cache指令实现 `+3分`
    * 多路组相连Cache `+2分` 
    * Cache流水化 `+5分`
    * 其他实现，请于答辩前至少一周联系助教或老师进行评分。
    > 如果一个同学实现了I/D两个Cache，同时Cache实现了二路组相连，其分值为12分。

* **外设功能扩展**
    * 通过汇编语言代码对SoC上已经提供实现了的外设进行操控 `3分`
    * 集成串口外设，并能通过汇编语言代码操纵外设`5分`
    * 其他扩展思路，请于答辩前一周与助教或老师联系进行评分

* **移植Linux内核并运行** `50分`

    * 实现MIPS32 Release1 中除CP1、JTAG等指令之外的100条指令
    * 实现12种中断异常
    * 实现20个CP0寄存器

* 其他选题，可以跟助教或老师进行联系沟通

所有的关于选题的沟通结果，都会及时更新在网站上，对于分值设定认为有不合理的（**无论是否是原有选题**），都可以及时和助教或老师联系讨论


#### 实验报告——团队部分 `10分`
> 实验报告具体的评分细则待发布

### Q&A

+ 关于参考工程和指导书
    - CG上已经开放TinyMIPS Extend的参考工程，该工程采用IP核实现了乘除法器，实现了部分的异常处理
    - 现有的网页指导书会尽量多更新基础的伪代码指导
+ 关于答辩会提的问题
    - 主要会围绕着创新基础内容部分进行考察
    - 对于创新扩展部分，主要考察原理和实现思路
+ 关于对扩展创新的评判
    - 创新扩展的所有内容基本上都可以在课设提供的89个测试用例里进行测试
    - 创新扩展的等级评定：比如不知道自己实现的算不算标准的华莱士树。课设给出的扩展要求分层是很明显的，把重点放在原理与方法的掌握上即可
+ 关于评分与难度的非线性关系
    - 之所以存在这种非线性关系，主要是为了一般的同学能够拿到分数，对此感兴趣的同学可以深入。不喜欢这个方向就没必要卷计组课设了，抓紧时间学好其他专业课
+ 关于分数分布
    - 扩展的高难度部分只占到15分，讲解的后三节课也主要针对这15分，剩下的部分都是只要努力做下去就可以获得的
    - 而剩下15分因为入门的分数定的比较高，所以只要好好完成基本任务，理解清楚实验原理，就可以取得不错分数了
