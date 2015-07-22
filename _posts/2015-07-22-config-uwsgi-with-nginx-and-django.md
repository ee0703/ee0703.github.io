---
layout:   post
title:    uWSGI＋Nginx＋Django安装和配置
date:     2015-07-22 11:00
summary:  uWSGI＋Nginx＋Django安装和配置
---


WSGI是为python语言定义的通用网关接口，它承担python web框架（django、flask、web.py等）和web服务器（nginx、apache、lighttpd等）之间的中间层。
```
    浏览器                      chrome、firefox、ie等
      ｜
    web服务器                   nginx、apache等
      ｜
    网关接口                    CGI、FastCGI、WSGI等
      ｜
    Python（程序、Web框架）     Django、Flask、Tornado等
```
python中自带的wsgiref就是一种wsgi接口的标准实现，但是，由于100%使用python实现等原因，导致wsgiref实在过于缓慢，只能用于测试和学习。生产环境中我们需要使用性能更高的服务器，目前常用的wsgi服务器有：uWSGI、Gunicorn、twisted.web。

## 1 uWSGI的安装
uWSGI是用C语言写的高性能WSGI服务器，安装uWSGI前我们需要安装Python和C编译器（GCC）。推荐使用python包管理器pip安装uWSGI。
```sh
#安装最新稳定版
pip install uWSGI
#也可以安装长期支持版（LTS版本）
#pip install http://projects.unbit.it/downloads/uwsgi-lts.tar.gz
```

在ubuntu下可以使用apt-get来安装
```sh
apt-get install uwsgi  
```

在fedora、redhat、centos下使用yum安装
```sh
yum groupinstall "Development Tools"
yum install python  
```

编译安装，从github下载uwsgi代码，cd到目录下
```sh
python uwsgiconfig.py --build
```

## 2 测试uwsgi是否安装成功
在终端中输入以下命令查看uwsgi的版本号，如果输出正常，说明uswgi已安装成功
```sh
$ uwsgi --version
2.0.11.1
```

我们可以编写一个简单的wsgi应用来测试uwsgi是否被安装成功，首先创建一个test.py文件：
```python
# test.py
def application(env, start_response):
    start_response('200 OK', [('Content-Type','text/html')])
    return [b"Hello World"] # python3
    #return ["Hello World"] # python2
```

运行uwsgi：
```sh
uwsgi --http :8000 --wsgi-file test.py
```

参数中，http :8000表示使用http协议，端口号为8000，wigi-file则表示要运行的wsgi应用程序文件。uwsgi运行后打开浏览器，访问http://127.0.0.1:8000/ ，或者是相应服务器地址的8000端口，就可以看到hello world 页面了。  

上面的例子中,我们用浏览器直接访问了uwsgi运行的python程序(只有一个入口函数的wsgi测试应用test.py)，其访问结构如下所示。
```
    浏览器 <-> uWSGI <-> Python
```

上述方式运行uWSGI服务的过程中，可以使用CTRL＋C即可停止服务，在后续的章节中会讲到自动管理和部署。

## 3 nginx和django的配置
nginx和django的安装不是本文的重点，故在此略去，只讨论配置部分。在这里，我们要实现的效果如下：
```
    浏览器 <-> nginx <-> uWSGI <-> Django(python)
```
### uwsgi_params 配置文件
uWSGI使用的协议不完全是标准的WSGI协议，我们需要从[Github](https://github.com/nginx/nginx/blob/master/conf/uwsgi_params)下载uwsgi_paraments配置文件，并将该文件拷贝到项目路径中（例如：/user/home/pengquanxin/projects/mysite1/）。  

### Nginx服务器配置
接下来，要配置nginx服务器和uWSGI互通，可以使用unix套接字方式和TCP端口方式。在nginx配置文件夹（/etc/naginx/site-enabled 或 /usr/local/etc/nginx/sites-enabled）中新建网站的配置文件mystie_nginx.conf，输入以下内容：
```
# mysite_nginx.conf

# nginx需要连接的上游
upstream django {
    server unix:///path/to/your/mysite/mysite.sock; # 使用unix套接字
    #server 127.0.0.1:8001; # 使用TCP端口请注释上一行，并取消本行注释，这里的端口指的是跑uwsgi的端口
}

# nginx服务器配置
server {
    # 监听端口
    listen      80;
    # 域名
    server_name .example.com; 
    # 编码
    charset     utf-8;

    # 最大上传大小
    client_max_body_size 75M;   

    # Django 的media路径
    location /media  {
        alias /path/to/your/mysite/media;  
    }

    # 静态文件路径
    location /static {
        alias /path/to/your/mysite/static; 
    }

    # 将动态请求转发到uwsgi跑的django程序
    location / {
        uwsgi_pass  django;
        include     /path/to/your/mysite/uwsgi_params; # 从github上下载的uwsgi_params 文件路径
    }
}
```

你也可以把这个配置文件放在项目路径中，然后建立一个链接到nginx配置文件夹：
```
sudo ln -s ~/path/to/your/mysite/mysite_nginx.conf /etc/nginx/sites-enabled/
```

### 部署静态文件
在部署服务器之前，需要先将Django的静态文件部署到静态文件夹中，首先，编辑django网站的settings.py文件
```python
STATIC_ROOT = os.path.join(BASE_DIR, "static/")
```
然后，运行以下命令
```
python manage.py collectstatic
```

## 4 启动服务
在启动nginx之前，我们需要先启动uWSGI，进入项目目录然后输入以下命令，在这里我们使用unix套接字方式：
```
#注：django1.6 前的版本需要手动添加wsgi.py
uwsgi --socket mysite.sock
```

如果nginx和uwsgi跑在同一台服务器上，使用unix套接字就可以了，unix套接字方式性能要高很多，但不能跨机器访问。当nginx和uWSGI不在一台服务器上时，就需要使用TCP端口方式（别忘了更改nginx配置文件，取消相应注释）：
```
uwsgi --socket :8001 --module mysite.wsgi --chmod-socket=664
```

接下来，启动nginx服务器，就可以访问django站点了。

## 5 使用ini配置文件跑uWSGI
到这里，我们已经把nginx＋uWSGI＋Django跑起来了，但uWSGI的参数比较多的时候，每次都要输入非常麻烦，这时，我们可以在django项目目录下建立一个mysite.uwsgi.ini 
```
[uwsgi]
# 项目根目录路径(full path)
chdir           = /path/to/your/project
# Django的 wsgi 文件
module          = mysite.wsgi
# virtualenv目录 (full path)
home            = /path/to/virtualenv

master          = true
# 最大工作进程数（CPU密集型建议设为CPU核心数，IO密集型建议设为CPU核心数的两倍） 
processes       = 16
# unix套接字文件路径
socket          = /path/to/your/project/mysite.sock
# socket文件权限
# chmod-socket    = 664
# 退出时清空环境
vacuum          = true
```
然后，直接根据配置文件运行uwsgi即可：
```
uwsgi --ini mysite.uwsgi.ini 
```

## 6 管理uwsgi
### Emperor模式
uWSGI的Epreror模式可以用来管理机器上部署的uwsgi服务，在这种模式下，会有一个特殊的进程（皇帝）对其它部署的服务（诸侯）进行监视。我们将所有配置文件（ini或xml文件，如上一节中的mysite.uwsgi.ini）统一放到一个文件夹（如：/etc/uwsgi/vassals）中，然后启动Emperor模式：
```
uwsgi --emperor /etc/uwsgi/vassals
```

这样，就会自动读取文件夹中的配置文件，并自动监控这些uwsgi服务：
- 检测文件夹中有新的配置文件时，会启动新的uwsgi服务实例
- 检测到一个配置文件发生改变，会自动重启该服务
- 检测到一个配置文件被移除，则自动停止该服务
- 如果一个服务死了（诸侯），皇帝进程会重启该服务
- 如果监控进程（皇帝）死了，所有服务（诸侯）都会停止

###  用systemd管理uwsgi服务
配合Eperor模式，在centos、fedora、archlinux中，我们可以用systemd来管理uwsgi，首先，创建一个systemd service文件（/etc/systemd/system/emperor.uwsgi.service）
```
[Unit]
Description=uWSGI Emperor
After=syslog.target

[Service]
ExecStart=/root/uwsgi/uwsgi --emperor /etc/uwsgi/vassals
Restart=always
KillSignal=SIGQUIT
Type=notify
StandardError=syslog
NotifyAccess=all

[Install]
WantedBy=multi-user.target
```

这样我们就可以用systemd来管理uwsgi服务了。启动服务：
```
$ systemctl start emperor.uwsgi.service
```
查询服务运行状态：
```
$ systemctl status emperor.uwsgi.service
```
停止服务
```
systemctl stop emperor.uwsgi.service
```

linux系统中，还有一种通用的方法，就是在init.d 或 rc.d 中加入启动脚本，这种方式不够智能，而且网上资料很多，在这里暂不讨论。

## 7 常用参数和选项
关于参数的具体使用，可以阅读官方文档http://uwsgi-docs.readthedocs.org/en/latest/Options.html ，在这里列出一些常用的参数：

- chdir         项目目录
- home          virtualenv目录（如没有运行virtualenv虚拟环境，则无需设置）
- socket        套接字文件或TCP套接字，例如：site1.uwsgi.sock 或 127.0.0.1:8000
- uid           用户id
- gid           用户组id
- processes     工作进程数
- harakiri      进程超过该时间未响应就重启该进程（默认单位为秒）
- module        要启动的wsgi模块入口，如：mysite.wsgi:application
- ini           指定ini配置文件
- xml           指定xml配置文件（与ini类似）
- file          指定要运行的wsgi程序文件，如：test.py
- emperor       Emperor模式
- so-keepalive  开启TCP KEEPALIVE（unix套接字方式下无效）
- vacuum        退出时清空环境

