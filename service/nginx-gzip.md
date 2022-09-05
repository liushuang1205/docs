## Nginx 开启 Gzip 压缩

​ Nginx 支持静态和动态两种包体 gzip 压缩方式,分别对应模块 ngx_http_gzip_static，ngx_http_gzip。nginx 实现资源压缩的原理是通过 ngx_http_gzip_module 模块拦截请求，并对需要做 gzip 的类型做 gzip 压缩，

### 动态：

```nginx
# 开启gzip  on/off
gzip off;

# 启用gzip压缩的最小文件，小于设置值的文件将不会压缩
gzip_min_length 1k;

# gzip 压缩级别，1-9，数字越大压缩的越好，也越占用CPU时间，后面会有详细说明
gzip_comp_level 1;

# 进行压缩的文件类型。javascript有多种形式。其中的值可以在 mime.types 文件中找到。
gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png application/vnd.ms-fontobject font/ttf font/opentype font/x-woff image/svg+xml;

# 是否在http header中添加Vary: Accept-Encoding，建议开启，对于支持gzip的请求反向代理缓存服务器将返回gzip内容，不支持gzip的客户端返回原始内容。
gzip_vary on;

# 禁用IE 6 gzip
gzip_disable "MSIE [1-6]\.";

# 设置压缩所需要的缓冲区大小
gzip_buffers 32 4k;

# 设置gzip压缩针对的HTTP协议版本
gzip_http_version 1.1;

# 设置是否对服务端返回的结果进行Gzip压缩
gzip_proxied any;


http{
 ....
     gzip on ;
     gzip_min_length: 1k;
     gzip_comp_level 6;
     gzip_types text/xml text/plain text/css application/javascript application/x-javascript application/rss+xml image/jpegimage/gif image/png;
 ...
 }

```

### 静态：

​ gzip 是 CPU 密集型的应用，实时动态压缩比较消耗 CPU 资源。另外，如果使用 gzip，则 sendfile 零拷贝技术无法使用。为进一步提高 Nginx 的性能，我们可以使用静态 gzip 压缩，提前将需要压缩的文件压缩好，当客服请求到达时，直接发送压缩好的.gz 文件，如此就减轻了服务器 CPU 的压力，提高了性能。缺省 ngx_http_gzip_static 模块并未启用，需要重新编译。

```nginx
http{
 ....
    gzip_static on;
    gzip_http_version 1.1;
    gzip_proxied no-cache no-store private auth;
 ...
 }
```

​ **gzip_static on;** 添加后，会报一个错误，`unknown directive "gzip_static"`主要的原因是 Nginx 默认是没有添 ngx_http_gzip_static_module 模块;添加 gzip_static 模块即可。

```
#注:根据需要自行添加其它参数
./configure --with-http_gzip_static_module
```

#### 添加 ngx_http_gzip_static_module 模块

- 查询 Nginx 配置参数

```markdown
$ nginx -V
```

- nginx 二进制文件更名

```markrdown
# 将 nginx 安装目录下 sbin 目录中的 nginx 二进制文件进行更名
# nginx 默认安装路径为 usr/local 若指定过安装目录,则进入对应的nginx目录
$ cd /usr/local/nginx/sbin

$ mv nginx nginxold
```

- 进入 Nginx 压缩包目录

```markdown
$ pwd
/opt/nginx/core/nginx-1.16.1

$ ls -l
drwxr-xr-x. 6 1001 1001 4096 May 31 23:21 auto
-rw-r--r--. 1 1001 1001 296463 Aug 13 2019 CHANGES
-rw-r--r--. 1 1001 1001 452171 Aug 13 2019 CHANGES.ru
drwxr-xr-x. 2 1001 1001 168 May 31 23:21 conf
```

- make clean

执行 make clean 清空之前编译的内容

```markdown
./configure --prefix=/usr/local/nginx --sbin-path=/usr/local/nginx/sbin/nginx \
--modules-path=/usr/local/nginx/modules --conf-path=/usr/local/nginx/conf/nginx.conf \
--error-log-path=/usr/local/nginx/logs/error.log \
--http-log-path=/usr/local/nginx/logs/access.log \
--pid-path=/usr/local/nginx/logs/nginx.pid --lock-path=/usr/local/nginx/logs/nginx.lock \
--with-http_gzip_static_module
```

- make

使用 make 命令进行编译

```markdown
$ make
```

- 移动 nginx 二进制文件

将 objs 目录下的 nginx 二进制执行文件移动到 nginx 安装目录下的 sbin 目录中

```markdown
# /usr/local/nginx/sbin 为 nginx 安装目录 若指定其它安装目录,修改成对应的即可

$ mv objs/nginx /usr/local/nginx/sbin/
```

- 执行更新命令

```markdown
$ make upgrade
```

### 不需要使用压缩的场景：

1. 图片/mp3 这样的二进制文件,不必压缩,因为压缩率比较小, 比如 100->80 字节,而且压缩也是耗费 CPU 资源的.
2. 比较小的文件不必压缩,

### 其他说明

- gzip_static 配置优先级高于 gzip
- 开启 nginx_static 后，对于任何文件都会先查找是否有对应的 gz 文件
- gzip_types 设置对 gzip_static 无效

### 注意：

1. 其中的 gzip_http_version 的设置，它的默认值是 1.1，就是说对 HTTP/1.1 协议的请求才会进行 gzip 压缩
   如果我们使用了 proxy_pass 进行反向代理，那么 nginx 和后端的 upstream server 之间是用 HTTP/1.0 协议通信的。
   如果我们使用 nginx 通过反向代理做 Cache Server，而且前端的 nginx 没有开启 gzip
   同时，我们后端的 nginx 上没有设置 gzip_http_version 为 1.0，那么 Cache 的 url 将不会进行 gzip 压缩

2. gzip_disable 的设置是禁用 IE6 的 gzip 压缩，又是因为杯具的 IE6
   IE6 的某些版本对 gzip 的压缩支持很不好，会造成页面的假死，今天产品的同学就测试出了这个问题
   后来调试后，发现是对 img 进行 gzip 后造成 IE6 的假死，把对 img 的 gzip 压缩去掉后就正常了
   为了确保其它的 IE6 版本不出问题，所以就加上了 gzip_disable 的设置

## 指令

**gzip**: 该指令用于开启或者关闭 gzip 功能

| 语法   | gzip on\|off;             |
| ------ | ------------------------- |
| 默认值 | gzip off;                 |
| 位置   | http、server、location... |

注意只有该指令为打开状态，下面的指令才有效果

```csharp
http{
   gzip on;
}
```

#### gzip_types 指令

**gzip_types**: 该指令可以根据响应页的 MIME 类型选择性地开启 Gzip 压缩功能

| 语法   | gzip_types mime-type ...; |
| ------ | ------------------------- |
| 默认值 | gzip_types text/html;     |
| 位置   | http、server、location    |

所选择的值可以从 mime.types 文件中进行查找，也可以使用"\*"代表所有。

```bash
http{
	gzip_types application/javascript;
}
```

#### gzip_comp_level 指令

**gzip_comp_level** : 该指令用于设置 Gzip 压缩程度，级别从 1-9; 1 表示压缩程度最低，效率最高，9 刚好相反，压缩程度最高，但是效率最低最费时间。

| 语法   | gzip_comp_level level; |
| ------ | ---------------------- |
| 默认值 | gzip_comp_level 1;     |
| 位置   | http、server、location |

```bash
http{
    # 建议设置到中间即可  推荐6
	gzip_comp_level 6;
}
```

#### gzip_vary 指令

**gzip_vary**: 该指令用于设置使用 Gzip 进行压缩发送是否携带 **"Vary:Accept-Encoding"** 头域的响应头部。主要是告诉接收方，所发送的数据经过了 Gzip 压缩处理。

| 语法   | gzip_vary on\|off;     |
| ------ | ---------------------- |
| 默认值 | gzip_vary off;         |
| 位置   | http、server、location |

#### gzip_buffers 指令

**gzip_buffers**: 该指令用于处理请求压缩的缓冲区数量和大小。

| 语法   | gzip_buffers number size;  |
| ------ | -------------------------- |
| 默认值 | gzip_buffers 32 4k\|16 8k; |
| 位置   | http、server、location     |

其中 number:指定 Nginx 服务器向系统申请缓存空间个数，size 指的是每个缓存空间的大小。主要实现的是申请 number 个每个大小为 size 的内存空间。这个值的设定一般会和服务器的操作系统有关，所以建议此项不设置，使用默认值即可。

```bash
gzip_buffers 4 16K;	  #缓存空间大小
```

#### gzip_disable 指令

**gzip_disable**: 针对不同种类客户端发起的请求，可以选择性地开启和关闭 Gzip 功能。

| 语法   | gzip_disable regex ...; |
| ------ | ----------------------- |
| 默认值 | —                       |
| 位置   | http、server、location  |

regex:根据客户端的浏览器标志(user-agent)来设置，支持使用正则表达式。指定的浏览器标志不使用 Gzip.该指令一般是用来排除一些明显不支持 Gzip 的浏览器。

```bash
gzip_disable "MSIE [1-6]\.";
```

#### gzip_http_version 指令

**gzip_http_version**: 针对不同的 HTTP 协议版本，可以选择性地开启和关闭 Gzip 功能。

| 语法   | gzip_http_version 1.0\|1.1; |
| ------ | --------------------------- |
| 默认值 | gzip_http_version 1.1;      |
| 位置   | http、server、location      |

该指令是指定使用 Gzip 的 HTTP 最低版本，该指令一般采用默认值即可。

#### gzip_min_length 指令

**gzip_min_length**: 该指令针对传输数据的大小，可以选择性地开启和关闭 Gzip 功能

| 语法   | gzip_min_length length; |
| ------ | ----------------------- |
| 默认值 | gzip_min_length 20;     |
| 位置   | http、server、location  |

```css
nignx计量大小的单位：bytes[字节] / kb[千字节] / M[兆]
例如: 1024 / 10k|K / 10m|M
```

Gzip 压缩功能对大数据的压缩效果明显，但是如果要压缩的数据比较小的化，可能出现越压缩数据量越大的情况，因此我们需要根据响应内容的大小来决定是否使用 Gzip 功能，响应页面的大小可以通过头信息中的`Content-Length`来获取。但是如何使用了 Chunk 编码动态压缩，该指令将被忽略。建议设置为 1K 或以上。

#### gzip_proxied 指令

**gzip_proxied**: 该指令设置是否对服务端返回的结果进行 Gzip 压缩。

| 语法   | gzip_proxied off\|expired\|no-cache\| no-store\|private\|no_last_modified\|no_etag\|auth\|any; |
| ------ | ---------------------------------------------------------------------------------------------- |
| 默认值 | gzip_proxied off;                                                                              |
| 位置   | http、server、location                                                                         |

```markdown
off - 关闭 Nginx 服务器对后台服务器返回结果的 Gzip 压缩

expired - 启用压缩，如果 header 头中包含 "Expires" 头信息

no-cache - 启用压缩，如果 header 头中包含 "Cache-Control:no-cache" 头信息

no-store - 启用压缩，如果 header 头中包含 "Cache-Control:no-store" 头信息

private - 启用压缩，如果 header 头中包含 "Cache-Control:private" 头信息

no_last_modified - 启用压缩,如果 header 头中不包含 "Last-Modified" 头信息

no_etag - 启用压缩 ,如果 header 头中不包含 "ETag" 头信息

auth - 启用压缩 , 如果 header 头中包含 "Authorization" 头信息

any - 无条件启用压缩
```

#### gzip_static 指令

**gzip_static**: 检查与访问资源同名的.gz 文件时，response 中以 gzip 相关的 header 返回.gz 文件的内容。

| 语法   | **gzip_static** on \| off \| always; |
| ------ | ------------------------------------ |
| 默认值 | gzip_static off;                     |
| 位置   | http、server、location               |

> nginx.conf

```markdown
http{
gzip_static on;
}
```
