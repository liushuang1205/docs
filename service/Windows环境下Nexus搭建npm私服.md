### Windows环境下 Nexus搭建npm私服

**第一步，下载nexus**

下载链接： https://pan.baidu.com/s/1GOYi2M3nT4Wcy7JEYmnqdA 提取码: a9hf



**第二步，解压缩**

我下载的是nexus-3.16.1-02-win64.zip这个版本，解压缩后，进入\nexus-3.16.1-02\bin

![](_media/ad0f8390f9594632b969188b702f3cbf.png)



再次目录下，用cmd命令行执行：`nexus.exe /run`

启动之后默认地址为 localhost:8081, 默认账号密码 admin/ admin123

![img](_media/ff036c9e7443423dbce78f79ff138a22.png)

------

```
第三步，创建存储空间（如果使用默认的存储空间，此步骤可省略）
```

![img](_media/09464770a9984dc398754fc4d2748ada.jpg)

```
输入空间的名称，点击create创建
```

![img](_media/874827b7913b4dafa145dbcaf9a9d76d.jpg)

------

 

```
第四步，创建仓库
```

![img](_media/dd13e85e559243529b0318924080b42a.jpg)

```
npm的仓库有三种：
```

![img](_media/fbf91666c22e434a94719bb8b853a78f.jpg)

```
这三种分别是：
hosted（私有仓库）：用于发布个人开发的npm组件
proxy（代理仓库）：可以代理npm和淘宝镜像
group（组合仓库）：对外公开的仓库，集合了hosted和proxy
```

------

1.创建 Hosted npm Registries

![img](_media/3f7e4f448ac048daaa353db98531de4c.png)

输入仓库名称，存储空间选择刚刚创建的，也可以用默认的。点击create创建。

2.创建 Proxy npm Registries

![img](_media/f499ba4497d94b7781c81e71ee9c455e.png)

输入仓库名称，选择存储空间，代理的远程地址可以写https://registry.npmjs.org，也可以写淘宝镜像。点击create创建。

3.创建 Group npm Registries

![img](_media/2af079ab02084630bf16cea3d0913da7.png)

组合仓库中除了输入名称和选择存储空间之外，还要选择要包括的仓库，越靠上优先级越高，如果私有仓库在上，用户下载npm包的时候会优先下载私有仓库中的，如果私有仓库没有再去代理仓库中下载。

------

**第五步，创建用户，设置权限（用于发布npm包）**

![img](_media/80646f567f8e479aaea5506630daf79a.png)

然后是设置权限，这一步如果不设置，是不能发布自己的npm包的。

![img](_media/11d3492578c740f29b334f653c2e0633.png)

------

**第六步，用户端使用私服**

到这里，nexus的设置都好了，但是用户如何使用私服下载npm和上传npm呢？

**1.用户端设置npm的registry为group仓库**

首先复制出group仓库的链接地址

![img](_media/bb53b6723d1842f6bc1db2af73c1f21d.png)

然后，用户端设置registry。

方法一：

命令行执行：

```
npm config set registry http://npm私服所在服务器的ip地址:8081/repository/npm-group/
```

 

方法二：

修改C:\Users\Administrator下的.npmrc文件，修改为：

registry=http://npm私服所在服务器的ip地址:8081/repository/npm-group/

两种方法都可以，修改后，就可以正常使用npm下载了。

**2.用户端发布自己的npm包到私服（执行的命令均在发布的模块根目录下）**

方法一：

首先，登陆私服：

命令行执行：

```
npm login –registry=http://npm私服所在服务器的ip地址:8081/repository/npm-hosted/
```

 

```
这时候需要输入nexus的用户名、密码和邮箱。
```

 

然后，就可以发布了，要发布的模块，必须保证在根目录下有package.json文件，否则会报错。

命令行执行：

```
npm publish –registry=http://npm私服所在服务器的ip地址:8081/repository/npm-hosted/
```



方法二：
在package.json 中添加

```javascript
"publishConfig": {
    "registry": "仓库地址"
 },
```

即：![](_media/4c72980e3dd4461f8dceee6ae87b0123.png)
然后执行：

```javascript
npm publish
```



到此，可以到nexus验证一下有没有发布成功

![img](_media/6ff4e51f021d4576b174cdd2f9e6b637.png)

![img](_media/e54d455c61494326bf753e71d24b4a01.png)