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
