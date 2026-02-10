# Micropython Mock Machine

一个设计用于模拟 machine 模块进行单元测试的 MicroPython 库。这个库允许测试使用 machine 模块的 MicroPython 驱动程序和应用程序，而无需实际的硬件。它为常见的外设（如I2C、SPI、Pin、ADC等）提供了模拟实现。

## 安装方法

- 使用 mip  
  ```
  import mip
  mip.install("github:planetinnovation/micropython-mock-machine")
  ```
- 手动安装  
复制 `mock_machine.py` 文件到设备的 `/lib` 文件夹。

## 使用方法

- **基本用法**

  ```py
  # In your test file
  import mock_machine
  
  # Register mock_machine as the machine module
  mock_machine.register_as_machine()
  
  # Now any import of 'machine' will use the mock
  import machine
  
  # Create mock hardware
  i2c = machine.I2C(0)
  pin = machine.Pin(0, machine.Pin.OUT)
  
  # Use as normal
  pin.value(1)
  assert pin.value() == 1
  ```

- **模拟 I2C 设备**

  ```python
  import mock_machine
  from mock_machine import I2C, I2CDevice
  
  # Create I2C bus
  i2c = I2C(0)
  
  # Create a mock device at address 0x68
  device = I2CDevice(addr=0x68, i2c=i2c)
  
  # Set up register values
  device.register_values[0x00] = b'\x12\x34'  # WHO_AM_I register
  device.register_values[0x01] = b'\x00\x00'  # Control register
  
  # Your driver can now read from the device
  data = i2c.readfrom_mem(0x68, 0x00, 2)
  assert data == b'\x12\x34'
  ```

- **引脚中断**

  ```py
  import mock_machine
  import asyncio
  
  mock_machine.register_as_machine()
  import machine
  
  # Create a pin with interrupt
  pin = machine.Pin(0, machine.Pin.IN)
  
  # Set up interrupt handler
  def pin_handler(pin):
      print(f"Pin interrupt! Value: {pin.value()}")
  
  pin.irq(handler=pin_handler, trigger=machine.Pin.IRQ_RISING)
  
  # Simulate pin change
  pin.value(0)  # No interrupt
  pin.value(1)  # Triggers interrupt (rising edge)
  ```

- **SPI 通信**

  ```py
  import mock_machine
  mock_machine.register_as_machine()
  import machine
  
  # Create SPI bus
  spi = machine.SPI(0)
  
  # Set up mock responses
  spi.read_buf = b'\x01\x02\x03\x04'
  
  # Test your SPI driver
  data = spi.read(4)
  assert data == b'\x01\x02\x03\x04'
  
  # Check what was written
  spi.write(b'\xAA\xBB')
  assert spi.write_buf == b'\xAA\xBB'
  ```

https://github.com/planetinnovation/micropython-mock-machine
