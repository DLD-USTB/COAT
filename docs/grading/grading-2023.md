## 任务要求

### 个人任务

#### CG 测评任务 `20分`

1. 完成CG上12 + 8 共计 20 道评测题，需要提交的代码已经在CDE工程中给出。请阅读实验指导书《计算机组成原理课程设计实验指导书 下》，学习TinyMIPS的基本结构。并提交相应代码
2. 20道评测题包括指导书第二章8道题，第三章12道题，在指导书的对应位置有说明该部分代码对应哪一道题目的评测。

校内用户访问网站通过[CG平台](http://202.204.62.165 ':crossorgin')进行访问。校外用户请通过[vpn](https://n.ustb.edu.cn)进行访问

#### 虚拟仿真实验平台`10分`

1. 完成虚拟仿真实验平台上的四个实验。包括模拟器运行和代码运行。
2. 本部分实验重点在于解决流水线的前递与暂停问题，与实验指导书的第二章结尾和第三章结尾有关。请注意体会其原理，在进行乘除法器的扩展过程中，同样需要利用到前递和暂停的知识。

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

---

### 团队任务


#### 创新基础部分 `20分`

创新部分基础实现包括《计算机组成原理课程设计指导书补充内容》所介绍的全部内容

* 乘除法指令扩展 `8分`
    * 通过乘除法测试点，包括`n44_div`,`n45_divu`,`n46_mult`,`n47_multu`,`n48_mfhi`,`n49_mflo`,`n50_mthi`,`n51_mflo` 
    * 乘除法模块可以采用IP核实现
* 异常相关指令扩展 `12分`
    * 第一阶段：`5分`
    	* 实现特权指令`ERET`,`MTC0`,`MFC0`
    	* 自陷指令`BREAK`,`SYSCALL`
    	* 实现A02文件中规定的所有CP0寄存器
    * 第二阶段：`7分`
        * 继续扩展异常，通过完整89个功能点的测试

#### 创新扩展部分 `10分`

* **缓存模块设计**

    Cache是现代计算机中不可缺少的技术

    * Cache基本实现为一路直接映射Cache，二者都实现时取DataCache成绩
    * Cache指令实现 
    * 多路组相连Cache 
    * Cache流水化 

* **外设功能扩展**

    * 通过汇编语言代码对SoC上已经提供实现了的外设进行操控 
    * 集成串口外设，并能通过汇编语言代码操纵外设

#### 实验报告——团队部分 `10分`

---

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
    - 本课程的总分100分，难度最大的部分其实在创新扩展部分的10分，而其他的部分都只要认真把握课程原理，即可实现，请同学们重点把握这90分的知识，充分理解
    - 不建议大家死磕创新扩展部分的实现，**更禁止通过抄袭的方式获得这一部分分数**，请大家实事求是