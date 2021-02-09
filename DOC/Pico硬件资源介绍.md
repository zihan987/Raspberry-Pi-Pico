## Pico硬件资源介绍

Raspberry Pi Pico的开发环境我已经搭建好，接下来我们开始进入开发的正题，Pico和之前的Pi一样，也是具备一些GPIO供大家使用。

![GPIO.jpg](https://github.com/zihan987/Raspberry-Pi-Pico/blob/main/images/GPIO.jpg?raw=true)

- Raspberry Pi在英国设计的RP2040微控制器芯片
- 双核Arm Cortex M0 +处理器，时钟频率高达133 MHz
- 264KB的SRAM和2MB的板载闪存
- 形模块允许直接焊接到载板
- 具有设备和主机支持的USB 1.1
- 低功耗睡眠和休眠模式

- 通过USB使用大容量存储进行拖放编程
- 26个多功能GPIO引脚
- 2个SPI，2个I2C，2个UART，3个12位ADC，16个可控PWM通道
- 片上准确的时钟和计时器
- 温度感应器
- 片上加速浮点库
- 8个可编程I/O(PIO)状态机，用于自定义外围设备支持

Pico的内核是ARM Crotex-M0的内核，也就是说他的CPU是ARM系列的单片机，那么对应的外设资源就是IO、串口、SPI、I2C等等。那么我们学习的时候可以根据单片机的方法去学习。

除了核心板，官网还提供了一个插件的面包板，如下图所示。

<center>![面包板.jpg](https://github.com/zihan987/Raspberry-Pi-Pico/blob/main/images/%E9%9D%A2%E5%8C%85%E6%9D%BF.jpg?raw=true?)</center>

支持自己的DIY，开源了外设板卡：
![外设板卡.jpg](https://github.com/zihan987/Raspberry-Pi-Pico/blob/main/images/%E5%A4%96%E8%AE%BE%E6%9D%BF%E5%8D%A1.jpg?raw=true)

同样，核心板的原理图也开源了：

![原理图.png](https://github.com/zihan987/Raspberry-Pi-Pico/blob/main/images/%E5%8E%9F%E7%90%86%E5%9B%BE.png?raw=true)

