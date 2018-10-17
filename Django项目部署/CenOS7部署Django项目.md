# CentOS7部署Django项目  

## 1. 云服务器  
这里使用的是腾讯云  
选择系统：CentOS7.3  

记住云服务器登录密码  

## 2. 配置Python3环境  
默认Python环境为python2.7,yum安装是需要python2的环境的  

安装Python3:  
```py
yum install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gcc make
```
这几个包必须得安装，否则安装python3时可能会出现各种错误   
运行下面两个命令，进行备份  
```py
cd /usr/bin
mv python python.bak
```

安装Python3.6与解压命令  
```py
wget https://www.python.org/ftp/python/3.6.3/Python-3.6.3.tar.xz
tar -xvJf  Python-3.6.3.tar.xz
cd Python-3.6.3
```
  
编译安装  
```py
./configure prefix=/usr/local/python3
make && make install
```
安装完毕，/usr/local/目录下就会有python3了  

## 3. 实现python3和python2的共存  
添加python3的软链接  
```py
rm /usr/bin/python
ln -s /usr/local/python3/bin/python3 /usr/bin/python
ln -s /usr/local/python3/bin/python3 /usr/bin/python3
```
这时候在执行命令python -v和python2 -V，应该就能看到python3和python2的版本了。   
因为执行yum需要python2版本，所以我们还要修改yum的配置，执行：  
```py
vi /usr/bin/yum
```
把#! /usr/bin/python修改为#! /usr/bin/python2   
同样地  
```py
vi /usr/libexec/urlgrabber-ext-down 

cd /usr/bin/
ls yum*

vim yum-config-manager
vim yum-debug-restore
vim yum-group-manager
vim yum-build-dep
vim yum-debug-dump
vim yumdownloader
```
把#! /usr/bin/python都修改为#! /usr/bin/python2  

> 相关文档：  
> [centos7+django+python3+mysql+阿里云部署项目全流程](https://blog.csdn.net/a394268045/article/details/79288718?utm_source=blogxgwz5)  
> [Flask构建弹幕微电影网站- 部署上线](https://www.jianshu.com/p/0f2eb99a5393)  
> [centos7 下通过nginx+uwsgi部署django应用](http://projectsedu.com/)  

## 4. 安装MySQL5.7
下载MySQL源安装包  
```py
wget http://dev.mysql.com/get/mysql57-community-release-el7-8.noarch.rpm
```
安装MySQL源  
```py
yum localinstall mysql57-community-release-el7-8.noarch.rpm
yum install mysql-devel
```
安装MySQL  
```py
yum install mysql-community-server
```
启动MySQL  
```py
systemctl start mysqld
```
查看MySQL的启动状态  
```py
systemctl status mysqld
```
设置开机启动  
```py
systemctl enable mysqld
```

修改 root 本地登录密码  
```py
grep 'temporary password' /var/log/mysqld.log
```
上面命令显示root初始密码  
这里登入MySQL进行修改密码  
```py
mysql -uroot -p
enter password: 粘贴上面显示的初始密码
mysql > set password for 'root'@'localhost'=password('!2Qw32sd');
```
配置默认编码为utf8   
修改/etc/my.cnf配置文件，在[mysqld]下添加编码配置，如下所示：  
```py
[mysqld]
character_set_server=utf8
init_connect='SET NAMES utf8'
```

## 5. 安装nginx
sudo yum install epel-release  
sudo yun install nginx  
sudo systemctl start nginx  

访问：  
http://云服务器的域名或者IP/

> 官方文档  
> https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-centos-7  


## 6. 安装virtualenvwrapper
```py
yum install python-setuptools python-devel  
pip install virtualenvwrapper  
```
编辑.bashrc文件  
```py
export WORKON_HOME=$HOME/.virtualenvs
source /usr/local/bin/virtualenvwrapper.sh
```
重新加载.bashrc文件  
source  ~/.bashrc  

新建虚拟环境  
mkvirtualenv mxonline  

进入虚拟环境   
workon mxonline  

我们可以通过 pip freeze > requirements.txt 将本地的虚拟环境安装包相信信息导出来  
然后将requirements.txt文件上传到服务器之后运行：  

## 7. 安装依赖包   
pip install -r requirements.txt  
pip install uwsgi  


## 8. 配置nginx  
新建uc_nginx.conf  
```py
# the upstream component nginx needs to connect to
upstream django {
# server unix:///path/to/your/mysite/mysite.sock; # for a file socket
server 127.0.0.1:8000; # for a web port socket (we'll use this first)
}
# configuration of the server

server {
# the port your site will be served on
listen      80;
# the domain name it will serve for
server_name 132.232.183.18; # substitute your machine's IP address or FQDN
charset     utf-8;

# max upload size
client_max_body_size 75M;   # adjust to taste

# Django media
location /media  {
    alias /home/git/Mx_Online/media;  # 指向django的media目录
}

location /static {
    alias /home/git/Mx_Online/static; # 指向django的static目录
}

# Finally, send all non-media requests to the Django server.
location / {
    uwsgi_pass  django;
    include     uwsgi_params; # the uwsgi_params file you installed
}
}
```

将该配置文件加入到nginx的启动配置文件中  
sudo ln -s /home/git/Mx_Online/MxOnline/uc_nginx.conf /etc/nginx/conf.d/  

拉取所有需要的static file 到同一个目录  
在django的setting文件中，添加下面一行内容：  
```py
    STATIC_ROOT = os.path.join(BASE_DIR, "static/")  
```
我这里会报错，所以改为  
但是这样执行collectstatic后所有的静态文件会重新放入MxOnline/static1这个文件里面  
```py
    STATIC_ROOT = os.path.join(BASE_DIR, "static1/")  
```
在django的urls.py中添加：
```py
    #静态文件
    re_path(r'^static/(?P<path>.*)', serve, {"document_root": STATIC_ROOT}),
```
运行命令  
    python manage.py collectstatic  

这里需要注意 一定是直接用nginx命令启动， 不要用systemctl启动nginx不然会有权限问题  
运行nginx  
sudo /usr/sbin/nginx  


## 9. 运行Django项目
```py
cd 项目名
python3 manage.py runserver 0.0.0.0:8000

```
访问：  
```py
http://ip:8000/
```







