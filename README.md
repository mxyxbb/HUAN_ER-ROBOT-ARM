# 5自由度机械臂控制程序

video:

https://www.bilibili.com/video/BV1dU4y1W7H9

----------------------------------------------------------------------

使用STM32CubeMX生成代码，使用了HAL库

这是一个5自由度机械臂的控制程序，机械臂机械结构以3D打印制作。

移植了letter-shell，可以通过串口命令行来发送命令，调试机械臂。

可以实现舵机卸力，之后用手调整机械臂的位置，然后存储舵机角度，实现机械臂的动作组。

硬件：

- 飞特舵机SCS0009+SCS215(全部测试通过)
- 飞特UART控制板
- 3S锂电池
- BUCK降压模块
- STM32F407VET6(使用了I2C1和UART1、UART2)
- EEPROM芯片24C08(连接在I2C1)
- USB转串口模块

电脑通过USB转串口芯片连接到USART2（115200），并给STM32和舵机控制板供电

PD5	USART2_TX	
PD6	USART2_RX

3S锂电池通过降压模块降压到7.8V连接到飞特舵机控制板的供电端子

STM32的串口1波特率设置为1000000，连接到飞特舵机控制板的TX和RX

USART1_RX	PA10
USART1_TX	PA9

![image-20210215204900302](https://i.loli.net/2021/02/15/p5YTWluGe4qNBn9.png)

![image-20210224215957655](https://gitee.com/buddismblingblinghead/MxyPic/raw/master/img/20210224215957.png)

使用SecureCRT软件连接串口，复位单片机，可以得到如下界面：

```
 _         _   _                  _          _ _ 
| |    ___| |_| |_ ___ _ __   ___| |__   ___| | |
| |   / _ \ __| __/ _ \ '__| / __| '_ \ / _ \ | |
| |__|  __/ |_| ||  __/ |    \__ \ | | |  __/ | |
|_____\___|\__|\__\___|_|    |___/_| |_|\___|_|_|

Build:       Feb 15 2021 10:52:47
Version:     3.0.6
Copyright:   (c) 2020 Letter

mxy666:/$
```

键入以下命令并回车

```
cmds
```

可以查看到命令列表如下

```
Command List:
ami                   CMD   --------  ArmInit()
amf                   CMD   --------  ArmForceEnable(id,en)
fll                   CMD   --------  forceAll(en)
sap                   CMD   --------  savePos(id,time)
gop                   CMD   --------  goPos(id)
p2g                   CMD   --------  Pos2Group(gid,gpid,pid)
dog                   CMD   --------  DoGroup(id)
sa                    CMD   --------  SaveAll2ee()
ra                    CMD   --------  readAll2ram()
wp                    CMD   --------  WritePos(id,po,time,speed)
```

首先输入以下指令进行初始化

```
ami
```

然后输入以下指令关闭机械臂所有舵机的力矩输出

```
fll 0
```

之后，叫你的小伙伴将机械臂调整至合适角度，你在串口命令行输入以下指令，存储一个动作

```
sap 0 500
```

这样，如果你将舵机转至别处，然后调用以下指令恢复所有舵机力矩输出

```
fll 1
```

之后调用以下指令，执行动作0

```
gop 0
```

机械臂则会按照你前面的设置，在500ms内转至位置0处

若以同样的方法再存储一个动作2，如下

```
sap 1 500
```

则我们就可以将这两个动作连成一个动作组，指令如下

```
p2g 0 0 0
p2g 0 1 1
p2g 0 2 0
p2g 0 3 1
```

上面的第一个数字0表示我们设置的是动作组0，

第二个数字则是执行动作的顺序，

第三个数字是要执行的动作，

上面四句指令表示，动作组0使机械臂先运动到位置0，再运动到位置1，再回到0，再运动到1。

调用以下指令就可以看到这个现象

```
dog 0
```

这就是机械臂动作组的配置过程。

最后不要忘记调用以下指令保存数据，

```
sa
```

当你复位后，可调用以下指令恢复数据

```
ami
ra
```

![image-20210215200850469](https://i.bmp.ovh/imgs/2021/02/cdec0119f82a7e20.png)

