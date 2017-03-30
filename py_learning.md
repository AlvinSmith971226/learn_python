## 错误调试和测试
#### 错误处理
try...except...finally...处理机制  
例如  
```py
  try:
      print('try...')
      r = 10 / int('2')
      print('result:', r)
  except ValueError as e:
      print('ValueError:', e)
  except ZeroDivisionError as e:
      print('ZeroDivisionError:', e)
  else:
      print('no error!')
  finally:
      print('finally...')
  print('END')
```
Python的错误其实也是class，所有的错误类型都继承自`BaseException`，所以在使用`except`时需要注意的是，它不但捕获该类型的错误，还把其子类也“一网打尽”  
Python所有的错误都是从`BaseException`类派生的，常见的错误类型和继承关系看[这里](https://docs.python.org/3/library/exceptions.html#exception-hierarchy)

#### 调用堆栈
根据错误类型，可以追寻错误信息找到出错的语句  

#### 记录错误
使用`logging`模块记录错误信息
```py
# err_logging.py

import logging

def foo(s):
    return 10 / int(s)

def bar(s):
    return foo(s) * 2

def main():
    try:
        bar('0')
    except Exception as e:
        logging.exception(e)

main()
print('END')
```
出错后，`logging`把错误记录到日志文件里，方便排查

#### 抛出错误
可以自己定义一个错误的class，选择好继承关系，然后用`raise`抛出错误  
```py
# err_raise.py
class FooError(ValueError):
    pass
```
调用时 `raise FooError('balabala...')`  
另外，也可以在补货一个错误后，将此错误抛出，以方便记录错误，方便排错  


## 调试
#### 断言`assert`  
```py
def foo(s):
    n = int(s)
    *assert n != 0, 'n is zero!'*
    return 10 / n

def main():
    foo('0')
```
若断言失败，`assert`会抛出`AssertionError`:  
```
$ python3 err.py
Traceback (most recent call last):
  ...
AssertionError: n is zero!
```
启动Python解释器时可以通过``-O``参数关闭`assert`,此时`assert`可以看成`pass`  

#### `logging`
和`assert`比，`logging`不会抛出错误，而且可以输出到文件  
```py
import logging
logging.basicConfig(level=logging.INFO)
s = '0'
n = int(s)
logging.info('n = %d' % n)
print(10 / n)
```
运行输出
```
$ python3 err.py
INFO:root:n = 0
Traceback (most recent call last):
  File "err.py", line 8, in <module>
    print(10 / n)
ZeroDivisionError: division by zero
```
这就是`logging`的好处，它允许你指定记录信息的级别，有`debug`，`info`，`warning`，`error`等几个级别，当我们指定`level=INFO`时，`logging.debug`就不起作用了。同理，指定`level=WARNING`后，`debug`和`info`就不起作用了。这样一来，你可以放心地输出不同级别的信息，也不用删除，最后统一控制输出哪个级别的信息


## 常用模块
#### datetime
```py
from datetime import datetime
now = datetime.now()#获取当前时间
print(now)

dt = datetime(2015, 4, 19, 12, 20)#要指定某个日期和时间，我们直接用参数构造一个datetime
# >>> print(dt)
# 2015-04-19 12:20:00
```
将datetime转换为timestamp  
```py
from datetime import datetime
dt = datetime(2017, 3, 30, 11, 01)
dt.timestamp()
# >>>
```
将timestamp转换为datetime  
```py
from datetime import datetime
t = 1490842996.489678
print(datetime.fromtimestamp(dt))
# >>>2017-03-30 11:03:16.489678
```

str转换为datetime  
```py
from datetime import datetime
cday = datetime.strptime('2017-03-30 11:07:00', '%Y-%m-%d %H:%M:%S')
print(cday)
# >>>2017-03-30 11:07:00

```

将datetime转换为str  
```py
from datetime import datetime
now = datetime.now()
print(now.strftime('%a %b %d %H:%M'))
```

datetime加减   
```py
from datetime import datetime, timedelta
now = datetime.now()
now += timedelta(hours=10)
print(now)
# >>>2017-03-30 21:17:00.279967
```

本地时间转换为UTC时间  
本地时间是指系统设定时区的时间，例如北京时间是UTC+8:00时区的时间，而UTC时间指UTC+0:00时区的时间。  
```py
from datetime import datetime, timedelta, timezone
tz_utc_8 = timezone(timedelta(hours=8))
now = datetime.now()
print(now)

dt = now.replace(tzinfo=tz_utc_8)#强制设置为UTC+8:00
print(dt)
```

时区转换  
```py
# 拿到UTC时间，并强制设置时区为UTC+0:00:
>>> utc_dt = datetime.utcnow().replace(tzinfo=timezone.utc)
>>> print(utc_dt)
2015-05-18 09:05:12.377316+00:00
# astimezone()将转换时区为北京时间:
>>> bj_dt = utc_dt.astimezone(timezone(timedelta(hours=8)))
>>> print(bj_dt)
2015-05-18 17:05:12.377316+08:00
# astimezone()将转换时区为东京时间:
>>> tokyo_dt = utc_dt.astimezone(timezone(timedelta(hours=9)))
>>> print(tokyo_dt)
2015-05-18 18:05:12.377316+09:00
# astimezone()将bj_dt转换时区为东京时间:
>>> tokyo_dt2 = bj_dt.astimezone(timezone(timedelta(hours=9)))
>>> print(tokyo_dt2)
2015-05-18 18:05:12.377316+09:00
```

*如果要存储datetime，最佳方法是将其转换为timestamp再存储，因为timestamp的值与时区完全无关。*

<!-- ### 练习 -->
<!-- 假设你获取了用户输入的日期和时间如2015-1-21 9:01:30，以及一个时区信息如UTC+5:00，均是str，请编写一个函数将其转换为timestamp：   
```py
import re
from datetime import datetime, timezone, timedelta


def to_timestamp(dt_str, tz_str):
  now = re.match(r'^(\d{4})-(\d+)-(\d+) (\d+):(\d+):(\d+)$', dt_str)
  utc_num = re.match(r'^UTC+(\d+):00',tz_str)
  cday = datetime.strptime(now.group(1)-now.group(2)-now.group(3) now.group(4):now.group(5):now.group(1), '%Y-%m-%d %H:%M:%S')
  tar_dt = cday.astimezone(timezone(timedelta(hours=utc_num.group(1))))
  return tar_dt -->



```
