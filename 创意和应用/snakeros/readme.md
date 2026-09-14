# MicroPython的纯Python ROS 2客户端 - Snakeros

SnakeROS允许MicroPython电路板（Pico 2 W、Pico W、ESP32）发布和订阅真正的ROS 2主题。没有C工具链、没有冷门、没有自定义固件，无需刷新即可添加消息类型。

```python
from snakeros import Node
from snakeros.msg.std_msgs import String

node = Node('pico_node', agent='192.168.1.10')
pub = node.create_publisher(String, 'chatter')

while True:
    pub.publish(String(data='hello from a Pico'))
    node.spin_once()
```

在主机上就是一个ROS 2主题:
```python
$ ros2 topic echo /chatter
data: hello from a Pico
```

## 运作原理

SnakeROS 用 Python 直接向 micro-ROS 代理发送 DDS-XRCE。Agent不关心线路另一端的内容，因此无需运行桥，不需要写入中继节点，也没有主机端软件。

```
┌──────────────────┐         ┌───────────────────┐         ┌─────────────┐
│  MicroPython     │  XRCE   │  micro-ROS Agent  │  DDS    │  ROS 2      │
│  board           │ ──────► │  (stock, unmod'd) │ ──────► │  graph      │
│  + snakeros      │  UDP /  │                   │         │             │
└──────────────────┘  serial └───────────────────┘         └─────────────┘
```

XRCE根据运行时发送的XML字符串创建实体，这是使纯Python客户端实用的诀窍：任何 ROS 2 消息类型都可以通过构建字符串来实现，因此添加类型绝不意味着需要重建固件。

## 相关链接

- [github仓库](https://github.com/kevinmcaleer/SnakeROS)
- 文档
  - [Getting started](https://github.com/kevinmcaleer/snakeros/blob/main/docs/index.md)
  - [Architecture](https://github.com/kevinmcaleer/snakeros/blob/main/docs/architecture.md)
  - [Why not DDS?](https://github.com/kevinmcaleer/snakeros/blob/main/docs/architecture.md#why-not-dds-directly)
  - [API](https://github.com/kevinmcaleer/snakeros/blob/main/docs/api.md)
  - [Messages](https://github.com/kevinmcaleer/snakeros/blob/main/docs/messages.md)
  - [Transports](https://github.com/kevinmcaleer/snakeros/blob/main/docs/transports.md)
  - [Memory](https://github.com/kevinmcaleer/snakeros/blob/main/docs/memory.md)
  - [Troubleshooting](https://github.com/kevinmcaleer/snakeros/blob/main/docs/troubleshooting.md)
  - [Limitations](https://github.com/kevinmcaleer/snakeros/blob/main/docs/limitations.md)
