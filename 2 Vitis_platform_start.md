# <font color="red">__Vitis 教程 __</font>：vivado2vitis～flow

---

---

黄维肖：2024-12-10

## **一** ： 教程配置 

- ### ***	Vitis Unified IDE 2024.2***, ***Vivado 2024.2***

- ### 	**Board: VCK190 VEK280**

- ### 	**提前去官网下载Versal对应的common镜像文件**

### 	     新建一个教程的工作空间文件夹，将镜像解压到里面

​		Install SDK tool by typing `sh xilinx-versal-common-v2024.2/sdk.sh -d xilinx-versal-common-v2024.2/ -y` in console. 

​		Option `-d` is to specify the directory where to install. 

​		Option `-y` means confirmation. So it gets installed in `xilinx-versal-common-v2024.2/` folder.

---

---

## **二：教程目的**	  

#### 	基于VCK 190或VEK 280评估板快速创建platform 并运行应用程序以验证此平台。	  

---

---

## 三：Create Vivado Design and Generate XSA

- 新建终端

Run `source <Vitis_Install_Directory>/settings64.sh` to setup Vivado running environment

其实这一步你可以直接写入/etc/bash.bashrc里面

- 打开vivado，tool->vivado store ->下载这个Example Designs，也可以点击refresh去获取最新的

![image-20241210145901110](/home/yellowsmile/.config/Typora/typora-user-images/image-20241210145901110.png)

- Create the Versal Extensible Embedded Platform Example project

![image-20241210150432015](/home/yellowsmile/.config/Typora/typora-user-images/image-20241210150432015.png)

- 填一个喜欢的名字，项目放在刚刚新建的文件夹里，重点来了，这里选vck190系列的套件

![image-20241210152839686](/home/yellowsmile/.config/Typora/typora-user-images/image-20241210152839686.png)

- 配置时钟设置。您可以在此视图中启用更多时钟、更新输出频率和定义默认时钟。在本例中，我们可以保留默认设置。
- Block Design Container（BDC）支持Vitis导出到Vivdao流程。用户可以相应地启用它。在本例中，将其禁用。
- 配置AXI主机和UART设置。您可以选择该平台应支持多少个中断和AXI主接口。63中断模式在级联模式下使用两个AXI_INTC。在这个例子中，我们可以保留默认设置。根据您的要求启用AI引擎（AI E）。在这个例子中，我们可以保留默认设置。

![image-20241210153040731](/home/yellowsmile/.config/Typora/typora-user-images/image-20241210153040731.png)

- finish 创建项目

在此阶段，Vivado模块自动化已经在图中添加了Control控制， Interface  接口和 Processing System处理系统（未来简称为CIPS）模块、AXI NOC模块、AI引擎和所有支持逻辑模块，并为VCK 190或VEK 280应用了所有板级调试。

![image-20241210154108568](/home/yellowsmile/.config/Typora/typora-user-images/image-20241210154108568.png)

- 通过 I**P integrator** 下面的**Generate Block Design**

![image-20241210154555474](/home/yellowsmile/.config/Typora/typora-user-images/image-20241210154555474.png)



- Select **Synthesis Options**   to **Global** to save generation time. **启动等待结束，这里会报错**

![image-20241210154922980](/home/yellowsmile/.config/Typora/typora-user-images/image-20241210154922980.png)

- 点击 **File -> Export -> Export Platform**.或 Flow Navigator的 **IP Integrator -> Export Platform**, 或 Platform Setup tab底部的**Export Platform** 按钮.

![image-20241210155657242](/home/yellowsmile/.config/Typora/typora-user-images/image-20241210155657242.png)

![image-20241210160350344](/home/yellowsmile/.config/Typora/typora-user-images/image-20241210160350344.png)

![image-20241210160901713](/home/yellowsmile/.config/Typora/typora-user-images/image-20241210160901713.png)

![image-20241210160954659](/home/yellowsmile/.config/Typora/typora-user-images/image-20241210160954659.png)

- **finish**之后xsa文件就在工作目录下面生成了

![image-20241210161130690](/home/yellowsmile/.config/Typora/typora-user-images/image-20241210161130690.png)



---

---



## 4 Create Vitis Plateform	

- 打开Vitis，设置工作空间在之前的文件夹
- **File > New Component > Platform** 
- 输入**Component name**. 比如 `vck190_custom`, 点击 **Next**.

![image-20241210163705419](/home/yellowsmile/.config/Typora/typora-user-images/image-20241210163705419.png)



![image-20241210164642121](/home/yellowsmile/.config/Typora/typora-user-images/image-20241210164642121.png)

![image-20241210164852685](/home/yellowsmile/.config/Typora/typora-user-images/image-20241210164852685.png)

![image-20241210165121876](/home/yellowsmile/.config/Typora/typora-user-images/image-20241210165121876.png)

- 点击 **linux On psv_cortexa72** domain.
- 输入 `xrt`作为display name。 这表明这是一个安装了XRT的Linux域，可以运行加速应用程序。
- **Bif file**: 点击browse右边的按钮自动生成。BIF文件在资源目录中生成。
- 再将Pre-built image Directory 设置成之前解压的镜像目录。Bootgen在此目录中查找BIF引用的BOOT组件以生成BOOT.BIN。

![image-20241210165512110](/home/yellowsmile/.config/Typora/typora-user-images/image-20241210165512110.png)



- 运行编译build

![image-20241210170011618](/home/yellowsmile/.config/Typora/typora-user-images/image-20241210170011618.png)

---

---

## 5 Validate the Vitis Plateform

- 这里教程让我们之间从vitis 的example去创建，但是这里我git报错remote-https无法识别，高半天终端里的git没问题但是vitis就是克隆不了仓库。我只好用之前的教程里的方法建了一个。



![vitis_examples.PNG](https://github.com/Xilinx/Vitis-Tutorials/blob/2024.2/Getting_Started/Vitis_Platform/images/vitis_examples.PNG?raw=true)

- 这是教程的项目情况

![missing image](https://github.com/Xilinx/Vitis-Tutorials/raw/2024.2/Getting_Started/Vitis_Platform/images/vitis_create_vadd.png)

- 这是我自己搭建的，这里和start 里的vitis教程无异只是把平台换成了vivado导出的这个vck190 costum 然后镜像不是zynq是versal了

![image-20241210192745537](/home/yellowsmile/.config/Typora/typora-user-images/image-20241210192745537.png)

- 对系统进行硬件build

![image-20241210193134733](/home/yellowsmile/.config/Typora/typora-user-images/image-20241210193134733.png)

结束后会形成镜像文件，我们可以通过sd卡将它导出，输入VCK190硬件里。后面的步骤就不执行了，看看吧

![image-20241210210512621](/home/yellowsmile/.config/Typora/typora-user-images/image-20241210210512621.png)

![image-20241210210631657](/home/yellowsmile/.config/Typora/typora-user-images/image-20241210210631657.png)

![image-20241210210705461](/home/yellowsmile/.config/Typora/typora-user-images/image-20241210210705461.png)
