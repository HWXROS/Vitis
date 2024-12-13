###    

#   Vitis     教程   

  基于  https://github.com/Xilinx/Vitis-Tutorials/tree/2024.2/Getting_Started

## 一切的开始

-   安装Vitis（Linux）  

####   ![img](file:///tmp/lu9155rhm5.tmp/lu9155rhne_tmp_94acb2fa7d13b5b5.png)     准备工作 

​			  使用    apt    安装    gcc     和    g++  

- 去官网下载安装平台

![img](file:///tmp/lu9155rhm5.tmp/lu9155rhne_tmp_b9e3ffbeece5831c.png) 
 

 `chmod +x FPGAs_AdaptiveSoCs_Unified_2024.2_1113_1001_Lin64.bin`  

`sudo .FPGAs_AdaptiveSoCs_Unified_2024.2_1113_1001_Lin64.bin`

- 下载选项中只勾选 Vitis，它也会顺便安装Vivado
- 在结束后按照提示执行`sudo bash Vitis/版本/scripts/installLibs.sh`

​			  ( 如果卡住尝试更新 source.list为国内源，然后apt-get update)   

- 还需要先去官网下载镜像文件

https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/embedded-platforms.html

下载zynq的commom image（zcu102就是zynq702好像？）

- 解压，进入解压后的文件夹，查看readme

`Install SDK. This will install an architecture specific sysroot. This sysroot can be used with various platforms`

`$./sdk.sh -y -d <dir path to install> -p`

- 我这里新建了一个SDK文件夹，这些在第8 9 章节要用

![img](file:///tmp/lu9155rhm5.tmp/lu9155rhne_tmp_d2e977d9081bfff4.png) 

![img](file:///tmp/lu9155rhm5.tmp/lu9155rhne_tmp_5aa6916795ee7dd0.png) 

##  Vitis 工具的作用 

​	  #    当我们编写了一个用    vivado    写的    FPGA    硬件加速器，称为 PL 。二号  

​	  #    我们当然希望看看实际的加速情况啦，所以我们需要写软件部分，称为 APP 。  

​	  #    这时我们有了硬件和软件，怎么把软件中的数据变成字节输入到硬件中呢？  

​	  #    这里提出    （域）    Domain    的概念来，它可以是    操作系统、    XSA     文件、或者    XRT    这样的工具，  

​	  #    可以简单的理解为中间的沟通者。  

![img](file:///tmp/lu9155rhm5.tmp/lu9155rhne_tmp_100ce9c620a76d9.jpg) 


![img](file:///tmp/lu9155rhm5.tmp/lu9155rhne_tmp_697e8ca04300719b.png) 
 



![img](file:///tmp/lu9155rhm5.tmp/lu9155rhne_tmp_ff1dee2da7707fda.png) 
 

​	  #Vitis    编译器提供三种不同的构建目标：   

​	 #    软件仿真、硬件仿真。  

  	#    以及用于生成实际    FPGA    二进制文件的默认硬件目标  

## 配置    XRT Xilinx Runtime  

-   什么是    **XRT**    ？这是官方开源仓库的定义：  

  Xilinx runtime（    XRT    ）由用户空间和内核驱动组件共同实现。它支持基于    PCIe    的板卡，如    U30    、    U50    、    U200    、    U250    、    U280    、    VCK190    ，以及基于    MPSoC    的嵌入式平台。    XRT    提供了一个标准化的软件接口，    用于访问    Xilinx FPGA    。主要的用户    API    定义在  `  xrt.h <src/runtime_src/core/include/xrt.h>  `  _    头文件中。  

-   可以下载开源仓库然后按照仓库网页的    Build    教程。  

-   这里偷懒一下，去    AMD    官网按图选择，下方就出现了    XRT    下载链接了  

![img](file:///tmp/lu9155rhm5.tmp/lu9155rhne_tmp_f2adb300e9abd7a5.png) 
 

-   这里说    XRT    是驱动和    API    之间的桥梁  

![img](file:///tmp/lu9155rhm5.tmp/lu9155rhne_tmp_da67cc7983f412dd.png)  

-  下载并安装    deb    文件，能在    opt    里找到它这样就可以啦（大概？)  

-   这一步可能会报错，需要安装很多依赖，按提示    debug    就可以  

![img](file:///tmp/lu9155rhm5.tmp/lu9155rhne_tmp_df803f9500dc62e4.png)  
  `sudo dkpg -i xrt_202410.2.17.319_20.04-amd64-xrt.deb` 

## 下载 要使用的硬件平台文件    

​          就是开头提到的数据中心或嵌入式处理器，  

  #     这里我们暂时跳过，教程其实都提供了，但我们暂时使用    IDE    工具  

​	![img](file:///tmp/lu9155rhm5.tmp/lu9155rhne_tmp_94acb2fa7d13b5b5.png)

  5     配置     Vitis     运行环境  

  #    新建终端，    将这个终端配置成类似    Vivado    的    tcl    终端一样的用于    Vitis    的特殊终端  

​	  source <Vitis_install_path>/Vitis/<version>/settings64.sh  

  #    配置    XRT    环境变量什么的  	

​	  source /opt/xilinx/xrt/setup.sh  

  #    配置平台文件（跳过）  

  To specify the location of any Data Center or Embedded platforms you have installed, set the following environment variable:  

```
		export PLATFORM_REPO_PATHS=<path to platforms>
```

>   NOTE:   On some Ubuntu distributions, you must also export the `LIBRARY_PATH` to properly set up Vitis.（可以执行一下）

```
		export LIBRARY_PATH=/usr/lib/x86_64-linux-gnu
```

##  阅读教程提供的代码  

###   硬件代码  

- ​	  Vitis    支持    Verilog VHDL C/C++    ，这里教程使用的是    C++    ，编写的一个向量合加速器。使用    C++    ，硬件加速器的描述适合在不到    20    行的代码，可以很容易地和有效地实现在    FPGA    中  	  使用    Vitis    编译器。  

- ​	  代码可以在    example/src/vadd.cpp    查看     ,      这个简单的例子强调了    C++    内核的两个重要方面：  

```
    	1要求将C++内核声明为extern“C”，以避免名称损坏问题
    	2编译过程的结果由源代码中pragma的使用来控制。
```

###   软件代码  

- ​	  代码可以在    example/src/host.cpp    查看  

​	  #    这是通过    XRT    在将二进制可执行文件加载到设备上  

![img](file:///tmp/lu9155rhm5.tmp/lu9155rhne_tmp_d54bee611d482656.png) 

​	  #    这里获取硬件的    kernel    并创建与之交换数据的三个    buffer  

![img](file:///tmp/lu9155rhm5.tmp/lu9155rhne_tmp_5bf5c3a83b7e2f8a.png)  
	  #    将创建的    buffer    映射到主存地址上  

![img](file:///tmp/lu9155rhm5.tmp/lu9155rhne_tmp_6e8e7de6d12ec765.png) 

​	  #    通过映射的地址产生测试数据和验证数据，  

​	  #    通过    sync    函数将数据发送给设备，等待设备输出  

	  ######    	  #    验证输出，报错或将输出写入输出 buffer  

![img](file:///tmp/lu9155rhm5.tmp/lu9155rhne_tmp_cbce1b9335fe1ad0.png) 

   

##  Build and Run with the Vitis unified IDE  

  创建硬件部分  	

​	   #    还是之前的终端，创建一个工作空间在你喜欢的位置  

```
		``mkdir <tutorial_path>/workVADD
	#``启动
		vitis -w workVADD
```

`	#`  File > New Component >      HLS  

​	  #       组件名           vadd  ，配置文件  empty  默认即可     

![img](file:///tmp/lu9155rhm5.tmp/lu9155rhne_tmp_a7f68e6ea20f08cd.png) 

​	     #  源文件选  vadd.cpp  ，头函数是  vadd  也可以点击  browse  查看  ,  没有测试文件     

![img](file:///tmp/lu9155rhm5.tmp/lu9155rhne_tmp_dad2ba80ca3aa464.png) 


​	     #  组件选择  zu102     

​	     ![img](file:///tmp/lu9155rhm5.tmp/lu9155rhne_tmp_8da4ee2983785fc5.png)  
  #  在“设置”页面上，为时钟指定  8ns  ，为  clock_uncertainty  指定  12%  。     

​	     对于  flow_target  ，选择 vitis kernel  。     

​	     对于  package.output.format  ，指定  Generate a Vitis XO 。     

​	     ![img](file:///tmp/lu9155rhm5.tmp/lu9155rhne_tmp_fa1ac7cac75d7902.png)  
  #Finish     

![img](file:///tmp/lu9155rhm5.tmp/lu9155rhne_tmp_fcf4221998ec35f4.png) 
 

​     可以发现这似乎产生了一个类似 vivado flow navigation 窗口的界面，     

​     区别是针对的是 C  语言没有 elaborate多了 package     

​     **你可能认为这是 vivado  这样的软件做的事情，事实上  vitis unified IDE  的一个优点是能够自底向上工作，构建  HLS  或  AIE  组件，然后将它们集成到更高级别的系统项目中**     

​     打开左上角的文件目录在中心编辑器中打开 vitis-comp.json  文件可以看到  hls-log.cfg  文件，     

​     其中将放置构建指令以控制模拟和合成过程。     

​     创建软件组件     

`	  #  `  File > New Component >         Application     

​	     #  命名为 host ，  Domain 选择 XRT     

​	     #  选择  sysroot 就是一开始下的镜像文件里有的     

​	![img](file:///tmp/lu9155rhm5.tmp/lu9155rhne_tmp_75f5244180728ece.png)

​	     #  添加  host.cpp  文件     

​     创建完毕后看到左下角有俩种不同的  build,  对于  IDE 而言就用  X86  仿真就行。 然而失败了     

![img](file:///tmp/lu9155rhm5.tmp/lu9155rhne_tmp_cf811d19e3b061c9.png) 
 

![img](file:///tmp/lu9155rhm5.tmp/lu9155rhne_tmp_632c1c6843120f02.png) 
      去官网![img](file:///tmp/lu9155rhm5.tmp/lu9155rhne_tmp_b05257ad91a3d687.png) 查看问答，解决方案如下，我选择安装库，虽然还是报错，但  x86 编译通过了     

![img](file:///tmp/lu9155rhm5.tmp/lu9155rhne_tmp_a8a41950979ca342.png) 
 

​     8 Creating the System Project     

​	     在系统项目中，构建的不同组件被集成到单个系统中，  HLS  组件和应用程序组件通过以下步骤集成到系统项目中。     	

1. ​     `From the main menu select        File > New Component > System Project`    

1. ​     `输入       Component name   ：system_project (default)`

   Component location   as the workspace (default)

1. Select Part 页面选择 `xilinx_zcu102_base_202310_1` platform
2. 设置路径，按照最开始下载的镜像位置

![img](file:///tmp/lu9155rhm5.tmp/lu9155rhne_tmp_18661ee160448861.png) 
 

![img](file:///tmp/lu9155rhm5.tmp/lu9155rhne_tmp_eb30285c35ef0772.png)  
 
 

​	     创建系统项目后，需要对其进行配置。必须定义当前工作区中的哪些组件应添加到系统中。然后，我们可以定义  v++--link  命令使用的硬件链接选项，以及  v++--package  命令使用的打包选项来构建系统。     

​	     你可以花一点时间看看  setting  的内容     

​	     看到  Components  空缺了吧，这里我们点击添加，将我们的加速器加入到系统里面     

​	     容器名也改为  vadd  。（ HLS  组件被添加到一个二进制容器中，该容器包含将在  xclbin  文件中链接在一起的系统元素 。 Vitis  还将进行检查，以确保添加到  System  项目的各种元素兼容。它不会将指定用于不同  platform  的元素添加到单个系统中。 ）     

![img](file:///tmp/lu9155rhm5.tmp/lu9155rhne_tmp_8c587b7ebcd63dbf.png) 
 

​     下面再将  host Application 添加进系统吧，观察，你将注意到它不会有一个二进制容器     

![img](file:///tmp/lu9155rhm5.tmp/lu9155rhne_tmp_b8822c49af22b955.png) 
 

​     在  Hardware Linker Settings  部分中，还有一个命令用于在  Binary Containers  部分中添加预构建的  Binary  ，该命令允许您将先前构建的 kernel.xo  文件 （例如来自 RTL  内核的文件）或现有  AI Engine  图形应用程序的  libadf. a  添加到  System  项目中。     

![img](file:///tmp/lu9155rhm5.tmp/lu9155rhne_tmp_17fbd97c465a8732.png) 
 

​     查看  vadd  的  config  文件，修改  config  也可以通过左上角第二个按钮进入  source edit  模式     

![img](file:///tmp/lu9155rhm5.tmp/lu9155rhne_tmp_b8f401b3f705f971.png) 
 
 

![img](file:///tmp/lu9155rhm5.tmp/lu9155rhne_tmp_cb9bf6d3e0227e90.png) 
 



​     还可以查看  package setting          下面的  package.config     

###   以上仅供了解，不需要更改，下面开始 Building and Running the System Project     

​     在  Flow Navigator  中的选中系统项目，看到有三个不同的构建目标，如下所述：     

![img](file:///tmp/lu9155rhm5.tmp/lu9155rhne_tmp_708dd968d07e6c6d.png)  

​        软件仿真：  kernel  代码和  host application  被编译为在  X86  处理器而不是平台的嵌入式处理器上运行。  Vitis unifiedIDE  使用  PS on X86  仿真流。       软件仿真    target允许通过快速构建和运行循环进行快速迭代算法细化       。       此目标对于识别语法错误、执行与应用程序一起运行的内核代码的源代码级调试以及验证系统的行为非常有用       。     

​        硬件仿真：  kernel  代码被编译成硬件模型（  RTL  ），在  Vivado  逻辑仿真器中运行。  host application  在嵌入式处理器的仿真环境（  QEMU  ）版本上运行。       这个构建    -运行循环需要更长的时间，但是提供了一个详细的、周期准确的内核活动视图。此目标对于测试将进入器件的可编程逻辑（PL）区域的逻辑的功能以及获得初始性能估计非常有用       。     

​       硬件：  kernel  代码被编译成硬件模型（  RTL  ），然后在设备上实现，从而产生将在实际硬件上运行的二进制文件。在嵌入式处理器平台中，  host application  、  xclbin  和所需的支持文件被写入  SD  卡（例如），然后用于  boot  和配置系统。     

​	     下面，您将逐步了解 硬件仿真 构建，作为构建和运行系统的练习。     

​     点击  Hardware simulation 下面的  build all ，如果之前  sysroot  没问题应该会成功     

![img](file:///tmp/lu9155rhm5.tmp/lu9155rhne_tmp_b2eaed63c7f8562a.png) 
 

​     报错说  iostream  找不到  system::hw  就是你  sysroot  错了     

​     编译成功后我们选择  launch.json 添加硬件仿真配置     

​      ![img](file:///tmp/lu9155rhm5.tmp/lu9155rhne_tmp_1c550979c3b31e76.png)  
​      在FlowNavigator  中，单击  StartEmulator  为指定平台配置和运行  QEMU  环境     

![img](file:///tmp/lu9155rhm5.tmp/lu9155rhne_tmp_160203e5ab69b32b.png) 
 

  提示：等待    QEMU环境提示符出现，    然后再启动    Run命令    。在继续之前，您应该在“输出”窗口中看到类似于    zynqmp-common-20231：/mnt#的内容。  

​     在  Emulator  运行的情况下，单击  HARDWARE EMULATION  标题下的  Run  以启动  QEMU  环境中运行的系统项目。       您也可以选择    QEMU来使用QEMU环境调试System项目。  

​     遗憾的是       ，       这里我并没呢成功启动虚拟机？提示卡在 QEMU waiting PMU  的阶段了，王学长说可能是镜像的问题？说对初学者到这熟悉下流程差不多了……，我也没在网上找到解决方法，暂时搁置吧     

​     教程最后还有一节是教怎么看仿真输出的，自己看看吧     

![img](file:///tmp/lu9155rhm5.tmp/lu9155rhne_tmp_9046238f034d8065.png) 
 


 


 