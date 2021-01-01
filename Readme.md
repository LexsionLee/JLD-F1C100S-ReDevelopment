# JLD_F1C100S_ReDevelopment

------

## 前言：

-----

这是一款桌面共享充电器。最初的信息是：IC被打磨且方案未知，看起来就很廉价，明显不像Android平台。估计该机器工作时，用户扫码充电看广告。平时机器的手机充电线是没有输出的，扫码后程序会控制电源板部分，打开充电电路，给手机充电。本项目意在以学习为目的对此硬件进行二次开发，欢迎拥有此硬件的朋友参与到此项目中。  

*本项目的目标在于研究学习，相关文件皆搜集自互联网。若有文件侵犯相关公司版权或商业隐私，可提交Issue告知我删除，谢谢！*

## 分析概览：

-----

1. 电池虽为非知名厂商产品，但容量较大，有一定的利用价值。最简单的利用方法便是设计一个简单的开关电路激活输出，作为一个桌面备用电源。
2. 主控为全志F1C100S，荔枝派NANO使用的也是此IC。网络上有大量学习资料，主要[电路逻辑](./Analyze/NetAnalyzeV0.4.md)已经测出，可以尝试参考这些资料对本硬件进行二次开发。
3. 屏幕分辨率为480*960，其会在主板上电时被单片机初始化，所以针对屏幕驱动的操作，可直接延时等待单片机对其初始化，而不必须考虑写代码初始化问题。

| ------------- | 型号         |
| ------------- | ------------ |
| 主控IC        | 全志 F1C100S |
| GPRS模块      | Air202       |
| WIFI模块      | RTL8710AF    |
| 屏幕驱动      | ST7701S      |
| 音频功放IC    | 8002         |

## 各目录用途：

------

### Analyze：

存放分析相关文件，如逻辑分析数据，串口输出信息，电路逻辑等。

### Documents：

存放项目相关文档，如芯片或模块的数据手册、规格书，相关的开发文档等。

### Firmware：

存放固件文件，即各种产出的二进制固件文件。

### Others：

存放其他文件，即与本项目相关但不便分类的文件。

### SourceCode：

存放源代码文件，即本项目进行过程中使用的源代码。



## Todo：

-----

- [x] 分析主控IC型号
- [x] 找寻IC相关文档
- [x] 测量主要线路逻辑
- [ ] 如何搭建开发环境
- [ ] 适配一个Bootloader
- [ ] 适配一个Linux系统
- [ ] 裸机开发



## 参考学习网站：

-----

挖坑网：https://whycan.cn/index.html

荔枝派NANO：http://nano.lichee.pro/index.html

[Lexsion's Blog](http://lexsion.com)

# 分析记录-正文

-----


机器拆解可分为上下两个部分，以下分别分析：


## 机器下半部分的分析：

机器的下半部分主要由电池和电源控制板构成。看电池表面印字，得知此为不知名厂商生产的两枚10000mAh锂聚合物电池。两枚电池并联后连接到电源控制板上。经过分析得知以下内容：

1：此电路板没有针对主IC（SP4502）的关机功能设计。
2：如果意外关机，需要插入外接电源使其开机。
3：输出线上无输出，想作为普通电源使用，需改电路打开。

### 接插件脚位的分析：

（方向：此类接插件标准编号方向为：宽边向左，窄边向右，从下向上依次为Pin1~7）
1：GND
2：+5V
3：过限流电阻与LM393-Pin1（1OUT）相连（指示某种状态）
4：过限流电阻与LM393-Pin7（2OUT）相连（外接电源指示）
5：充电输出开关（高电平有效）
6：过晃动开关接地（晃动检测，防盗功能）
7：电池电压采样电路（输出约为电池电压40%）


## 机器上半部分的分析：

具有显示屏的主控部分拆开后可见：显示屏 * 1、主电路板 * 1、8Ω扬声器 * 1、2.4GPCB天线 * 1、GSMPCB天线 * 1.

### 显示屏方案分析：

显示屏上带有CTC5.7+7701S字样，屏幕排线40Pin。判断此屏幕可能是深超光电生产的尺寸为5.7英寸屏幕，其驱动IC可能是ST7701S。如果前面判断成立，根据查到资料分析，分辨率可能在480 * 854或480 * 960。根据外观结构判断，此屏幕不具备触摸交互功能。（后尝试驱动屏幕时，已经确定分辨率480*960）

### 主板方案分析：  

首先我们可以从主板上取下以下附件：  

1: 16G MicroSD卡 * 1 （该内存卡表面无印字且被刻意分区，Win下只显示3.x G，之前被误认为4G容量，但实际为16G）    
2: 中国移动物联卡 * 1 (测试可以连接基站，但无法打开数据，估计已经欠费)      

然后我们可以看到主板上主要的集成电路(IC)或模块如下：  

U1：观察电路可知，此为主IC，其表面印字被打磨，无法看出型号。此IC封装为QFN88，一枚24M晶振连接在51&52脚，RST键连接在70脚。目前有大量理由怀疑是全志的F1C100S（或F1C600，网上有资料表示此两款IC结构功能几乎或完全相同）  

U23：RTL8710AF，初步判断螃蟹家的2.4GHz WIFI 芯片。  

U14：Air202，一款GSM模块，网上可以查到资料。  

U9：25Q32CS16，一枚32Mbit（即4MB）SPIFlash*（注：后期开发中可改装为更大的16MB）*    

U10：8002E，音频功放。  

U13：IC表面无印字，经分析可能是定制单片机，用于初始化屏幕，亦存在加密IC功能之可能。    

U4、U5、U15：印字A17X，U4输出电压1.1V，这一般是ARM核心的供电电压；U5输出电压3.3V，这一般是各种嵌入式处理器，微控制器等各种器件的工作电压；U15输出受U1控制，激活后输出电压3.85V，电路中明显看出，其为U14供电，即GSM模块Air202。此IC类型为小电流DC/DC，具体型号为M3406。

U7：印字43TL，测量其输出电压为2.5V，这一般是DRAM的供电电压。具体型号有待分析。  

U8：印字B39F，分析电路此为液晶背光升压驱动，具体型号WD3139。

**主要IC的连接逻辑已经部分测出，具体在[Analyze目录下的NetAnalyze文件中](./Analyze/NetAnalyzeV0.4.md)提供。**

#### U1的详细分析过程：

通过观察首先确定此为主IC，表面打磨非常彻底，无法看出型号。首先数出IC引脚总数，确定其封装为QFN88。观察外围电路确定其晶振频率为24M，其连接在IC的第51和52脚。板载按键SW3实际功能为复位（RESET），通过测量得知其连接在IC的第70脚。  

拔下存储卡，发现机器不能正常显示开机动画，而是出现白屏倒计时，这情况表示可能有一部分系统数据存储在存储卡中。翻出读卡器，插到电脑上研究里面的文件，发现除了广告视频文件之外，还有一些二进制文件或配置文件。其中有一个扩展名为.axf的文件，这个文件很有意思。通过查询，在Stack Overflow中得到以下描述：The AXF file is an object file format generated by ARM's RealView compiler (also part of Keil's ARM-MDK) and contains both object code and debug information. （译：AXF文件是由ARM的RealView编译器（也是Keil的ARM-MDK的一部分）生成的目标文件格式，同时包含目标代码和调试信息）。二话不说，直接上16进制编辑器，发现其中多次出现“cpu=ARM926EJ-S”字段，判断此IC内核可能是ARM926EJ-S，*有用的知识增加了！*通过Google搜索ARM926EJ-S + QFN88，全志公司的官网就出来了。搜索ARM9 QFN88 24MHz就找到了AllWinnerF1C100S的数据手册。之前没有接触过全志的微处理器，了解了一下，发现F1C100S或F1C600封装符合，功能定位也符合这个货的功能。Google搜索到了数据手册和应用手册，下载手册后通过对比发现：晶振频率与脚位、复位脚位、各路电源脚位与取值，均完全符合，估计就是这货了。  

 之后有另一位网友表示，他从电路板上的测试点中探出了串口，获取了串口输出的启动信息。荔枝派技术大群有大牛看过该信息后，判断其极可能为F1C100S的某个系统所输出。

于是我们随便编译了一个全志F1C100S的Bootloader，烧录到了SPI Flash中，然后我们成功的从串口观察到了启动信息。如此便证实了此IC的型号确为：全志F1C100S。

#### U13研究：

  **U13电路连接情况：**

| 1 : 3V3                             | 8 : GND               |
| ----------------------------------- | --------------------- |
| 2 : To --> U1:49 / PE0              | 7 : To -- > CON1 - 37 |
| 3 : To --> CON1 - 40                | 6 : To --> CON1 - 39  |
| 4 : To --> U1:53 / PE5 (10K to 3V3) | 5 : To --> CON1 - 38  |

 

**逻辑分析仪各通道连接情况与功能猜测：**

| 逻辑分析仪通道 | U13脚位 | 功能猜测  |
| -------------- | ------- | --------- |
| 0              | Pin 4   | SCL       |
| 1              | Pin 2   | SDA       |
| -              | -       | -         |
| 2              | Pin 7   | CS/UnKnow |
| 3              | Pin 5   | MOSI      |
| 4              | Pin 6   | CLK       |
| 5              | Pin 3   | MISO /CS  |



我们通过分析发现，U13仅在上电时与屏幕通信。保持供电，仅对U1复位时，U13只会产生与U1的通信。猜测：U13可能为单片机，负责为显示屏上电初始化。考虑U13与U1通信后面部分会有变化，猜测U13可能还兼做加密使用。U13与屏幕通信方式可能为SPI、与U1通信方式可能为I2C。

使用DreamSourceLab的逻辑分析仪采集的信息存在于Analyze目录下，需要查看的可从中下载，上位机工具可去梦源实验室官网下载。

------

