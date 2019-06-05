---
layout:     post
title:      用Python写类似tail -f命令的脚本监控文件
subtitle:   用Python写类似tail -f命令的脚本监控文件
date:       2019-05-31
author:     WLX
header-img:  
catalog: 	 true
tags:
    - 用Python写类似tail -f命令的脚本监控文件
---


```python
import time
import sys,os

if len(sys.argv) > 1:  #to determine if provide a parameter
    filename = sys.argv[1] #fetch the 1st parameter
    if not filename:
        print('Please provide a file that you want to monitor')
        sys.exit()
else:
    print('Please provide a parameter to specify filename')
    sys.exit()

if not os.path.isfile(filename):
    print('Please provide a valid filename')
    sys.exit()

def tail(filename):
    with open(filename,'r') as f:
        f.seek(0,2)
        while True:
            line = f.readline()
            if not line:
                time.sleep(0.1)
                continue
            yield line

try:
    for line in tail(filename):
        print(line)
except KeyboardInterrupe as keys:
    sys.exit()
```

类相关问题：

- 调用父类，父类则执行一次，如果有多个类继承了父类， 然后有类又继承了这些类， 那么这个类调用一次，这父类会被调用多次，资源浪费。
- 解决方法：调用父类的时候使用super()函数， 参数使用*args, **kargs

```python

class Parent(Object):
    def __init__(self, name, *args, **kwargs):
        print("xxx Parent xxx")
        self.name = name
        print("yyy Parent yyy")

class Son1(Parent):
    def __init__(self, name, age, *args, **kwargs):
        print("xxx Son1 xxx")
        self.age = age
        super().__init__(self, *args, **kwargs)
        print("yyy Son2 yyy")

class Son2(Parent):
    def __init__(self, name, gender, *args, **kwargs):
        print("xxx Son1 xxx")
        self.gender = gender
        super().__init__(self, *args, **kwargs)
        print("yyy Son2 yyy")

class Grandson(Son1, Son2):
    def __init__(self, name, age, gender):
        print("xxx Grandson xxx")
        # 多继承时，相对于使用类名，_init__方法，要把每个父类全部写一遍
        # 而super只用一句话， 执行了全部父类的方法， 这也是为何多继承需要全部传参的一个原因
        # spuer(Grandson, self).__init__(name, age, gender)
        super().__init__(name, age, gender)
        print("yyy Grandson yyy")

print(Grandson.__mro__)

```
- C3 算法，决定了Grandson.__mro__的结果
- 当调用super()会拿当前类的名字去mro元组去匹配，去元组中直接调用他的下一个类去执行。
- 这样的时候，Parent
- 调用父类类的三种方式：
    1. 类名+方法： Parent.__init__()
    2. super() + 方法: super().__init__() 拿着当前类的类名去mro元组中查找，调用其下一个类
    3. super(类名, self) + 方法: spuer(Grandson, self).__init__(name, age, gender) 拿着指定的Grandson类名，去mro元组中查找，调用其下一个类