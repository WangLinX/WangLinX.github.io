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
    print('>>>>>>>')
    print(recv_data)
    result = re.search(r"[^/]+/([^ ]*)", recv_data.splitlines()[0])
    if result:
        uri = result.group(1)
    document_root = 'C:\\Users\\wlx\\Desktop\\'
    if uri:
        path = document_root + uri
    else:
        path = document_root + "index.html"
    try:
        f = open(path, 'rb')
    except:
        send_header = 'HTTP/1.1 404 NOT FOUND\r\n\r\n'
        send_body = '--- file not found ---'.encode('utf-8')
    else:
        send_body = f.read()
        send_header = 'HTTP/1.1 200 OK\r\n\r\n'
        f.close()
    conn.send(send_header.encode("utf-8"))
    conn.send(send_body)
    conn.close()


def main():
    sk = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sk.bind(('0.0.0.0', 9369))
    sk.listen(5)
    while True:
        print("Waiting for connection......")
        conn, addr = sk.accept()
        tt = threading.Thread(target=recv, args=(conn, addr))
        tt.start()
    sk.close()


if __name__ == '__main__':
    main()
```