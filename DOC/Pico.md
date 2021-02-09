## Pico开发环境安装

Pico可以在电脑和树莓派上做开发，阿chai在这里主要介绍在电脑上的开发环境，分为C版本与MicroPython版。阿chai建议大家使用MicroPython版本。

### MicroPython环境搭建

首先需要刷固件。

可将USB将Pico连接到计算机，然后将文件拖放到其中来对Pico进行编程，官方整理了一个可[下载的UF2](https://micropython.org/download/rp2-pico/)文件。

1. 下载MicroPython UF2文件。
2. 按住BOOTSEL按钮，然后将Pico插入Raspberry Pi或其他计算机的USB端口。连接Pico后，释放BOOTSEL按钮。
3. 它将安装为称为RPI-RP2的大容量存储设备。
4. 将MicroPython UF2文件拖放到RPI-RP2卷上。您的Pico将重新启动。您现在正在运行MicroPython。

![pico](https://github.com/zihan987/Raspberry-Pi-Pico/blob/main/images/pico.gif)

上图中的IDE是官方推荐使用Thonny，但是阿chai还是喜欢VSCode，那么下面介绍一下VSCode中搭建环境。

调试工具与使用：

```bash
$ brew install minicom
$ minicom -b 115200 -o -D /dev/cu.usbmodem0000000000001
```

安装MicroPython的插件：

![image-20210208000330608](https://mmbiz.qpic.cn/mmbiz_jpg/gIb5Pv6jCfdBAkoqibdyrFBMPib6ndEckfSH97PxicO6bDsIDicCzxnQoRZXnH9JTY4zSYGruHzpeTefK0663VlFkg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

因为upload项目是需要使用串口的，因此我们进行串口配置：

```bash
# 查看groups
$ groups

# 添加
$ sudo usermod --append --groups dialout tux
```

使用如下代码进行测试可以看到LED的闪烁。

```python
from machine import Pin, Timer

led = Pin(25, Pin.OUT)
tim = Timer()
def tick(timer):
    global led
    led.toggle()

tim.init(freq=2.5, mode=Timer.PERIODIC, callback=tick)
```

### C/C++ SDK

C/C++版本的程序比较偏向底层，开发需具备嵌入式相关知识。

首先我们下载代码并安装对应的SDK。

```bash
$ git clone -b master https://github.com/raspberrypi/pico-sdk.git
$ cd pico-sdk
$ git submodule update --init
$ cd ..
$ git clone -b master https://github.com/raspberrypi/pico-examples.git
```

添加环境变量：

```bash
export PICO_SDK_PATH=自己设置的pico-sdk路径 
```

调试工具与使用：

```bash
$ brew install minicom
$ minicom -b 115200 -o -D /dev/cu.usbmodem0000000000001
```

在VSCode中安装cmake tools：

![image-20210207233004408](/Users/zty/Library/Application Support/typora-user-images/image-20210207233004408.png)

settings中找到CMake Tools configure ，点击 CMake Tools configure 选项，往下拉， 找到 Configure Environment 、 点击添加项， 在项中写 PICO_MDK_PATH 在值中填入 SDK 的安装路径。

![image-20210207233249534](/Users/zty/Library/Application Support/typora-user-images/image-20210207233249534.png)

Cmake: Generator 中填写 NMake Makefiles。

![image-20210207233705969](/Users/zty/Library/Application Support/typora-user-images/image-20210207233705969.png)

安装交叉编译工具：

```bash
$ brew install cmake
$ brew tap ArmMbed/homebrew-formulae
$ brew install arm-none-eabi-gcc
```

在VSCode顶部的导航栏中选择arm的交叉编译工具即可使用。

可使用如下的程序进行测试：

```c
#include "pico/stdlib.h"

int main() {
    const uint LED_PIN = 25;
    gpio_init(LED_PIN);
    gpio_set_dir(LED_PIN, GPIO_OUT);
    while (true) {
        gpio_put(LED_PIN, 1);
        sleep_ms(250);
        gpio_put(LED_PIN, 0);
        sleep_ms(250);
    }
}
```

CMakeLists.txt如下：

```cmake
add_executable(blink
        blink.c
        )
target_link_libraries(blink pico_stdlib)
pico_add_extra_outputs(blink)
example_auto_set_url(blink)
```

在 bulid 文件夹中找到对应的编译生成文件 bin elf u2f 。将 U2f 拖入到 Pico 识别出的磁盘中可以看到LED的闪烁。
