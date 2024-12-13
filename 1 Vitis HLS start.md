# Vitis HLS Analysis and Optimization

黄维肖 2024年12月11日

---

---

# **教程目标**

- ### **Vitis高级综合[（HLS）]()可让您将C/C++代码编译为RTL代码，以便在AMD设备的可编程逻辑（PL）区域中实施。**

- ### **它是执行C/C++代码的高级合成并将其导出为Vivado IP（.zip）以用于Vivado设计套件和嵌入式软件开发流程，或导出为Vitis Kernel [.xo]()以用于葡萄属加速流程和异构系统设计的工具。**

- ### **尽管Vitis Kernel和Vivadp IP流相似，但仍存在一些根本差异，如  [Introduction to Vitis HLS Components](https://docs.amd.com/r/en-US/ug1399-vitis-hls/Introduction-to-Vitis-HLS-Components)中所述。本教程演示了用于自底向上开发HLS设计的Vitis Unified IDE。**

---

---

# 软硬件要求

- 和之前的情况差不多，这里自己看看把

![image-20241211144108421](/home/yellowsmile/.config/Typora/typora-user-images/image-20241211144108421.png)

---

---

# 获得教程有关文件

- **git clone https://github.com/Xilinx/Vitis-Tutorials.**
- **导航到Getting_Started/Vitis_HLS目录，然后访问reference-files目录。**
- **vitis -w workspace**
- 去官网https://amd.entitlenow.com/AcrossUser/main.gsp?获取vitis hls license 

​	这是免费的，终端输入ifconfig 查看ether这个参数（网卡地址？），获取后在vivado/help/manage license里面激活即可，以后要用

![image-20241211171937937](/home/yellowsmile/.config/Typora/typora-user-images/image-20241211171937937.png)

---

---



# 创建 Vitis HLS 项目 

### Create the project to deine the HLS component.

- 创建工作区：
- `mkdir     <tutorial_path>/Getting_Started/Vitis_HLS/reference_files/workDCT`
- 启动IDE：`vitis  -w  workDCT`
- **File > New Component > HLS**
- **Component name <------dct**
- **Add Files**   `<tutorial_path>/Getting_Started/Vitis_HLS/reference_files/src/dct.cpp` 
- **Top function** `dct(short short)`
- 在**Test Bench Files**下，选择src 目录下的以下三个文件：
      dct_test. cpp是一个用于多次迭代内核的设计的测试平台。
      in.dat提供要由内核处理的输入值。
      out.golden.dat提供已知的输出结果以用于比较DCT函数的输出。

- `编写一个好的测试平台可以大大提高您的生产力，因为C函数的执行速度比RTL模拟快几个数量级。在综合之前使用C语言开发和验证算法比开发和调试RTL代码要快得多。`
- **part** 选择vitis默认分配的
- **clock**为周期指定8ns，为**clock_uncertainty**指定12%
- 选择**Vitis kernel flow**目标，并为package.output.format指定**Generate a vitis xo**

---



### Running Simulation

- 首先看一看设置，浏览hls_config.cfg文件或者点击C_SIMULATION右边的螺丝按钮自行了解

![image-20241211155359209](/home/yellowsmile/.config/Typora/typora-user-images/image-20241211155359209.png)

- 直接点击**C_SIMULATION的RUN**或者使用**命令行**

<u>`vitis-run --mode hls --csim --work_dir dct --config <tutorial_path>/Getting_Started/Vitis_HLS/reference-files/workDCT/dct/hls_config.cfg`</u>

![image-20241211160950954](/home/yellowsmile/.config/Typora/typora-user-images/image-20241211160950954.png)

###  Running Synthesis

同理使用命令行或者图形化界面都可以

 <u>`v++ -c --mode hls --config /home/yellowsmile/Downloads/Vitis-Tutorials-2024.2/Getting_Started/Vitis_HLS/reference-files/workDCT/dct/hls_config.cfg --work_dir dct`</u>

`C synthesis 的默认输出格式是 RTL. 这是为了让您在导出Vivado IP或Vitis Kernel以进一步用于嵌入式系统设计或数据中心加速之前评估高级综合的结果。`
`在设计综合时可以从Output 窗口的 看到dct::synthesis transcript. 它解释了 what steps the tool takes during synthesis. The following list describes some of the steps listed:`

![image-20241211161935714](/home/yellowsmile/.config/Typora/typora-user-images/image-20241211161935714.png)



`Vitis HLS工具还自动内联小函数，将逻辑分解到更高级别的调用函数中，并通过有限的迭代来流水线小循环。这些功能可通过hls_config. cfg文件中的用户指令或dct. cpp源代码中的杂注进行配置。综合完成后，您将看到Run（运行）命令标记有一个绿色圆圈和一个复选标记，表示它已成功运行。然而，它也将有一个黄色的警告三角形，以表明输出格式是RTL，没有IP或XO从设计中生成。在HLS组件可以在下游工具（如Vivado Design Suite）或系统项目中使用之前，需要生成此输出。`

### Analyze the Result

![image-20241211163137876](/home/yellowsmile/.config/Typora/typora-user-images/image-20241211163137876.png)

- 这里可以点击Timing Estimate 的下方图标将ns改为MHz

----

![image-20241211163628430](/home/yellowsmile/.config/Typora/typora-user-images/image-20241211163628430.png)

- 请注意，dct.cpp源中的各种子函数未在合成结果中报告。这是因为该工具已自动[内联]()了这些函数。您可以通过为函数添加INLINE OFF杂注或指令，或通过向设计中添加DATAFLOW优化（您将在本教程的后面部分执行此操作）来禁用特定函数的内联。

- HLS工具还自动对迭代次数少于指定次数的循环进行流水线处理。迭代次数少于64次的流水线循环是默认设置。当流水线时，该工具试图实现1的II。II是处理循环的下一次迭代之前的时钟周期数。当使用II=1的流水线进行循环时，您希望下一次迭代在下一个时钟周期开始。

- **确保已经激活vitis hls证书**

  **Function Call**

  ![image-20241211172256058](/home/yellowsmile/.config/Typora/typora-user-images/image-20241211172256058.png)**Schedule Viewer**计划查看器的左侧按合成函数中的时间顺序列出了每个操作。它以时间轴的形式水平显示设计控制步骤，从步骤0开始，一直到完成。您可以从列表中选择操作以查看它们之间的连接。

![image-20241211172739556](/home/yellowsmile/.config/Typora/typora-user-images/image-20241211172739556.png)



---

---



# Using Optimization Techniques

- 打开hls_log. cfg文件。

### compile.pipeline_loops

- 选择配置文件编辑器左侧的编译类别。

![image-20241211173112012](/home/yellowsmile/.config/Typora/typora-user-images/image-20241211173112012.png)

- 找到**compile.pipeline_loops**选项并将其设置为5。这表明该工具应自动展开具有六次或更多次迭代的循环。默认设置为64。
- 这里点击**Source Editor**可以看到最后一行有了变化

![image-20241211173348138](/home/yellowsmile/.config/Typora/typora-user-images/image-20241211173348138.png)

- 在Flow Navigator（流导航器）中，单击C Synthesis（C合成）下的Run（运行）命令，以使用新指令进行编译。
- 检查更新后的报告以查看是否有任何性能改进。

### Pipeline Directive（指令）

慢慢看吧，不是很理解 ，ll是交互时间？

Another possible optimization is to tell the tool that a function or loop should occur before processing another sample. The `Pipeline` pragma or directive defines an acceptable level of performance, and can eliminate II violations from the reports because the latency would then match your specification. The overall latency of an application could  indicate that perhaps**（interval latency） `II=4`** is acceptable for some loops.

This configuration might be an acceptable response to II  violations when the loops are not in the critical path of the design, or they represent a small problem relative to some larger problems that  must be resolved. In other words, not all violations need to be  resolved, and in some cases, not all violations can be resolved. They  are simply artifacts of performance.

-  删掉之前的更改，同样在cfg里找到**Design Directives > Pipeline**

![image-20241211174624923](/home/yellowsmile/.config/Typora/typora-user-images/image-20241211174624923.png)

![image-20241211174903257](/home/yellowsmile/.config/Typora/typora-user-images/image-20241211174903257.png)

- 更改ALIAS选择PIPELINE指令并指定**II**（interval latency）为4。可以选择Source File将其作为杂注直接添加到代码中，也可以选择Config File将其作为指令添加到HLS配置文件中。单击确定将杂注或指令添加到设计中。[就是蓝色这一行]()

- ![image-20241211175017913](/home/yellowsmile/.config/Typora/typora-user-images/image-20241211175017913.png)

- 在Flow Navigator（流导航器）中，单击C Synthesis（C合成）下的Run（运行）命令，以使用新指令进行编译。

- 检查更新后的报告以查看是否有任何性能改进。

  

### Assign Dual-Port RAMs with BIND_STORAGE

**分配双端口**

In some designs, a Guidance message `Unable to schedule load operation...` indicates a load/load (or read/read conflict) issue with memory  transactions. In these cases rather than accepting the latency, you  could try to optimize the implementation to achieve the best performance (II=1).

The specific problem of reading or writing to memory can  possibly be addressed by increasing the available memory ports to read  from, or to write to. One approach is to use the BIND_STORAGE pragma or  directive to specify the type of device resource to use in implementing  the storage. BIND_STORAGE defines a specific device resource for use in  implementing a storage structure associated with a variable in the RTL.  For more information, refer to [BIND_STORAGE pragma](https://docs.amd.com/r/en-US/ug1399-vitis-hls/pragma-HLS-bind_storage) or [syn.directive.bind_storage](https://docs.amd.com/r/en-US/ug1399-vitis-hls/syn.directive.bind_storage) .

- 一样回退更改
- 查看综合报告的Storage report部分，您可以看到该工具已使用ram_s2p实现了buf_2d_in变量。这允许在一个端口上进行阅读操作，同时在另一个端口上进行写操作。**但RAM 2P允许在两个端口上同时进行阅读，或者在一个端口上进行阅读，在另一个端口上进行写。这可能会带来一些性能改进。**

1. **Directive Editor-->Bind Storage** . 在HLS Directive view 选择dct function, 选择buf_2d_in variable, and select **Add Directive**.

   ![image-20241211180912704](/home/yellowsmile/.config/Typora/typora-user-images/image-20241211180912704.png)

2.  **BIND_STORAGE** 更改 **type** `ram_2p` **impl** `bram`

![image-20241211181222305](/home/yellowsmile/.config/Typora/typora-user-images/image-20241211181222305.png)

- 重新综合测试结果



### Using Array_Partition

Another approach to solve memory port conflicts is to use the `Array_Partition` directive to [reconfigure the structure of an array.]() `Array_Partition` lets you partition an array into smaller arrays or into individual  registers instead of one large array. This effectively increases the  amount of read and write ports for the storage and potentially improves  the throughput of the design. However, `Array_Partition` also requires more memory instances or registers, and so increases area and  resource consumption. For more information, refer to [Array ](https://docs.amd.com/r/en-US/ug1399-vitis-hls/Array-Partitioning)

- 回退更改

- Config Editor select **Add Item** for **Array Partition** to open the Directive Editor. In the HLS Directive view navigate to the `dct` function,

![image-20241211181918750](/home/yellowsmile/.config/Typora/typora-user-images/image-20241211181918750.png)

- select the `buf_2d_out` variable, and select **Add Directive**.

In the Add Directive dialog box select the **ARRAY_PARTITION** pragma, specify **type** of `cyclic` and **factor** of `8`, and click **OK** to add the directive or pragma to your design.

![image-20241211182020148](/home/yellowsmile/.config/Typora/typora-user-images/image-20241211182020148.png)

- Repeat this process for the `col_inbuf` variable of the `dct_2d` function,  with the same settings for the **ARRAY_PARTITION** pragma.

选择因子为8的循环划分的原因与所涉及的代码结构有关。该循环正在处理一个8x8矩阵，这需要通过外部循环进行八次传递，并通过内部循环进行八次传递。通过选择因子为8的循环数组分区，您将创建八个单独的数组，每个数组在每次迭代中读取一次。这消除了在循环的流水线期间访问存储器的任何争用。

- 运行合成并重新检查“合成摘要”报告以查看此最新更改的结果。