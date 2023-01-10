### linux下安装java环境

#### 第一步：下载jdk

```
官网地址：https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
```



#### 第二步：创建jdk安装文件夹

安装到/usr/local/java下
执行一下命令

```
// 进入到/usr/local
cd /usr/local  	
// 创建java文件夹
mkdir java		
// 查看当前目录下文件
ls			
//进入java文件夹目录下	
cd /java		
```

#### 第三步：将本地下载好的JDK压缩包上传到服务器并解压


解压命令：tar -zxvf 上传的jdk压缩包名

#### 第四步：配置环境变量

打开配置文件 vim /etc/profile
在文件末尾追加

```
JAVA_HOME=/usr/local/java/jdk文件名
CLASSPATH=%JAVA_HOME%/lib:%JAVA_HOME%/jre/lib
PATH=$PATH:$JAVA_HOME/bin:$JAVA_HOME/jre/bin
export PATH CLASSPATH JAVA_HOME
```


注意：/usr/local/java/jdk文件名中的jdk文件名需要改为自己准确的文件名称哦

然后按esc退出编辑模式，按shift+:输入wq保存并退出

#### 第五步：使配置文件生效

执行命令：source /etc/profile

##### 第六步：检查是否安装成功

执行命令：java -version

