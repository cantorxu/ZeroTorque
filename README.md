# 单关节阻抗控制算法

# 说明

本项目是基于基于[零差云控](https://www.zeroerr.cn./index.html?t=1715655854967)的机器人关节实现的单关节柔顺控制DEMO，可以包含零力模拟，阻抗控制等算法的实现。可以在省去六维力传感器的情况下，节约成本实现拖动示教等功能。可以实现对方向盘手感的初步模拟，本项目尚不完善，欢迎复刻并加以完善。

所使用的eRob110V6是一款国产高精度机器人关节

> **关节参数：**
>
> - 重复/绝对定位精度：±7/±15角秒或±10/±25角秒
> - 通信方式：EtherCAT/CANopen/Modbus
> - 输出端编码器分辨率：19/20Bit

# 文件构成

[ZeroTorque](https://github.com/cantorxu/ZeroTorque/tree/main/ZeroTorque)：主程序

TwinCAT Measurement Project1/2：参数测量辅助程序

# 硬件组成

- [eRob110H100I-BHM-18ET](https://www.zeroerr.cn./eRob/eRob110.html?t=1715650661007)（100速比，高精度编码器，EtherCAT版本）V6版本一台
- 连接法兰一件
- [汽车方向盘](https://item.taobao.com/item.htm?_u=d201kknersdc04&id=612764214603&spm=a1z09.2.0.0.55022e8dwVVWHW)一个
- [ePower电源](https://www.zeroerr.cn./Tools/ePower.html?t=1715650596404)一台
- [eRob专用线缆](https://www.zeroerr.cn./Tools/112.html?t=1715650615527)一根
- 网线一根
- M4沉头螺丝

# 开发环境

将关节固定，安装连接法兰，打上沉头螺丝，固定法兰与关节，使用4根方向盘配套螺丝将方向盘与法兰进行固定

使用ePower电源产生48V电压对关节进行供电，通过网线接双绞线使关节与主站进行通信

参考[《eRob CANopen and EtherCAT用户手册v1.8.pdf》](https://www.zeroerr.cn./d/file/download/eRob%20CANopen%20and%20EtherCAT%E7%94%A8%E6%88%B7%E6%89%8B%E5%86%8Cv1.9.pdf)的第六章“TwinCAT主站控制”介绍的基于 Beckhoff TwinCAT3 主站对 ZeroErr EtherCAT 从站设备进行 PDO 配置以及运动控制的方法和步骤进行基本的配置

## **PDO配置**

按照手册添加PDO参数，并将这些变量与程序(程序在后面给出)中定义的变量一一对应

| Name                          | Address | Align               |
| ----------------------------- | ------- | ------------------- |
| Transmit PDO                  | 0x1A00  | 链接变量            |
| Dual encoder difference value | 0x2241  | GVL.DualPosDiff     |
| Position actual value         | 0x6064  | GVL.actPosition     |
| Actual velocity               | 0x606C  | GVL.actVelo         |
| Receive PDO                   | 0x1600  | 链接变量            |
| Profile acceleration          | 0x6083  | GVL.udiProAcc       |
| Profile deceleration          | 0x6084  | GVL.udiProDec       |
| Modes of operation            | 0x6060  | GVL.siOperationMode |
| Target Velocity               | 0x60FF  | GVL.targetVelo      |

# 程序使用

导入程序`Main.TcPOU`，`KalmanFilter.TcPOU` 

在GVLs下新建一个`GVL.TcGVL`

## 导入配置参数

根据[数据标定程序](https://github.com/cantorxu/Data-calibration-for-Zerotorque)(另一个仓库中)输出的配置参数，对应的修改`Main.TcPOU`的`Offset`中的参数

## 运行程序

1. 将程序进行编译→Restart TwinCAT(Config Mode)→Activate Configuration
2. 在Receive PDO中手动将Control word从6使能(Force)为47，使能关节
3. 载入程序，将参数`iStep` 从0修改为1，开始程序
4. （可选）在示波器模块中观测关节运行状态
5. 调整K,B,M等参数来进行**柔顺控制**的不同情况的模拟
6. 将参数`iStep` 从15修改为1000，停止程序
7. 等待`iStep` 自动变为0，关节速度为0，将Control word从47使能(Force)为6，下使能关节

# 实验结果

## K = 0.3，B = 0.3，M = 0.5

此时，关节模拟处于一个标准的导纳控制环中，如下图所示，是关节被拖动时的位置曲线，关节受K作用弹回原始位置，存在阻尼会让波动趋于稳定

![Untitled](.\pic\Untitled.png)

## K = 0.3，B = 0，M = 0.5

将阻尼系数B调为0后，关节受到外力会不断震荡，不会停止，随后添加阻尼使之停下

![Untitled](.\pic\Untitled 1.png)

## K = 0，B = 0.3，M = 0.5

如下图所示，是关节在K调为0后的速度曲线，关节受到外力不会回归原位，而是速度趋于归0

![Untitled](.\pic\Untitled 2.png)

## K = 0，B = 0，M = 0.5

如下图所示，是关节在K调为0和B调为0后的速度曲线，遵守牛顿第一定律，在不受外力作用时，呈现一个匀速运行的状态，受外力时会改变自身速度

![Untitled](.\pic\Untitled 3.png)

# 后续开发

可以增加零速控制算法，因为高精度编码器的存在，对噪声较为敏感，如果不加死区限制，在零速时会不断受到扰动导致震荡的存在