# 计算机组成原理课程设计实验

## 课程要求

!> 课程要求分为个人要求和团队要求。个人要求占60分，团队要求占40分。

具体要求见[验收要求](grading)


## 资源使用路线

1. **本站**。本站是基于计组课设实验安排，结合往年经验和其他学校相关课程的先进经验，旨在推进课程文档系统化规范化而构建的课程网站。本站以尝试对过去的实验资料文档进行串联，帮助大家快速的找到需要使用的文档，同时对其内容进行补充。同学们在后面的`扩展内容`中可以重点参考本站的内容，

2. **《计算机组成原理课程设计实验指导书 上》**。

   1. 本书讲义二，介绍了关于Verilog语言的使用，对于一些常用的电路模块都给出了很优秀的写法。我们非常推荐大家采用这种代码风格进行编码。

      !> 强烈建议同学们在进行实验前先阅读一下这个部分，规范自己的Verilog代码写法。

   2. 本书讲义三，介绍了CPU仿真环境的基本使用。这部分内容介绍了仿真环境的基本结构和开发的基本方法。有部分内容和当前的实验环境有一定差异，请注意甄别。

      > 对于一些我们要求大家RTFM和RTFSC的内容，这本书里也提供了答案

   3. 本书讲义四，介绍了波形可能出现的问题，以及这些现象可能对应的原因，如果你在实验中遇到波形错误或是波形突然终止的问题，不妨尝试阅读一下这一部分。

3. **《计算机组成原理课程设计实验指导书 下》**。本书是构建流水线CPU设计部分的主体，介绍了TinyMIPS处理器的结构和功能，这部分内容是所有扩展实验内容的基础，也是验收考察的重点。

4. **《计算机组成原理课程设计指导书补充内容》**。本书介绍了在TinyMIPS的基础上扩展乘除法指令和异常处理的内容。

5. **Appendix压缩包。**其中，

 	* A02简略的介绍了实验所要求实现的MIPS指令集的规范。
 	* A04为MIPS官方的指令集规范。需要重点阅读。
 	* A07为实验环境的使用方法。
 	* 其他部分可以等有需要的时候在进行阅读。

6. **视频讲解。**来自16级的神级学长——MaxXing，TinyMIPS实验的设计者，19年计组课设助教。视频为当年Max学长为同学们答疑过程中留下的录播。

## 课程学习路线

<div id="timeline1" class = "block">

  <el-timeline >
    <el-timeline-item
      v-for="(activity, index) in activities"
      :key="index"
      :timestamp="activity.timestamp"
      :type="activity.type"     
      :color="activity.color"
      :id="activity.timestamp"
      placement="top">
      <el-card>

      <h4>{{activity.title}}</h4>

​      <br>
​      <b>主要任务:</b><div v-html="activity.content"></div><br>
​      <i>主要参考资料:{{activity.ref}}</i>
​      </el-card>
​    </el-timeline-item>
  </el-timeline>
</div>

<script type="text/javascript">
{
    let a = new Vue({
        el: '#timeline1',
        data:{
        reverse: true,
        activities: [{
          title:'TinyMIPS工程结构学习',
          content: '<ul><li>复习Verilog和计算机组成原理。</li><li>了解处理器各部分功能模块</li><li>完成第二章CG评测题</li><ul>',
          type:' success',
          color:'green',
          timestamp: '第一周',
          ref:'《计算机组成原理课程设计指导书 上》,《计算机组成原理课程设计指导书 下》第二章,视频资料:TinyMIPS的22条指令和五级流水线介绍'
        }, {
          title:'流水线前递与暂停机制学习',
          content: '<ul><li>了解流水线处理器基本原理</li><li>完成第三章CG评测题</li><li>完成<a href="https://www.ilab-x.com/details/page?id=6594&isView=true")>虚拟仿真实验平台</a>任务</li></ul>',
          color:'green',
          timestamp: '第二周',
          ref:'《计算机组成原理课程设计指导书 下》第三章,视频资料:读写相关的产生与解决'
        }, {
          title:'个人指令扩展',
          color:'green',
          content: '<ul><li>学习CDE仿真环境的使用</li><li>结合讲义和自己对TinyMIPS工程的理解。完成若干条指令的验收</li><ul>',
          timestamp: '第三周',
          ref:'《计算机组成原理课程设计指导书 上》讲义三'
        },{
          color:'green',
          title:'个人指令扩展',
          color:'green',
          content: '<ul><li>继续完成指令集扩展</li><li>欢度国庆三天长假</li><ul>',
          timestamp: '第四周',
          ref:'《计算机组成原理课程设计指导书 上》讲义三'
        },{
          title:'扩展任务——乘除法器设计/异常处理',
          content:'<ul><li>学习异常处理机制和乘除法器设计</li><li>按小组安排自行规划任务</li><ul>',
          timestamp:'第五周',
          color:'green',
          ref:'《计算机组成原理课程设计指导书补充内容》《计算机体系结构基础 第三版》'
        },{
          title:'扩展任务——AXI总线/设备输入输出',
          content:'<ul><li>学习AXI总线和设备输入输出</li><li>按小组安排自行规划任务</li><ul>',
          color:'green',  
          timestamp:'第六周',
          ref:'AXI总线协议（见参考资料）'
        },{
          title:'扩展任务——Cache设计与实现',
          color:'green',  
          content:'<ul><li>学习Cache的设计与实现</li><li>按小组安排自行规划任务</li><ul>',
          timestamp:'第七周',
          ref:'《超标量处理器设计》（见参考资料）'
        },{
          title:'扩展任务——答疑',
          color:'green',  
          content:'按小组安排自行规划任务',
          timestamp:'第八周',
          ref:'无'
        },{
          title:'扩展任务——答辩',
          color:'blue',  
          content:'分组完成答辩',
          timestamp:'第九周',
          ref:'无'
        }
        ]
    }
  });
    }

</script>