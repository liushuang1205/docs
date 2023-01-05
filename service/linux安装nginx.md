##### 1.安装依赖包

```
//一键安装上面四个依赖
yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel
```

##### 2.下载并解压安装包

```
//创建一个文件夹
cd /usr/local
mkdir nginx
cd nginx
//下载tar包
wget http://nginx.org/download/nginx-1.20.2.tar.gz
tar -xvf nginx-1.20.2.tar.gz
```

##### 3.安装nginx

```
//进入nginx目录
cd /usr/local/nginx
//进入目录
cd nginx-1.13.7
//执行命令 考虑到后续安装ssl证书 添加两个模块
./configure --with-http_stub_status_module --with-http_ssl_module
//执行make命令
make
//执行make install命令
make install
```

##### 4.启动nginx服务

```
 /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
```

```
// 配置nginx.conf，编辑配置文件
vi /usr/local/nginx/conf/nginx.conf
```


将端口号改成8089(随便挑个端口)，因为可能apeache占用80端口，apeache端口尽量不要修改，我们选择修改nginx端口。

将localhost修改为你服务器的公网ip地址。



##### 5.重启nginx

```
/usr/local/nginx/sbin/nginx -s reload
```


查看nginx进程是否启动：

```
ps -ef | grep nginx
```



##### 6.若想使用外部主机访问nginx，需要关闭服务器防火墙或开放nginx服务端口，端口为上一步nginx.conf的配置端口：

```
centOS6及以前版本使用命令： systemctl stop iptables.service

centOS7关闭防火墙命令： systemctl stop firewalld.service
```

关闭防火墙会导致服务器有一定风险，所以建议是单独开放服务端口 ：

开放80端口：

```
firewall-cmd --zone=public --add-port=80/tcp --permanent
```

查询端口号80 是否开启：

```
firewall-cmd --query-port=80/tcp
```

重启防火墙：

```
firewall-cmd --reload
```


随后访问该ip:端口 即可看到nginx界面。

##### 7.安装完成一般常用命令

进入安装目录中，

```
命令： cd /usr/local/nginx/sbin

启动，关闭，重启，命令：

./nginx 启动

./nginx -s stop 关闭

./nginx -s reload 重启
```


