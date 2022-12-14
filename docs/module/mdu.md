## 乘除法器

乘除法模块是处理器中的重要运算部件，关于乘除法指令的扩展，可以参考《计算机组成原理课程设计实验指导书上》附录7的内容和《计算机组成原理课程设计实验指导书补充内容》进行扩展。指导书中给出了利用vivado IP实现乘除法器的方式。

在实现乘除法器时请注意流水线的控制信号的给出。如果实现异常的话，请思考异常处理情况下流水级级间寄存器刷新对于乘除法器的影响。

对于希望完成难度更高的扩展的同学，可以考虑自己实现乘除法器，下面是对于自己实现乘除法器的思路的简单介绍。

### 乘法器

众所周知，乘法就是多次加法。将多个组合逻辑加法器直接叠在一起，就能得到乘法器的结果，然而此种实现延迟极高，因为进位链极长，会造成处理器主频的大幅下降。同时这种方式占用面积巨大，对于32位处理器，其乘法结果需要32x32个全加器才能得到。在实际处理器中，并不会出现数逻讲义中介绍过的那种阵列乘法器。

最为简单高效的做法是实现一个迭代乘法器，将32位加法拆解为32次被乘数的累加。这种方式可以有效的避免主频下降的问题，然而乘法的超多的周期数，则会导致处理器一直被乘法指令阻塞，造成乘法指令1条更比32条强的周期浪费。在计组课程中，曾经介绍过一种乘法，称为两位Booth乘法，同样是对补码进行运算，两位Booth乘法只需要一半的周期数就可以完成乘法。可以大大加速的乘法器的效率。

两位Booth乘法将32个加法转换为16个部分积的和，通过华莱士树可以将其转换成两个数的加和。通过华莱士树乘法器可以实现比阵列乘法器的更小的面积和更低的时延。

华莱士树乘法器的时延和面积虽然更小，然而其时延对于高性能处理器来说还是比较高的。常用策略是对华莱士树乘法器进行流水化，从而降低华莱士树乘法器的时延。

限于时间，本部分对于乘法器只能泛泛介绍，具体的原理与实现方法，可以参考[第 8 章 运算器设计 | 计算机体系结构基础 (foxsen.github.io)](https://foxsen.github.io/archbase/运算器设计.html#定点补码乘法器)

一位迭代乘法器示例伪代码如下，如果你要采用下面的设计，请自己思考。

!> 此代码不能直接运行

```verilog
module Mult(
    input clk, 
    input rst,
    input [31 : 0] oprand1,// X
    input [31 : 0] oprand2,// Y
    input signbit, // is 2's component or not
    input en ,     // the enable signal of multiplier
    output [63 : 0 ] result, // X * Y
    output done
);

    parameter IDLE = 1'b0;
    parameter CALCULATING = 1'b1;

    reg state;
    reg [65 : 0] multiplicand;
    reg [33 : 0] multiplier;
    reg [65 : 0 ] result_r; 
    wire [65 : 0] parital_product;
    wire carry;
    
    // the state machine
    if (rst)begin
        state <= IDLE;
    end
    else case(state)
        IDLE:
            if(en)begin
                state <= CALCULATING;
            end
        CALCULATING:
            if(!en)begin
                state <= IDLE;
            end
        default:
            state <= IDLE;
    endcase  
    
    // the multiplicand
    case(state)
        IDLE:
            if (en)begin
                multiplicand <= {signbit&oprand1[31], oprand1};
            end
        CALCULATING:
            multiplicand <= multiplicand << 1;
    endcase
    
    // the multiplier
    case(state)
        IDLE:
            if (en)begin
                multiplier <= {signbit & oprand2[31], oprand2, 1'b0}; // add zero to multiplier as y_{-1}
            end
        CALCULATING:
            multiplier <= multiplier >> 1;
    endcase
    
    // calculating the result
    case(state)
        IDLE:
            if (en)begin
                result_r <= 0;
            end
        CALCULATING:
            if (!done)begin
                result_r <= result_r + parital_product + carry;
            end
    endcase 
        
    assign result = result_r[63 : 0];
    // generate the -X
    assign parital_product = multiplier[1:0] == 2'b01 ? multiplicand :
                              multiplier[1:0] == 2'b10 ? ~multiplicand  :
                                                 0;
    assign carry = multiplier[1:0] == 2'b10 ? 1'b1 : 1'b0;
    assign done = multiplier == 0;
    
endmodule
```



### 除法器

#### 无符号除法器

电路实现除法器的方式与手工计算的方式类似，也就是试商法。尝试用被除数减去除数，判断是否够减，如果够减，就从被除数中减去除数。

如果不够减呢？如果减了，可以下个周期再加回来，也可以这周期就不做减法，这两种策略分别是恢复余数法和不恢复余数法。这里我们采用恢复余数法，换言之，当不够减时不会执行减法操作。

当出现除法请求时，第一个周期我们可以先将被除数送入一个64bits的寄存器A的低位，高位补0（回想一下竖式减法的运算，是不是也有这种补0的操作）。

此后迭代32个周期的减法，每个周期取A的高33位和B高位补零的结果做减法，根据结果是否是负数，判断是否够减，如果够减，该位除数结果为1，反之为0，对于被除数来说，如果够减，则将A减去B，最后将A寄存器左移一位，回到此步骤开头。

经过32个周期之后，数据运算完毕，此时的被除数A即为计算得到的余数，可以将余数和商写入相关寄存器或送至下一流水级。

#### 有符号除法器

对于有符号除法器，可以通过通过将其转化为无符号除法器的方式实现，可以将被除数和除数都转换为其绝对值，最后得到的结果再根据其符号进行调整。

####  更先进的除法器

乘法满足交换律，乘法器可以通过并行处理部分积的方式，但是除法不同，没有办法通过类似的方式实现。迭代法的一位一位的计算方式较慢，现代的高性能处理器中可能会采用基于SRT-4的除法器。由于除法器往往不是处理器的性能瓶颈，所以一般来说完成一个普通的迭代法的除法器就足够，感兴趣的同学可以进一步了解相关知识。