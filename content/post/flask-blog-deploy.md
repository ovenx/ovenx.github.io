---
title: flask-blog 部署
date: 2020-02-09 20:27:00
categories:
- 程序世界
tags: ["python", "flask"]
---

本文的 python 环境为 python3.8.1

预览地址：[首页](https://flask-blog.ovenx.cn/) 、[后台管理](https://flask-blog.ovenx.cn/admin)

### 安装 flask-blog
clone 代码到本地
```bash
cd /www
git clone https://github.com/ovenx/flask-blog.git
```

安装虚拟环境
```bash
python -m venv venv
. venv/bin/activate
```

安装依赖
```bash
pip install -r requirements.txt
```

创建数据库,导入数据表
```sql
create database flask-blog
source /www/flask-blog/blog/schema.sql
```

创建配置文件，修改数据库配置

```bash
cp /www/flask-blog/blog/config.py.example /www/flask-blog/blog/config.py
```

### 使用 gunicorn
flask 自带的 server 不能用于生产环境，需要使用其他的 server 替代，一般会使用 `gunicorn`。
```bash
gunicorn -w4 -b127.0.0.1:8000 run:app // 在venv中启动
```
此时我们使用了 8000 来访问，替代了原先的端口。-w 表示 worker 数量 -b 表示端口地址。



到这里其实我们已经可以正常运行 flask-blog 程序了，用 nginx 反代 8000 端口即可。但是 gunicorn 管理起来比较麻烦，为了管理方便我们需要使用 supervisor 来管理 gunicorn 进程。

### 使用 supervisor
首先安装 supervisor
```bash
pip install supervisor
echo_supervisord_conf > /etc/supervisord.conf // 生成配置文件
```
编辑配置文件 /etc/supervisord.conf，添加 gunicorn
```bash
[program:flask-blog]
command=sh -c 'source "$0" && exec "$@"' venv/bin/activate gunicorn -w4 -b 127.0.0.1:8000 run:app
directory=/www/flask-blog
autostart=true
autorestart=true
redirect_stderr=true
stdout_logfile=/var/log/flask-blog.log
stderr_logfile=/var/log/flask-blog.log
stopasgroup=true
stopsignal=QUIT
```
这里注意需要先启动 virtualenv


然后启动 supervisord，默认会启动所有的服务
```bash
supervisord -c /etc/supervisord.conf //启动
supervisorctl shutdown // 关闭
```


管理某个服务
```bash
supervisorctl start flask-blog #启动
supervisorctl stop flask-blog  #停止
```

### 配置 nginx 反代
创建 /etc/nginx/conf.d/flask-blog.conf，https 配置参考上一篇[《let's encrypt 小记》](/2017/10/12/let's-encrypt/)

```bash
server{
    listen 443 http2 ssl;
    server_name www.domain_1.com;
    ssl_certificate /etc/letsencrypt/live/www.domain_1.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/www.domain_1.com/privkey.pem;
    include /etc/nginx/default.d/ssl.conf;
    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```
重启 nginx，大功告成，访问 https://www.domain_1.com/admin 来添加文章吧！