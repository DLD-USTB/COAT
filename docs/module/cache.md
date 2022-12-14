## Cache原理与设计

如果你曾有心观察过TinyMIPS的仿真波形，你或许会发现，TinyMIPS执行一条指令的周期其实远远长于五个周期。这是因为TinyMIPS通过AXI接口访问内存时，会需要耗费多个周期的时间，等待存储器响应访存请求。面对这种需要多周期才能返回的访存操作，当处理器发出的访存请求尚未被接受时，处理器采用的策略是进行暂停。因而，在TinyMIPS的运行过程当中有大量的时间会耗费在因为访存而引起的暂停上。

在真实处理器中，CPU对存储的访问也会耗费大量的时间。CPU运行的太快，存储器访问的太慢，因而存储器成为了制约处理器发展的关键路径。对此，计算机科学家们提出了两种主要的方案，缓存和预取。

预取策略是提前发出访存请求，通过此种方式可以一定程度上隐藏访存的时延，然而对于我们的TinyMIPS来说，预取实现的效果有限：访存时延相较于指令执行时间长得多。

缓存策略则是将一部分的数据项存入CPU上的高速存储单元上，这些访问速度更快，往往很短的时钟周期就可以获取到数据，存储单元中存储了CPU经常使用的数据项。对于TinyMIPS来说，实现缓存可以很大程度上提升处理器的性能。

现代的处理器中，往往是两种策略同时使用，尽可能将CPU访问存储器的成本降低。

### 局部性原理

局部性原理是计算机系统中的重要原理。包括时间局部性和空间局部性。

所谓时间局部性是指，存储器中的一个数据如果被访问了，那么，以后很可能会再次访问该数据。时间的局部性有很多例子，比如程序中常见的循环结构，一段时间内指令访问都会集中于对循环体内指令的访问。当我们访问变量时，一个变量往往还会在后面被重新使用。

所谓空间局部性是指，存储器中的一个数据如果被访问了，那么，它周围的数据也很可能被访问到。在取指时，当我们访问了一条指令时，大概率会访问其后的那条指令。当我们访问呢全局数组，或是栈上的内存空间时，也很大程度上会访问到一段连续的地址空间。

### 高速缓存

缓存（Cache）是一种广泛用于计算机系统中的方法，将需要耗费高额成本获取的数据存放在一个易于访问的地方。比如说Vivado的工程中就有对于IP的Cache，QQ微信等软件中也有对于网络下载的图片的Cache，操作系统中也会有对文件的Cache。

所以，计算机访问存储器需要耗费巨大的时间，将内存数据的一部分备份到一种高速存储介质中是一种很容易想到的方法。那么，放在什么地方呢？

很容易想到的一种介质就是寄存器堆。寄存器堆可以在当拍返回数据，访问速度极快。但是如果我们想要缓存更多的资源，也就意味着更大的寄存器堆和更差的时序，因为我们的读写的控制逻辑变得复杂了（想想寄存器堆对应的电路）。另一方面，一位的寄存器耗费的硬件资源也比较高，往往需要耗费较多的晶体管才能组成一个一位的寄存器。在一般的CPU中，片上的高速缓存采用的往往是SRAM，他占用的面积更小，一般来说其时序也更好。

在FPGA上，我们可以考虑使用其片上专用的存储资源BRAM（Block Random Access Memory）。对于大容量的存储，采用BRAM在时序上更有优势，可以生成更高性能的电路。但是BRAM的访问往往需要一个周期，即发出访存请求后，会在下一个周期内才会返回结果，在用BRAM做Cache的时候一定要注意这一点。

### Cache是一张表

根据空间局部性原理，我们使用一个数据项后，我们往往会使用到其相邻的数据项，因此，我们把内存划分为若干的Block，每个Block内有多个数据。当我们讨论内存数据在Cache中的存储时，我们实际上时讨论各个Block在Cache中的存储。

Cache事实上就是一张由Block中数据组成的表，当处理器访存时，会去访问Cache表，读取Cache表中的数据。在数据之外，每个Block还需要一些其他的元数据，这些元数据会帮助处理器确认Cache与内存中数据的映射关系。试想一下，Cache不过是处理器中的一个容量非常小的存储单元，Cache的容量往往只有KB级别，然而内存的空间则是GB级别。Cache与内存的巨大容量差异巨大，因此Cache这张表中，每个表项可以存放来自不同内存单元的数据，决定每个表项能放什么数据，将Cache的结构划分成三种基本结构——全相联Cache，组相联Cache，直接映射Cache。

#### 全相联Cache

我们首先考虑一种简单的Cache——全相联Cache。但处理器访问Cache的一个Block时，如何确定这个Block是内存中的哪个单元呢？答案很简单，只需要将该Block的地址也存在Cache表当中，作为元数据即可，我们这个地址称为Cache的tag。当我们访问Cache时，通过与该地址进行比较，即可判断这个Cache块是否是我们需要找的。除了tag之外，cache中还需要另一种元数据，该数据是Valid位。当我们处理器复位时，如果不对Cache进行初始化（加载一些内存数据到Cache当中），Cache中的每一项都是无效的，为了表示这种无效状态，我们需要Valid位，当访问Cache的一个Block时，我们会比较当前需要访问的地址和Cache表的Valid位，判断该项是否为我们需要的Block。

对于由一个Block，TAG，Valid位构成的Cache表中的一行，我们称之为一个Cache line。一个全相联Cache中绝不会只有一个Cache line，当我们访问一个全相联Cache时，要如何判断Cache中是否有我们需要的数据呢？显然，必须要将这多个Block的TAG都去做一遍比较，在硬件上，就是多个并行的比较器。然后我们根据每个比较器的结果的或，判断Cache是否命中。然后，根据命中的结果，从多个Cache的Block中选出一个数据。

![image-20221026213054347](module.assets/image-20221026213054347.png)

可以看到，上图将一个访存地址划分成为了两个部分，一个是TAG，一个是Block Offset。我们前面提到，一个Block中会存放多个字的数据，所以访问一个Block时，需要判断的是Block的地址。然后只需要根据Block Offset就可以判断出，该访存地址到底需要的是Block中的哪个数据。

![image-20221026214012644](module.assets/image-20221026214012644.png)

全相联Cache的一般会设计的比较小，因为如果设计的太大，意味着更多的TAG比较器，更高的时延和更大的开销。

#### 直接映射Cache

为什么全相联Cache需要如此之多的比较器呢？因为处理器的访存请求可能在全相联表中的任何一个Cache line中，那么，如果我们有一种方式，让一个Block只能存放在Cache表中的固定位置，这样每次去比较时，就只需要取出对应位置的tag，然后使用一个比较器就可以判断是否命中。

那么如何判断一个Block应该放在Cache表的哪里呢？这就可以引入直接映射Cache。

直接映射Cache将TAG段的尾段称为index。通过index指示，该地址想要访问的Block究竟在Cache表中的哪一行。

![image-20221026214901527](module.assets/image-20221026214901527.png)

因此，当我们访问Cache时，首先会根据访存地址得到Tag和Index，然后我们会根据Index找到要访问的Cache line，然后将访存地址tag与该cache line中的tag进行比较，判断是否一致且该cache line是否有效。

Index的宽度事实上受限于我们的Cache大小决定，如果Cache有64个line，那么index的宽度就应该是6。

直接映射Cache非常简单，一个数据项在Cache表中位置是固定的，所以直接相连Cache甚至没有替换算法，当两个数据项的地址在Cache表中对应着同一个位置时，新来的数据项就必须要将旧的数据项替换掉。

但是如果有下面这样一种情形，CPU频繁访问的两个数据项，映射到了Cache中的同一个位置，此时就会触发频繁的Cache替换，这种现象我们称为抖动。

#### 组相联Cache

既然Cache表的任意位置都可以随意放置的全相联Cache访问效率上会比较低，而直接映射这种Block只能位于Cache表一处的Cache会导致抖动问题。那么是否可以考虑折中一下，让一个Block可以放在Cache表中的固定几个位置。

组相联Cache就是这样一种Cache，组相联同样通过index来确定一个Block存放的位置，但是同一个Index，可以有多个可以存放的位置，每一个位置都可以放一个Cache line，这些Cache line称为一个Cache Set。

组相联Cache可以理解为多个直接相连Cache的组合，每个这样的直接相连Cache，称为一个Cache way。如果一个index对应的数据能够放到n个位置上，就称其为n路组相联。下面是一个二路组相联的例子。

![image-20221026224456246](module.assets/image-20221026224456246.png)

一般来说，Cache的way的数量越多，其抖动的现象越会减轻，但相应的Cache的访问效率也会降低。在Cache的大小一定时，当2^Index = n时，组相联Cache就会演变为全相联Cache，而当n=1时，Cache就会演变为直接映射Cache。

### Uncache与Cache一致性

Cache能够一定程度上解决存储器速度与主存之间差异过大的问题，然而Cache也引入了新的问题，也就是一致性问题和对于外设访问的问题。

#### Cache写入

一般来说，我们会将CPU直接访问的L1 Cache划分为I-Cache和D-Cache，分别是Instruction Cache和Data Cache，我们的I-Cache只有读功能，然而，D-Cache则兼具读取与写入的功能。当我们向D-Cache进行写入操作时，我们是否需要将当前的写入操作，更新到下级存储器呢？我们将更新和不更新下级存储器这两种不同策略分别称为写穿法（Write Though）和写回法（Write Back）。

写穿法，意味着下级存储器中始终都是最新的内容，没有D-Cache与内存不一致的问题。然而每次对D-Cache的写入，都会导致需要触发一次对总线的写操作，耗费的时钟周期更多。

写回法，表示Cache的写入并不会直接更新内存当中的内容，而是将修改暂存在Cache中，如果发生Cache Miss的情况时，需要将被修改的Cache line写回到存储器中，保证正确性。为了标记一个Cache line是否发生了修改，每个Cache line需要增加一个Dirty位，如果该位使能，则意味该Cache line被修改了，该页是脏页，当发生该Cacheline的换入换出时，需要将修改后的内容写回到下级存储器。

Cache写入的过程中，也同样会遇到Cache Miss，对于Miss的Cache line，也同样有两种方式解决。一种思路是，既然Cache line已经miss，直接将当前的数据写回下级存储器就好，这种思路称为Non-Write Allocate。而另一种与之相对的思路Write  Allocate，是，先将对应的Data Block从内存加载进来，与要写入的数据合并，然后存入Cache中。

一般来说，Write though和Non-Write Allocate会配合使用，Write back和Write Allocate配合使用。

#### Uncache

当我们访问外设时，如果访存请求直接由Cache代理会怎么样呢？如果我们发起来一个对外设的读请求，Cache如果命中，就会从返回一个过去的已有的数据，没有时效性。如果我们发一个写请求，那么如果Cache采用了写回法，那么这个对外设的命令就没办法发送到对应的外设上。

因此，对于对外设的访问需要实现Uncache，通过内存地址判断出一段地址是Cache区的数据或是Uncache区的数据。从而决定将访问请求发送到总线上还是Cache上，这一点可以通过一个简单的选通器实现。

#### Cache一致性

Cache实际上是一个内存数据中的备份，而备份往往会带来不一致问题，如果我们修改了多个备份中的一个，但是没有修改其他的备份，就会造成不一致的情况，这就是一致性问题。

在划分为I，D两种Cache的我们的处理器当中，面临着何种一致性问题呢？不考虑其他复杂可以修改内存的外设（如DMA），我们的CPU对内存的修改只能通过对D-Cache的写入解决。对于D-Cache的写入，可能未修改的副本就有内存和I-Cache两种。CPU对内存的读写直接通过Cache代理，因此我们主要关注I-Cache与D-Cache的一致性问题。

如果D-Cache和I-Cache存储的内容映射到了同一块内存区域，此时如果D-Cache中写入内容，那么I-Cache的内容却没有同时修改，这就会影响CPU的正确性。这种操作被称为自修改操作，当发生自修改时，为了维护Cache的一致性，一般有软件处理和硬件处理两种操作，软件处理就是通过Cache指令来实现。

![image-20221030221238934](module.assets/image-20221030221238934.png)

op与Cache操作的关系如下：

| OP | 操作名称 | 地址使用方式 |功能描述|
| ---- | ---- | ---- |----|
| 0b00000 | Index Invalid | Index |用Index索引到I-Cache的Cache行，将该行无效|
| 0b01000 | Hit Invalid | Hit |用有效地址查找I-Cache，若Cache Hit，则将Hit的改行置为无效|
| 0b00001 | Index Writeback Invalid | Index |用Index索引到D-Cache的Cache行，若该行有效且脏，则将Cache行写回内存然后使该行无效，否则直接使得该行无效|
| 0b10001 | Hit Invalid | Hit |用有效地址索引到D-Cache的Cache行，无论该行是否有效且脏，直接使该行无效|
| 0b10101 | Hit Writeback Invalid | Hit |用有效索引到D-Cache的Cache行，若该行有效且脏，则将Cache行写回内存然后使改行无效，否则直接使得该行无效|

> 计组课设中不会出现自修改现象，所以可以不实现

### Cache设计

一般来说，处理器的L1 Cache会分成I-Cache和D-Cache两个部分，I-Cache负责缓存指令，D-Cache负责缓存数据。对于实际设计中，I-Cache会更为简单，因为不用考虑I-Cache的写入，在计组课设当中，I-Cache也可以不做uncache。以下部分一个非常简单的Cache为例——一个直接映射的阻塞式状态机I-Cache。

Cache的主体是Cache状态机，AXI总线控制器和Cache表三个部分。对于我们的实现来说，可以按照以下的步骤

* 确定状态机
* 根据状态给出各个时序部件的控制信号

#### 总线接口

I-Cache事实上兼具两个功能，一个是为CPU返回数据，一个是发起总线请求。

发起总线请求的功能在原本的TinyMIPS中的AXI-Adapter就已经实现，该模块将CPU的访存请求直接转化为AXI上的访问请求。在实现Cache的过程中，为了降低单个数据项的访存成本，我们采用了Burst模式，一次访存可以从内存中读取多个数据项。对于总线的访问，可以单独设计成一个状态机，而不用将其与Cache的状态机融合。

#### Cache表设计

在这里我们设计了一个4KiB大小的I-Cache，一个Cache line存放8个字的数据项，换言之一个Cache line的数据大小为32B，需要五位的寻址空间。Cache大小为4KiB，因此Cache中应当有128个Cache line，index的空间应当是$log_2(128)=7$, Tag的大小为32位地址减去7位index和5位的block offset，共20位。

对于Data部分，我们可以将所有的Cache数据都放在一个BRAM块中，也可以根据Block Offset，设计8个bank，每个bank中存放一个Cache数据，Tag采用另一个BRAM实现。Valid位采用寄存器堆实现。

#### 处理器暂停

Cache的访问可能会出现MISS，我们对Cache的访问也需要多于一个周期访问。因此，Cache的访问需要类似于AXI-Adapter的机制，从而暂停处理器的执行，暂停并不需要像AXI-Adapter那样暂停整个处理器，可以只暂停相关的流水线阶段。

#### Cache状态机

我们将Cache的工作分为四种状态，当没有处理任何来自CPU的访存请求时，其状态是IDLE态，对Cache进行查找判断是否命中是LOOKUP态，换入Cache数据是DATA态，对TAG和Valid位的更新是最后的TAG态

IDLE阶段Cache接受来自外部的访存请求。这个阶段需要给BRAM中的Tag和data发出访存请求。

LOOKUP阶段判断CPU的访问请求是否命中，这个阶段会获取到BRAM中的数据。

TAG态更新BRAM中的TAG，以及Valid位。

#### Cache代码实现

主状态机的伪代码如下：

```verilog
reg cache_state;
case(cache_state)
    IDLE:if(sram_rd_en) cache_state <= LOOKUP;
    LOOKUP:if(cache_miss)cache_state <= REPLACE;else cache_state <= IDLE;
    REPLACE:if(axi_rvalid && axi_rready && axi_rlast)cache_state <= TAG;
    TAG:cache_state <= IDLE;
endcase
```

AXI状态机的伪代码如下：

```verilog
reg axi_rd_state;
case(axi_rd_state)
    IDLE:if(cache_miss) axi_rd_state <= ADDR;
    ADDR:if(axi_arvalid && axi_arready)axi_rd_state <= ADDR;
    DATA:if(axi_rvalid && axi_rready && axi_rlast)axi_rd_state <= DATA;
endcase
```

两个状态机和Cache表是Cache设计中的主要时序器件，其他还有对地址的锁存等，此处不再赘述。

根据状态机的状态，我们可以生成对CPU和总线的信号与对BRAM的控制信号。下面以Tag BRAM的控制为例展示如何分析控制信号的生成方式。

```verilog
sram_128x20 tag_bank(
    .addra      (tag_index    )  ,
    .clka       (clk             )  ,
    .dina       (miss_tag        )  ,
    .douta      (cache_tag       )  ,
    .ena        (tag_en          )  ,
    .wea        (write_tag_en    )   
); 
```

当CPU发送过来一个访存请求时，我们需要对其访存地址进行拆分。

```verilog
assign {req_rd_tag,req_rd_index,req_rd_block_offset} = req_addr[31 : 2];
```

其中，tag用于判断命中，index用于访问data部分和对应的tag.

* 一个访存请求来时，tag BRAM的地址应该为何呢？有两个时段我们需要对tag做操作，当IDLE的时候，我们需要发出读请求，当TAG时，我们需要写入BRAM，二者访问的地址是相同的，都来自于cpu发起的请求地址。

  ```verilog
  assign tag_index = req_rd_index;
  ```

* 何时需要使能tag BRAM? IDLE态有访存请求时需要使能，TAG态写入时需要使能。

  ```verilog
  assign tag_en = (cache_state == IDLE) && sram_rd_en || (cache_state == TAG) ;
  ```

  > 但仔细思考一下，在其他状态中Cache是否就必须要保持不使能呢？其实不然，因此这部分的逻辑也可以更加简化。

*  何时需要写入tag BRAM?仅有TAG态需要使能，此时需要写入数据

  ```verilog
  assign write_tag_en = cache_state == TAG;
  ```

* 重新填回tag BRAM的内容是什么呢？

  ```verilog
  assign miss_tag = req_rd_tag;
  ```

* tag BRAM中输出的tag，其作用是判断Cache是否命中，判断方法如下

  ```verilog
  assign cache_hit = (req_rd_tag == cache_tag) && valid[req_rd_index];
  ```

限于时间关系，对于其他的控制信号的生成也可以按照类似的方式处理，分析在某个状态下改信号应当为何种值，通过代码将这种分析结果表达出来即可。

---

#### *不用总线的Cache*

> 是的，cache也可以不实现总线，如果将cache读取数据的方式改为AXI_adapter里直接读取，就可以越过复杂的处理。这种cache的基本原理仍是局部性原理，因为你过去访问到的数据未来仍可能被用到。但是你不再能够充分利用Burst的优势降低单个数据项的访问时延。
> 
> 而对于Cache表的设计，你仍然能够采用上面介绍的设计Cache表的方法，但是你的单个Block大小就是1个字。由于此种Cache做的会比较小，所以你可以考虑采用直接采用寄存器堆实现，不用处理复杂的Cache表逻辑。
> 
> 实践证明，此种Cache虽然比较简陋，但是仍旧能对CPU性能起到一定的提升。
