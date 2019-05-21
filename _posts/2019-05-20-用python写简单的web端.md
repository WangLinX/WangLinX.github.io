---
layout:     post
title:      用python写简单的web端
subtitle:   用python写简单的web端
date:       2019-05-11
author:     WLX
header-img:  
catalog: 	 true
tags:
    - python
---

- http协议


```python
#!/usr/bin/env python3


'''
web服务器
1. 服务端接收： 接收tcp连接， http协议， 解析请求
2. 客户端发送： 返回200， 返回数据。 定义返回数据的函数
3. socket tcp
4. 多线程或者协程，同时收发。
'''

import socket
import threading
import re


def recv(conn, addr):
    recv_data = ''
    data = conn.recv(1024)
    recv_data += data.decode("utf-8")
    result = re.search(r"GET .+ HTTP/1.1", recv_data)
    if result:
        print(result)
        # uri = result.group().split()[1]
        send_data = '''HTTP/1.1 200 OK\r\n\r\n
        <!DOCTYPE html>
        <html>
        <h1>welcome, %s<h1>
        </html>
        ''' % str(addr)
        conn.send(send_data.encode("utf-8"))
        conn.close()


def main():
    sk = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sk.bind(('0.0.0.0', 9369))
    sk.listen(5)
    while True:
        print("Waiting for connection......")
        conn, addr = sk.accept()
        print(conn)
        print(addr)
        tt = threading.Thread(target=recv, args=(conn, addr))
        tt.start()
    sk.close()


if __name__ == '__main__':
    main()
```