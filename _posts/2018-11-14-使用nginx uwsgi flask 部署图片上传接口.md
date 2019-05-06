---
layout:     post
title:      使用nginx uwsgi flask 部署图片上传接口  
subtitle:   nginx uwsgi flask
date:       2018-11-14
author:     WLX
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - python
---

## 安装nginx uwsgi flask
安装nginx： 略
安装uwsgi： pip install uwsgi
安装flask： pip install flask 

## 配置 nginx

 - nginx作为图片资源的访问入口
 - nginx作为uwsgi的代理
 
nginx的配置如下，11000端口为uwsgi的代理， 12000端口为图片资源：
```
server {
        listen       11000;
        location / {            
            include  uwsgi_params;
            uwsgi_pass  127.0.0.1:9090;
            uwsgi_param UWSGI_SCRIPT wsgi:app;
            uwsgi_param UWSGI_PYHOME /home/trio/app/base/python;
            uwsgi_param UWSGI_CHDIR /home/trio/TrioApi/up_pic;
            index  index.html index.htm;
            client_max_body_size 35m;
        }
}

server {
	listen 12000;
	location / {
		root   /home/trio/TrioApi/up_pic/uploads;

	}
	location ~ .*\.(gif|jpg|jpeg|png)$ {  
        expires 24h;  
        root /home/trio/TrioApi/up_pic/uploads;
        client_max_body_size    50m;  
        client_body_buffer_size 12800k;  
    }
}
```

说明：

 - uwsgi_pass 为uwsgi的监听地址
 - uwsgi_param UWSGI_SCRIPT wsgi:app  代表flask的项目目录中的wsgi.py文件的app。
    这里要注意的是flask的app文件，要有这几行
```
    app = Flask(__name__)
    if __name__ == "__main__":
        app.run()
```
 - uwsgi_param UWSGI_CHDIR 为flask的项目目录
 - uwsgi_param UWSGI_PYHOME 为python的路径，因为我的python是源码装的。如果是yum装的，则不需要填这个属性

## 配置 uwsgi
在/home/trio/app/base/python下新建uwsgi9090.ini
```
[uwsgi]
socket = 127.0.0.1:9090
master = true
vhost = true
#touch-reload = /home/trio/TrioApi/up_pic
no-site = true
workers = 2
reload-mercy = 10     
vacuum = true 
max-requests = 1000   
limit-as = 512
buffer-size = 30000
pidfile = /home/trio/app/base/python/uwsgi9090.pid
daemonize = /home/trio/app/base/python/uwsgi9090.log
```
说明：
 - socket为uwsgi监听的ip和端口。
 - touch-reload 指定目录，目录中文件变化，自动reload uwsgi

## 配置flask
创建项目目录，/home/trio/TrioApi/up_pic
然后创建入口文件wsgi.py
```python
from flask import Flask,render_template,request
from werkzeug import secure_filename
import hmd5
import os
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'loo World!'

@app.route('/upload', methods=['POST', 'GET'])
def upload():
    if request.method == 'POST':
        f = request.files['file']
        basepath = os.path.dirname(__file__)
        upload_path = os.path.join(basepath, 'uploads/')
        fff = upload_path + secure_filename(f.filename)
        f.save(fff)
        md5 = hmd5.GetFileMd5(fff)
        if '.png' in fff or '.jpg' in fff or '.gif' in fff:
            new_fff = upload_path + md5 + '.' + os.path.basename(fff).split('.')[1]
        else:
             new_fff = upload_path + md5
        os.rename(fff, new_fff)
        pic_url = 'http://40.123.110.211:12000/' + os.path.basename(new_fff)
        return render_template('upload.html', var1 = pic_url)
    return render_template('upload.html')

if __name__ == "__main__":
    app.run(debug=True)
```
hmd5.py
```python
#!/usr/bin/env python

import hashlib                    
import os
                                  
def GetFileMd5(filename):
    if not os.path.isfile(filename):
        return
    myhash = hashlib.md5()
    f = file(filename,'rb')
    while True:
        b = f.read(8096)
        if not b :
            break
        myhash.update(b)
    f.close()
    return myhash.hexdigest()
```
创建templates模板目录，模板文件upload.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>文件上传示例</h1>
    <form action="" enctype='multipart/form-data' method='POST'>
        <input type="file" name="file">
        <input type="submit" value="上传">
    </form>
    <h1>图片地址如下</h1>
    <b>{{var1}}</b>
</body>
</html>
```

效果：
上传前：
![](http://40.125.164.174:12000/67809f3141428cf810da5f57b76009a1.png)

上传后：
![](http://40.125.164.174:12000/ae59fa461b862b83f6bac9534af04d21.png)
