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
    if not data:
        conn.close()
    recv_data += data.decode("utf-8")
    print('>>>>>>>')
    print(recv_data)
    result = re.search(r"[^/]+/([^ ]*)", recv_data.splitlines()[0])
    if result:
        uri = result.group(1)
    else:
        uri = index.html
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

- 用gevent实现
1. 此程序是服务端主动断开连接。

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
import re
import gevent
from gevent import monkey
monkey.patch_all()


def recv(conn, addr):
    recv_data = ''
    data = conn.recv(1024)
    if not data:
        conn.close()
    recv_data += data.decode("utf-8")
    print('>>>>>>>')
    print(recv_data)
    result = re.search(r"[^/]+/([^ ]*)", recv_data.splitlines()[0])
    if result:
        uri = result.group(1)
    else:
        uri = 'index.html'
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
        gevent.spawn(recv, conn, addr)
    sk.close()


if __name__ == '__main__':
    main()

```

- 如果接收端很大， 此时用浏览器访问时，浏览器会一直转圈。原因是浏览器一直会发TCP Keep-Alive包给服务器端，
服务端接收到后，认为客户端仍没有发完数据，就会一直进入阻塞状态。

```python
    while True:
        try:
            data = conn.recv(1024)
        except Exception as ret:
            pass
        else:
            if data:
                ....
            else:
                conn.close()
                break

```

- epoll版web服务端
1. epoll使用了内存映射技术（mmap）， 应用程序和kernel共享一块内存
2. epoll基于事件的就绪通知方式， 内存中的fd就绪，才通知，不会去轮询内存中的每个fd了
3. 以前的select方式叫轮询且有个数限制， poll也是轮询，但没有个数限制。

```python
#!/usr/bin/env python36
# coding=utf-8

'''
web服务器
1. 服务端接收： 接收tcp连接， http协议， 解析请求
2. 客户端发送： 返回200， 返回数据。 定义返回数据的函数
3. socket tcp
4. 多线程或者协程，同时收发。
'''

import socket
import re
import select


def recv(conn, addr, data):
    recv_data = data.decode("utf-8")
    print('>>>>>>>')
    print(recv_data)
    result = re.search(r"[^/]+/([^ ]*)", recv_data.splitlines()[0])
    if result:
        uri = result.group(1)
    else:
        uri = 'index.html'
    document_root = '/home/wanglinxiang/html/'
    if uri:
        path = document_root + uri
    else:
        path = document_root + "index.html"
    print(path)
    try:
        f = open(path, 'rb')
    except:
        send_body = '--- file not found ---'.encode("utf-8")
        send_header = 'HTTP/1.1 404 NOT FOUND\r\n'
        send_header += 'Content-Length: %d\r\n\r\n' % len(send_body)
    else:
        send_body = f.read()
        send_header = 'HTTP/1.1 200 OK\r\n'
        send_header += 'Content-Length: %d\r\n\r\n' % len(send_body)
        f.close()
    print(addr)
    print(send_header)
    conn.send(send_header.encode("utf-8"))
    conn.send(send_body)



def main():
    sk = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

    # 设置套接字端口可以复用
    sk.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

    sk.bind(('0.0.0.0', 9369))
    sk.listen(5)

    # 将套接字设置为非阻塞
    sk.setblocking(False)

    # 创建epoll对象
    epl = select.epoll()

    fd_socket_dict = {}

    # 将监听套接字对应的fd注册到epoll中
    epl.register(sk.fileno(), select.EPOLLIN)

    while True:
        fd_event_list = epl.poll() # 默认会阻塞， 直到os检测到数据到来 通过事件通知方式告诉这个程序， 此时才会解阻塞, 返回一个列表
        # fd_event_list --> [(fd, event), ... ] (套接字对应的文件描述符， event表示是什么事件，例如 可以调用recv接收等)
        for fd, event in fd_event_list:
            if fd == sk.fileno():
                conn, addr = sk.accept()
                epl.register(conn.fileno(), select.EPOLLIN)
                fd_socket_dict[conn.fileno()] = conn, addr
            elif event == select.EPOLLIN:
                # 判断已经链接的客户端释放有数据发送过来
                data = fd_socket_dict[fd][0].recv(1024)
                if data:
                    recv(fd_socket_dict[fd][0], fd_socket_dict[fd][1], data)
                else:
                    fd_socket_dict[fd][0].close()
                    fd_socket_dict.pop(fd)
                    epl.unregister(fd)
    epl.unregister(sk.fileno())
    sk.close()


if __name__ == '__main__':
    main()
```