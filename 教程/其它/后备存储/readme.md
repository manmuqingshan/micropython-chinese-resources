# 后备存储 （machine.mem_backup）

后备存储（`machine.mem_backup`）是从 micropython 1.29 版本开始提供的新功能。使用 `machine.mem_backup` 可以访问系统中的一个特殊内存区域（后备存储的硬件实现方式以及后备存储区的数量和大小，在不同硬件上差异很大，请参考具体硬件说明），这个区域的数据在复位/掉电时可以保持不变，可以用来保存一些需要临时使用而又不需要持久化保存的数据，如传感器采集的数据、电池电压、重要寄存器状态等，这对于需要休眠的应用是非常重要的。相比将临时数据以文件方式写入 flash，使用后备存储速度快、功耗低，也不会对 flash 产生磨损。

其实在 esp32 上，以前的固件中就可以通过 `machine.RTC.memory` 使用这个功能，但是不够方便。`machine.RTC.memory` 是一个 bytes 对象，需要一次性读取/写入；而 `machine.mem_backup` 是内存视图（memoryview）对象，可以任意访问，使用简单，效率也更高。

`machine.mem_backup` 的使用非常简单，基本使用方式是：

- 通过 `machine.mem_backup(-1)` 获取后备存储区的数量和大小
- 通过 `machine.mem_backup(0)`、`machine.mem_backup(1)`（如果存在）这样方式获取后备存储区对象，以下标、切片等方式使用，读写数据

下面是官方文档给出的参考代码：

```python
import machine

mem = machine.mem_backup()
mem[0] = 0x12345678               # 写入元素 0
print(hex(mem[0]))                # 读取元素 0
print(len(mem))                   # 元素的数量
print(mem.itemsize)               # 每个元素大小
print(len(mem) * mem.itemsize)    # 总字节数

# 发现所有可用区域
for i, r in enumerate(machine.mem_backup(-1)):
    print(i, len(r), r.itemsize)
```


我们将上面代码修改一下，增加了数据修改和显示功能，查看在不同复位方式下后备存储数据的变化。它先获取系统可用的每个后备存储区，显示区域长度和元素大小，再修改部分存储区的元素（第二个元素加一，第三个元素加二，其它保持不变）并以hex方式显示出来。

```python
import machine

mem = machine.mem_backup()

for i, r in enumerate(machine.mem_backup(-1)):
    print(f'\nbackup memory {i}> length: {len(r)}, itemsize: {r.itemsize}')
    m = machine.mem_backup(i)
    n = min(10, len(r))
    print(m[0:3].hex(' '))
    m[1]+=1;m[2]+=2
```

然后在不同mcu上依次执行

- 软复位（machine.soft_reset()）
- 硬复位（machine.reset()）
- 深度休眠复位（machine.deepsleep(100)）
- 看门狗复位（设置machine.WDT(timeout=100)但不喂狗）
- RESET引脚复位

等操作，再次运行测试程序观察后备数据的变化，比较这些 mcu 的差异。


## STM32F405

第一次运行：

```
backup memory 0> length: 4096, itemsize: 1
8f fe b9 d2 4e 9c a0 24 de 6f

backup memory 1> length: 20, itemsize: 4
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
```

通过 `machine.soft_reset()` 复位：

```
backup memory 0> length: 4096, itemsize: 1
8f ff bb d2 4e 9c a0 24 de 6f

backup memory 1> length: 20, itemsize: 4
00 00 00 00 01 00 00 00 02 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
```

通过 `machine.reset()` 复位：

```
backup memory 0> length: 4096, itemsize: 1
8f 00 bd d2 4e 9c a0 24 de 6f

backup memory 1> length: 20, itemsize: 4
00 00 00 00 02 00 00 00 04 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
```

通过 deepsleep 复位

```
backup memory 0> length: 4096, itemsize: 1
8f 01 bf d2 4e 9c a0 24 de 6f

backup memory 1> length: 20, itemsize: 4
00 00 00 00 03 00 00 00 06 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
```

通过 WDT 复位

```
backup memory 0> length: 4096, itemsize: 1
8f 02 c1 d2 4e 9c a0 24 de 6f

backup memory 1> length: 20, itemsize: 4
00 00 00 00 04 00 00 00 08 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
```

通过 RESET 引脚复位

```
backup memory 0> length: 4096, itemsize: 1
8f 03 c3 d2 4e 9c a0 24 de 6f

backup memory 1> length: 20, itemsize: 4
00 00 00 00 05 00 00 00 0a 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
```

可以看到STM32F405上，后备存储区有两个，一个区域长度4096，每个元素大小是一个字节；另一个区域长度20，每个元素大小4字节。在不同复位情况下，后备区域数据保持不变或按照程序设定变化（以上数值仅作参考，不同环境下会有差异）。区域0的元素大小是1，所以数据按照字节变化；区域1元素大小是4，数据按照4字节变化。如果使用了电池给VBAT供电，还能实现断电情况下数据保存。


## ESP32S3

不同型号ESP32上后备储存的效果是一致的，下面以esp32s3为例。

第一次运行：

```
backup memory 0> length: 2048, itemsize: 1
e6 7e 25 07 a7 43 72 1c 8f b2
```

通过 `machine.soft_reset()` 复位：

```
backup memory 0> length: 2048, itemsize: 1
e6 7f 27 07 a7 43 72 1c 8f b2
```

通过 `machine.reset()` 复位：

```
backup memory 0> length: 2048, itemsize: 1
e6 80 29 07 a7 43 72 1c 8f b2
```

通过 deepsleep 复位

```
backup memory 0> length: 2048, itemsize: 1
e6 81 2b 07 a7 43 72 1c 8f b2
```

通过 WDT 复位

```
backup memory 0> length: 2048, itemsize: 1
e6 82 2d 07 a7 43 72 1c 8f b2
```

通过 EN 引脚复位

```
backup memory 0> length: 2048, itemsize: 1
e6 7e 21 07 a7 46 72 9c 0f 32
```

可以看到在 esp32s3 中，只有一个后备存储区，其大小是 2048，每个元素一个字节。数据在 deepsleep、machine.reset、WDT 等方式下复位保持不变，但是在 EN 引脚产生复位后数据丢失，这一点和文档描述一致。

## RP2040

第一次运行：

```
backup memory 0> length: 2048, itemsize: 1
e6 7f 27 07 a7 43 72 1c 8f b2

backup memory 1> length: 3, itemsize: 4
00 00 00 00 00 00 00 00 00 00 00 00
```

通过 `machine.soft_reset()` 复位：

```
backup memory 0> length: 4, itemsize: 4
00 00 00 00 01 00 00 00 02 00 00 00 00 00 00 00

backup memory 1> length: 3, itemsize: 4
00 00 00 00 01 00 00 00 02 00 00 00
```

通过 `machine.reset()` 复位：

```
backup memory 0> length: 4, itemsize: 4
00 00 00 00 02 00 00 00 04 00 00 00 00 00 00 00

backup memory 1> length: 3, itemsize: 4
00 00 00 00 02 00 00 00 04 00 00 00
```

通过 deepsleep 复位

```
backup memory 0> length: 4, itemsize: 4
00 00 00 00 03 00 00 00 06 00 00 00 00 00 00 00

backup memory 1> length: 3, itemsize: 4
00 00 00 00 03 00 00 00 06 00 00 00
```

通过 WDT 复位

```
backup memory 0> length: 4, itemsize: 4
00 00 00 00 04 00 00 00 08 00 00 00 00 00 00 00

backup memory 1> length: 3, itemsize: 4
00 00 00 00 04 00 00 00 08 00 00 00
```

通过 RUN 引脚复位

```
backup memory 0> length: 4, itemsize: 4
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00

backup memory 1> length: 3, itemsize: 4
00 00 00 00 00 00 00 00 00 00 00 00
```

## RP2350

第一次运行：

```
backup memory 0> length: 4, itemsize: 4
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00

backup memory 1> length: 3, itemsize: 4
00 00 00 00 00 00 00 00 00 00 00 00

backup memory 2> length: 8, itemsize: 4
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
```

通过 `machine.soft_reset()` 复位：

```
backup memory 0> length: 4, itemsize: 4
00 00 00 00 01 00 00 00 02 00 00 00 00 00 00 00

backup memory 1> length: 3, itemsize: 4
00 00 00 00 01 00 00 00 02 00 00 00

backup memory 2> length: 8, itemsize: 4
00 00 00 00 01 00 00 00 02 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
```

通过 `machine.reset()` 复位：

```
backup memory 0> length: 4, itemsize: 4
00 00 00 00 02 00 00 00 04 00 00 00 00 00 00 00

backup memory 1> length: 3, itemsize: 4
00 00 00 00 02 00 00 00 04 00 00 00

backup memory 2> length: 8, itemsize: 4
00 00 00 00 02 00 00 00 04 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
```

通过 deepsleep 复位

```
backup memory 0> length: 4, itemsize: 4
00 00 00 00 03 00 00 00 06 00 00 00 00 00 00 00

backup memory 1> length: 3, itemsize: 4
00 00 00 00 03 00 00 00 06 00 00 00

backup memory 2> length: 8, itemsize: 4
00 00 00 00 03 00 00 00 06 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
```

通过 WDT 复位

```
backup memory 0> length: 4, itemsize: 4
00 00 00 00 04 00 00 00 08 00 00 00 00 00 00 00

backup memory 1> length: 3, itemsize: 4
00 00 00 00 04 00 00 00 08 00 00 00

backup memory 2> length: 8, itemsize: 4
00 00 00 00 04 00 00 00 08 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
```

通过 RUN 引脚复位

```
backup memory 0> length: 4, itemsize: 4
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00

backup memory 1> length: 3, itemsize: 4
00 00 00 00 00 00 00 00 00 00 00 00

backup memory 2> length: 8, itemsize: 4
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
```

RP2040 和 RP2350 上，后备存储区都比较小，只能保存少量数据。它们和ESP32类似，在 RESET 引脚方式下复位会丢失数据，其它方式下复位数据可以保持。

## SAM D51

第一次运行：

```
backup memory 0> length: 8192, itemsize: 1
b3 77 b3 7e 74 c8 d4 5a e5 3b
```

通过 `machine.soft_reset()` 复位：

```
backup memory 0> length: 8192, itemsize: 1
b3 78 b5 7e 74 c8 d4 5a e5 3b
```

通过 `machine.reset()` 复位：

```
backup memory 0> length: 8192, itemsize: 1
b3 79 b7 7e 74 c8 d4 5a e5 3b
```

通过 deepsleep 复位

```
backup memory 0> length: 8192, itemsize: 1
b3 7a b9 7e 74 c8 d4 5a e5 3b
```

通过 WDT 复位

```
backup memory 0> length: 8192, itemsize: 1
b3 7b bb 7e 74 c8 d4 5a e5 3b
```

通过 RESET 引脚复位

```
backup memory 0> length: 8192, itemsize: 1
b3 7c bd 7e 74 c8 d4 5a e5 3b
```

SAM D51 的后备存储表现形式和STM32类似，也支持VBAT（PB03）。毕竟是同一时期的芯片，都是 Cortex-M4 内核，设计理念和应用场景类似。
