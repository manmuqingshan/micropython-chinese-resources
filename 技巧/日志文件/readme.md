# 日志文件

在系统中，日志文件是一个非常有用的功能，可以记录运行中的事件、故障、状态等，方便系统后续的维护、分析、回溯等等。

下面是一个基本的日志文件函数，提供了基本的日志记录功能，能够指定文件名、设置日志文件最大大小、超过预设大小后自动改名、追加消息到日志文件、自动添加时间戳等功能，可以在此基础上修改，添加更多功能。

```python
import time, os

def filesize(filename):
    try:
        return os.stat(filename)[6]
    except:
        return -1

def runlog(msg, filename='log.txt', maxsize=50000):
    try:
        if filesize(filename) > maxsize:
            os.rename(filename, filename+'.1')

        r = time.localtime()
        with open(filename, 'at') as f:
            f.write(f'{r[0]}-{r[1]:02}-{r[2]:02} {r[3]:02}:{r[4]:02}:{r[5]:02} {msg}\n')
            
    except Exception as e:
        print('', e)
```
