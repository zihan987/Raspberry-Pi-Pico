## I/O使用

操作I/O是玩硬件的基础，我们可以用来控制开关量的设备，例如点灯、蜂鸣器、按键等等。详细的硬件的设计这里并不做解释，课参考单片机的学习资料，主要介绍程序撰写。

我们以点灯为例子看一下GPIO的使用，在核心板上就一偶一个LED：

![LED1.jpg](https://github.com/zihan987/Raspberry-Pi-Pico/blob/main/images/LED1.jpg?raw=true)

在开始安装环境的时候阿chai提供的程序就是点灯的程序，接下来我们看看具体是如何操作的。

![LED2.jpg](https://github.com/zihan987/Raspberry-Pi-Pico/blob/main/images/LED2.jpg?raw=true)

### MicroPython版本

MicroPython中提供了点灯的接口，我们看一下程序：

```python
# 调用pico的GPIO模块
from machine import Pin

# 设置GPIO的模式，案例中为输出模式
led = Pin(25, Pin.OUT)

# 输出高低电平
led.value(1)
led.value(0)
```

这个时候我们可以看到小灯是亮一下灭了。

### C/C++版本

以下为运行程序：

```c
#include "pico/stdlib.h"  //头文件

int main() {
    const uint LED_PIN = 25;  //设置IO口
    gpio_init(LED_PIN);  //初始化IO
    gpio_set_dir(LED_PIN, GPIO_OUT);  //设置IO口的模式
    while (true) {
        gpio_put(LED_PIN, 1);  //输出高电平
        sleep_ms(250);				 //延时250ms
        gpio_put(LED_PIN, 0);  //输出低电平
        sleep_ms(250);				 //延时250ms
    }
}
```

CMakeLists.txt:

```cmake
add_executable(blink
        blink.c
        )

# Pull in our pico_stdlib which pulls in commonly used features
target_link_libraries(blink pico_stdlib)

# create map/bin/hex file etc.
pico_add_extra_outputs(blink)

# add url via pico_set_program_url
example_auto_set_url(blink)
```

