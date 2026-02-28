# TM1637 数码管驱动

TM1637 是一种带键盘扫描接口的LED（发光二极管显示器）驱动控制专用电路，内部集成有MCU 数字接口、数据锁存器、LED 高压驱动、键盘扫描等电路。TM1637 的功能和用法与 TM1650 类似，只是接口方式不同，TM1650是I2C接口，而TM1637是两线方式，可以使用任意两个IO进行驱动。

使用方法(需要先将 [TM1637 驱动](https://gitee.com/microbit/mpy-lib/tree/master/LED/TM1637) 复制到开发板中)：

```py
from machine import Pin
import time
import TM1637

dio = Pin(5, Pin.OUT)
clk = Pin(4, Pin.OUT)
tm=TM1637.TM1637(dio=dio,clk=clk)

n = 0
while 1:
    tm.shownum(n)
    n += 1
    time.sleep_ms(1000)
```