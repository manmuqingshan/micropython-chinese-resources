# 单线数据通信

仅使用一个普通GPIO（支持电平中断功能）进行双向数据通信的 micropython 库，支持多个主机，适合只需要少量数据传输的应用。已在 STM32、ESP32、ESP8266 上测试。

```
         .---------.                .---------.
         |         |                |         |
         |         |                |         |
         |   GPIO8 o--------o-------o PA1     |
         |         |        |       |         |
         |         |        |       |         |
         |  ESP32  |        |       |  STM32  |
         |         |        |       |         |
         '---------'        |       '---------'
                            |
                            |       .---------.
                            |       |         |
                            |       |         |
                            '-------o IO5     |
                                    |         |
                                    | ESP8266 |
                                    |         |
                                    '---------'
```

**原理**

在 `SingleLineDataTrans` 中，一个数据由一个起始位（bitus）、8个数据位和一个停止位（charus）组成。其中 bitus 代表了基本数据位宽，数据位中时间宽度大于 bitus 的代表 1，否则是 0，这种机制简单而灵活，可以适应不同 mcu 的速度。bitus/charus受到mcu性能影响，性能越强，数值可以设置越小，传输速度也越快。如 esp32 可以设置为 300/600，而 esp8266 可以设置为 3500/6500。


```
bit      7  6   5  4 3 2 1  0
0x61     0  1   1  0 0 0 0  1
   ``|  |`|   |```| |`| |`|   |``````|
     |  | |   |   | | | | |   |      |
     |__| |___|   |_| |_| |___|      |_
    bitus                      charus
```

**使用方法**

```py
from SingleLineDataTrans import SingleLineDataTrans
from machine import Pin

sldt = SingleLineDataTrans(Pin(5))
sldt.write('123')
sldt.read()
```

**项目网址**
- https://github.com/shaoziyang/SingleLineDataTrans
