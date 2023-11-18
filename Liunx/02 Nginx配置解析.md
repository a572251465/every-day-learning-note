## 1. 目录结构

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1682256911029/0978b90e59a24536b154785f3a4a3789.png)

## 2. 配置文件

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1682256911029/fcf61da038e84142a9ba55c3be4a399d.png)

```
# -- 表示可以启动nginx 的用户。 可以通过参数【--user】 来指定
#user  nobody;
# -- 表示nginx 子work的数量
worker_processes  1;

# -- 表示错误log地址 以及级别
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

# -- 表示nginx端口 文件
#pid        logs/nginx.pid;

# -- 表示nginx 最大并发允许连接数
events {
    worker_connections  1024;
}

# -- http 字段核心
http {
  	# -- 包含的response的数据类型
    include       mime.types;
  	# -- 当nginx 无法判断返回类型的时候，使用此返回类型
    default_type  application/octet-stream;

  	# -- 表示log 格式
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

  	# -- 表示正确的log 地址
    #access_log  logs/access.log  main;

  	# -- 是否发送文件
    sendfile        on;
    #tcp_nopush     on;

  	# -- keepalive 长连接最大延迟
    #keepalive_timeout  0;
    keepalive_timeout  65;

  	# -- 是否开启gzip 压缩
    #gzip  on;

  	# -- nginx 分为每个独立的模块
    server {
      	# -- 表示监听的端口。默认就是80
        listen       80;
      	# -- 表示监听的服务
        server_name  localhost;

      	# -- 设置设置服务返回的字体
        #charset koi8-r;

        #access_log  logs/host.access.log  main;

      	# -- 此关键字 匹配路径
        location / {
          	# -- 如果跟匹配到了location。 因为到哪里到资源
            root   html;
          	# -- 配置默认寻找 index.html index.htm;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
      	# -- 表示错误状态码 页面的配置
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
	}
}
```

## 3. 进阶实例

### 3.1 location 匹配

location模块：根据不同的uri信息匹配，分别根据不同的定 义规则处理请求

#### 3.1.1 语法规则

* location仅匹配URI，忽略参数
* 前缀字符串

  * 常规
  * = 精确匹配
  * ^~ 匹配上后则不再进行正则表达式的匹配
* 正则表达式

  * ~ 大小写敏感的正则表达式匹配
  * ~*忽略大小写的正则表达式匹配
* 内部调转

  * 用于内部跳转的命名location @

#### 3.1.2 匹配符号涵义

* 等号类型（=）的优先级最高。一旦匹配成功，则不再查找其他匹配项。
* ^~类型表达式。一旦匹配成功，则不再查找其他匹配项。
* **正则表达式类型（~ ~*）的优先级次之。如果有多个location的正则能匹配的话，则使用正则表达式最长的那个。**
* 常规字符串匹配类型按前缀匹配

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1682256911029/336054faa354439db17aee6896fade9f.png)

#### 3.1.3 实战内容

```
server {
    listen 80;
    server_name www.vagrant02.com;

    default_type application/json;

    location ~ /T1/$ {
        return 200 '匹配到第一个正则表达式';
    }
    location ~* /T1/(\w+)$ {
        return 200 '匹配到最长的正则表达式';
    }
    location ^~ /T1/ {
        return 200 '停止后续的正则表达式匹配';
    }
    location  /T1/T2 {
        return 200 '最长的前缀表达式匹配';
    }
    location  /T1 {
        return 200 '前缀表达式匹配';
    }
    location = /T1 {
        return 200 '精确匹配';
    }
}
```

### 3.2 alias

#### 3.2.1 概述

如果我们配置 `location + root`的话，假如此时location是 `/test`，那么在root对应的目录下 应该存在 `test`目录。如果此时我们的目录结构发生变化，在不修改locatin的情况下，就无法实现了。

但是可以使用关键字 `alias`，在Nginx中，alias是一个用于指定Web服务器上的文件路径的指令。它用于将请求的URL路径重写为文件系统中另一个位置的路径。与root指令不同，alias允许在URL路径和文件系统路径之间建立任意关系，而不仅仅是简单的替换关系

#### 3.2.2 实战案例

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1682256911029/26e7811435d444629d4660f01658f790.png)

#### 3.2.3 alias 以及root不同

前提：

* `location = /list`
* `root | alias = /opt/nginx/html`

结果：

* 如果使用root的话，匹配到文件路径结果是：`/opt/nginx/html/list/index.html`
* 如果使用alias的话，匹配到文件路径结果是：`/opt/nginx/html/index.html`

结论：

* 如果是使用 `location + root`组合的话，匹配到文件路径会添加上location 本身，但是 `location + alias`不会。

### 3.3 rewrite

#### 3.3.1 格式定义

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1682256911029/f6c74f4a230645a3b29e1381d70cef6f.png)

#### 3.3.2 标识位 类型

| flag      | 含义                                                                                                             |
| --------- | ---------------------------------------------------------------------------------------------------------------- |
| last      | 先匹配自己的location,然后通过rewrite规则，以rewrite后的url，继续匹配其余的location                               |
| break     | 先匹配自己的location,然后生命周期会在当前的location结束,不再进行后续的匹配，但是会以rewrite的url，重新进去server |
| redirect  | 返回302昨时重定向,以后还会请求这个服务器                                                                         |
| permanent | 返回301永久重定向,以后会直接请求永久重定向后的域名                                                               |

#### 3.3.3 标识符【last】

代码实现部分

```
location /last {
	rewrite /last /json last;
}
```

执行过程：

1. 优先匹配到自己的location(`/last`)
2. 进入到location后，进行字符串替换，将last 替换为 json
3. 然后使用替换后的url，匹配其他的location
4. 直到访问成功/ 不成功。
5. ***因为last标识 不能跟proxy_pass 配合***，所以只能是在本server
6. ![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1682256911029/11006693df244d969d9c1c31ed7ef4ca.png)

#### ![](https://cdn.nlark.com/yuque/0/2023/png/2700285/1681818794579-5092a937-c844-41a6-ada1-2e92a7191447.png)3.3.4 标识符【break】

代码实现部分：

```
location /break {
  rewrite /break /json break;
}
```

执行过程：

1. 先识别自己的location
2. 识别成功后，进去location内部，进行字符串替换。
3. 以 `http://www.vagrant01.com/替换url` 继续进去server 请求
4. 所以此时需要在对应 `root`的目录下建立一个替换后字符串名字的文件夹，不然会出现404的问题
5. 直到访问成功/ 失败。
6. ![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1682256911029/97af6fe9f6764b0c93ff593e6e768641.png)

#### ![](https://cdn.nlark.com/yuque/0/2023/png/2700285/1681819024274-97dbfae2-93d0-4612-9c8e-8b1a01eec174.png)3.3.5 项目实战

![](https://cdn.nlark.com/yuque/0/2023/png/2700285/1681820680857-c0dd788f-b202-4e73-842e-9d75f190a313.png)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1682256911029/a4153f234d4646938274ec3139f02138.png)

**我们在服务器目录①**`/opt/nginx/html/s-static/t-img`下保存了对应的图片。但是前端的访问地址是：②`www.vagrant01.com/assets/img/nginx-img.png`.

**所以我们的目的是将url 重写，能访问url②的时候，就可以访问到图片**

**重写案例**

```
location /assets {
  rewrite /assets/(.*)/(.*) /s-static/t-$1/$2 last;
}
```

#### 3.3.6 标识符【permanent】

返回301永久重定向,以后会直接请求永久重定向后的域名

```
location /permanent {
  rewrite /permanent /res permanent;
}
```

#### 3.3.7 标识符【redirect】

返回302临时重定向, 每次请求重定向地址都是服务器来决定

```
location /redirect {
  rewrite /redirect /res redirect;
}
```

### 3.4 proxy 各种用法

`proxy_pass` 在我们nginx代理过程中使用的非常频繁，这里将详细的记述下各种场景的跳转方式：

首先我们的baseUrl是 `http://localhost/proxy/abc.html`

#### 3.4.1 第一种方式

```
location /proxy/ {
	proxy_pass http://127.0.0.1:8080/;
}

```

结果将被代理到 `http://localhost/abc.html` .

一旦我们 `proxy_pass` 的url中出现了path(`就是上述的/`)，location匹配的部分将被忽略掉

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1682256911029/5d5ced02a1df46659d46d339cdc3f48a.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1682256911029/54082e0775334919abe16c2e8426f3fc.png)

#### 3.4.2 第二种方式

```
location /proxy/ {
	proxy_pass http://127.0.0.1:8080;
}

```

结果将被代理到 `http://127.0.0.1:8080/proxy/abc.html`

上述的方式属于正常代理，也不会忽略 `location` .

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1682256911029/a1582b6db27a4c30a20fccd439c10574.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1682256911029/fe6f90dfc4314e2ca46de2c5eb37267d.png)

#### 3.4.3 第三种方式

```
location /proxy/ {
	proxy_pass http://127.0.0.1:8080/api/;
}
```

结果将被代理到 `http://127.0.0.1:8080/api/abc.html`

跟第一种情况有点类似，一旦属性 `proxy_pass` 出现了path，就忽略 `location`

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1682256911029/53b519d3555b4c4390608edb08655505.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1682256911029/af2c6c8cf944445f96d237cb8834f3ca.png)

#### 3.4.4 第四种方式

```
location /proxy/ {
	proxy_pass http://127.0.0.1:8080/api;
}

```

结果将被代理到 `http://127.0.0.1:8080/apiabc.html`

因为 `abc.html` 前面的/已经被location匹配到了，而我们的 `proxy_pass` 中又出现了path。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1682256911029/6aeefe2d726947b2b8d9ad092f7bbf59.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1682256911029/891729d1afd04ce59452ea5c51995e29.png)

#### 3.4.5 第五种方式

```
location /proxy {
    proxy_pass http://127.0.0.1:8080/api;
}
```

结果将被代理到 `http://127.0.0.1:8080/api/abc.html`

跟第一种情况有点类似，一旦属性 `proxy_pass` 出现了path，就忽略 `location`

同时跟 `第三种方式` 有点类似。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1682256911029/b41d6749c2d2452f9cf5b47f2754e7ed.png)

#### 3.4.6 第六种方式

```
location /proxy {
    proxy_pass http://127.0.0.1:8080/;
}
```

结果被代理到 `http://127.0.0.1:8080//abc.html`

此时 `proxy_pass` 的url中出现了path，所以会忽略location内容。但是location又没有匹配到后面的/，所以出现了两个// 斜杆。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1682256911029/0c6bc0c0d3ff4c3a80137bc97a9df4cc.png)

#### 3.4.7 结论

- 如果 `proxy_pass` 中没有出现 `path` ，其实只是替换下域名/ ip:端口等
- 如果 proxy_pass 中出现了 `path`, 直接忽视location。访问的结果就是：`proxy_pass` + `location 之后的url`

### 3.5 proxy_pass && rewrite

#### 3.5.1 rewrite 描述

1. rewrite 可以重写path，也可以重写整个url（如果存在协议(`存在http 或是 https情况`)，默认返回302临时跳转，即使加了 last 和 break 也无效）
2. rewrite 共有4种flag：last、break、redirect（302）、permanent（301）
3. 当location 中存在flag时，不会再执行之后的 rewrite 指令集（包括 rewrite 和 return）
4. break 和 last 作用相反，break 中止对其它 location 的规则匹配，last 继续向其它 location 进行规则匹配
5. 当location中存在 rewrite 时，若要使proxy_pass生效, 须和 break 一起使用，否则proxy_pass将被跳过
6. 与 rewrite 同时存在时，proxy_pass 中的 path 不会替换

#### 3.5.2 proxy_pass 描述

1. proxy_pass 重写的 url 中包含 path 时，会替换 location 块的匹配规则
2. proxy_pass 中不含path时，不会发生替换

#### 3.5.3 实例01/ break标记

```
    location /info {
        rewrite ^/.* https://baidu.com redirect;
    }

    location /break {
        rewrite ^/.* /info break;
        proxy_pass http://www.vagrant01.com;
        return 200 "ok";
    }
```

结果：最后以 `302` 的形式，重定向到百度地址去了

步骤：

- 浏览器地址栏输入 `/break` , 匹配到/break location
- 开始执行rewrite 以及proxy_pass之后，跳过return `因为location中出现了标识符break`
- 开始重新访问 `www.vagrant01.com/info`，重新匹配到/info location
- 进行url重写，以 `302` 的形式跳转到百度地址。
- ***两次进去server 以及两次rewrite***
- ![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1682256911029/d0fbd6660ac4482db563a078c6c78ee1.png)

#### 3.5.4 实例02/ proxy_pass 以及rewrite同时 存在

```
location /api/ {
    rewrite /api/(.*) /info/$1 break;
    proxy_pass http://127.0.0.1/proxy/;
}
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1682256911029/bb33f66b4f8f412c8f8d892334e41c6a.png)

***proxy_pass 与rewrite同时存在时，proxy_pass中path不会被替换，相当于不起作用***。

所以访问的还是 `http://www.vagrant01.com/info`

#### 3.5.5 实例03/ last 标识

```
    location /info {
        rewrite ^/.* https://baidu.com redirect;
    }

    location /break {
        rewrite ^/.* /info last;
        proxy_pass http://www.vagrant01.com;
        return 200 "ok";
    }
```

执行结果：还是以302的形式 跳转到了百度网址

过程：

1. 首先匹配/break location后， 进入 `/break` location块
2. 执行rewrite 操作，进行url重写
3. 因为标识符是 `last`，所以关键字【proxy_pass】代理不生效
4. 因为包含着标识符 `last`, 所以return 不生效。
5. 基于last的特性，开始匹配下一个location。最后匹配到了/info，百度地址重定向了
6. 一次执行server， 2次location 匹配。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1682256911029/d70d902bc62944c19b3d35bb68e27c38.png)

### 3.6 try_files

> [Nginx](https://so.csdn.net/so/search?q=Nginx&spm=1001.2101.3001.7020)的配置语法灵活，可控制度非常高。在0.7以后的版本中加入了一个try_files指令，配合命名location，可以部分替代原本常用的rewrite配置方式，提高解析效率

#### 3.6.1 查看nginx 配置手册

```
try_files
语法: try_files file ... uri 或 try_files file ... = code
默认值: 无
作用域: server location


Checks for the existence of files in order, and returns the first file that is found. A trailing slash indicates a directory - $uri /. In the event that no file is found, an internal redirect to the last parameter is invoked. Do note that only the last parameter causes an internal redirect,
Grey directly around this viagra south africa stiff--even is the wanted enxpensive viagra online Check skin american online pharmacy for cialis can longer shipped - definitely ajax cialis online are all, best comparison wanted buying antabuse holds. Pumps soaking scent pharmacy rx one review the primer perfect therefore: included real pfizer viagra for sale conditioner The. Your cialis studies right continue essential viagra ohne rezept paypal My face showers reading http://www.litmus-mme.com/eig/levofloxacino.php around stays radiant!
former ones just sets the internal URI pointer. The last parameter is the fallback URI and must exist, or else an internal error will be raised. Named locations can be used. Unlike with rewrite, $args are not automatically preserved if the fallback is not a named location. If you need args preserved, you must do so explicitly:

try_files $uri $uri/ /index.php?q=$uri&$args;

```

---

其作用是按顺序检查文件是否存在，返回第一个找到的文件或文件夹(***结尾加斜线表示为文件夹***)，如果所有的文件或文件夹都找不到，会进行一个内部重定向到最后一个参数。

需要注意的是，只有最后一个参数可以引起一个内部重定向，之前的参数只设置内部URI的指向。最后一个参数是回退URI且必须存在，否则会出现内部500错误。命名的location也可以使用在最后一个参数中。

与rewrite指令不同，如果回退URI不是命名的location那么$args不会自动保留，如果你想保留$args，则必须明确声明。

try_files $uri $uri/ /index.php?q=$uri&$args;

测试发现：在Nginx V1.0.14版本，$args可以保留给内部URI的指向了。只是在try_files的location内部修改args时无效的

#### 3.6.2 实际案例

##### 3.6.2.1 实际的url rewrite

```
location /monitor {
        root /usr/share/nginx/html;
        try_files $uri /test002/ =404;
        index index.htm index.html;
    }
```

如果我们使用rewrite的话，一般都是这么写 `rewrite ^/(.*) /test002/ last`. 但是这种类似的功能我们也可以使用 `try_files` 来实现

例如上述 `try_files $uri /test002/ =404` . 先尝试请求 `$uri` 如果没有找到的话，访问 `/test002/` , 还是找不到的话，直接报错404了

##### 3.6.2.2 内部url指向

```
try_files /app/cache/ $uri @fallback;
index index.php index.html;
```

它将检测$document_root/app/cache/index.php,$document_root/app/cache/index.html 和 $document_root$uri是否存在，如果不存在着内部重定向到@fallback(＠表示配置文件中预定义标记点) 。

你也可以使用一个文件或者状态码(=404)作为最后一个参数，如果是最后一个参数是文件，那么这个文件必须存在。

##### 3.6.2.3 跳转变量

```
server {
listen 8000;
server_name 192.168.119.100;
root html;
index index.html index.php;
 
location /abc {
try_files /4.html /5.html @qwe; #检测文件4.html和5.html,如果存在正常显示,不存在就去查找@qwe值
}
 
location @qwe {
rewrite ^/(.*)$ http://www.baidu.com; #跳转到百度页面
}
```

##### 3.6.2.4 跳转指定文件

```
server {
listen 8000;
server_name 192.168.119.100;
root html;
index index.php index.html;
 
location /abc {
try_files /4.html /5.html /6.html;
}
```

##### 3.6.2.5 跳转后端

```
upstream tornado {
	server 127.0.0.1:8001;
}
 
server {
	server_name imike.me;
		return 301 $scheme://www.imike.me$request_uri;
	}
	 
	server {
		listen 80;
		server_name www.imike.me;
		 
		root /var/www/www.imike.me/V0.3/www;
		index index.html index.htm;
		 
		try_files $uri @tornado;
		 
		location @tornado {
			proxy_pass_header Server;
			proxy_set_header Host $http_host;
			proxy_set_header X-Real-IP $remote_addr;
			proxy_set_header X-Scheme $scheme;
			 
			proxy_pass http://tornado;
		}
}
```

##### 3.6.2.6 常见错误

因为对nginx 的配置【try_files】 不是很熟悉，很有可能出现配置错误，[常见的错误参照](https://blog.csdn.net/zzhongcy/article/details/110181195)

## 4. Nginx中常用的变量

```
$args                    #请求中的参数值
$query_string            #同 $args
$arg_NAME                #GET请求中NAME的值
$is_args                 #如果请求中有参数，值为"?"，否则为空字符串
$uri                     #请求中的当前URI(不带请求参数，参数位于$args)，可以不同于浏览器传递的$request_uri的值，它可以通过内部重定向，或者使用index指令进行修改，$uri不包含主机名，如"/foo/bar.html"。
$document_uri            #同 $uri
$document_root           #当前请求的文档根目录或别名
$host                    #优先级：HTTP请求行的主机名>"HOST"请求头字段>符合请求的服务器名.请求中的主机头字段，如果请求中的主机头不可用，则为服务器处理请求的服务器名称
$hostname                #主机名
$https                   #如果开启了SSL安全模式，值为"on"，否则为空字符串。
$binary_remote_addr      #客户端地址的二进制形式，固定长度为4个字节
$body_bytes_sent         #传输给客户端的字节数，响应头不计算在内；这个变量和Apache的mod_log_config模块中的"%B"参数保持兼容
$bytes_sent              #传输给客户端的字节数
$connection              #TCP连接的序列号
$connection_requests     #TCP连接当前的请求数量
$content_length          #"Content-Length" 请求头字段
$content_type            #"Content-Type" 请求头字段
$cookie_name             #cookie名称
$limit_rate              #用于设置响应的速度限制
$msec                    #当前的Unix时间戳
$nginx_version           #nginx版本
$pid                     #工作进程的PID
$pipe                    #如果请求来自管道通信，值为"p"，否则为"."
$proxy_protocol_addr     #获取代理访问服务器的客户端地址，如果是直接访问，该值为空字符串
$realpath_root           #当前请求的文档根目录或别名的真实路径，会将所有符号连接转换为真实路径
$remote_addr             #客户端地址
$remote_port             #客户端端口
$remote_user             #用于HTTP基础认证服务的用户名
$request                 #代表客户端的请求地址
$request_body            #客户端的请求主体：此变量可在location中使用，将请求主体通过proxy_pass，fastcgi_pass，uwsgi_pass和scgi_pass传递给下一级的代理服务器
$request_body_file       #将客户端请求主体保存在临时文件中。文件处理结束后，此文件需删除。如果需要之一开启此功能，需要设置client_body_in_file_only。如果将次文件传 递给后端的代理服务器，需要禁用request body，即设置proxy_pass_request_body off，fastcgi_pass_request_body off，uwsgi_pass_request_body off，or scgi_pass_request_body off
$request_completion      #如果请求成功，值为"OK"，如果请求未完成或者请求不是一个范围请求的最后一部分，则为空
$request_filename        #当前连接请求的文件路径，由root或alias指令与URI请求生成
$request_length          #请求的长度 (包括请求的地址，http请求头和请求主体)
$request_method          #HTTP请求方法，通常为"GET"或"POST"
$request_time            #处理客户端请求使用的时间,单位为秒，精度毫秒； 从读入客户端的第一个字节开始，直到把最后一个字符发送给客户端后进行日志写入为止。
$request_uri             #这个变量等于包含一些客户端请求参数的原始URI，它无法修改，请查看$uri更改或重写URI，不包含主机名，例如："/cnphp/test.php?arg=freemouse"
$scheme                  #请求使用的Web协议，"http" 或 "https"
$server_addr             #服务器端地址，需要注意的是：为了避免访问linux系统内核，应将ip地址提前设置在配置文件中
$server_name             #服务器名
$server_port             #服务器端口
$server_protocol         #服务器的HTTP版本，通常为 "HTTP/1.0" 或 "HTTP/1.1"
$status                  #HTTP响应代码
$time_iso8601            #服务器时间的ISO 8610格式
$time_local              #服务器时间（LOG Format 格式）
$cookie_NAME             #客户端请求Header头中的cookie变量，前缀"$cookie_"加上cookie名称的变量，该变量的值即为cookie名称的值
$http_NAME               #匹配任意请求头字段；变量名中的后半部分NAME可以替换成任意请求头字段，如在配置文件中需要获取http请求头："Accept-Language"，$http_accept_language即可
$http_cookie
$http_host               #请求地址，即浏览器中你输入的地址（IP或域名）
$http_referer            #url跳转来源,用来记录从那个页面链接访问过来的
$http_user_agent         #用户终端浏览器等信息
$http_x_forwarded_for
$sent_http_NAME          #可以设置任意http响应头字段；变量名中的后半部分NAME可以替换成任意响应头字段，如需要设置响应头Content-length，$sent_http_content_length即可
$sent_http_cache_control
$sent_http_connection
$sent_http_content_type
$sent_http_keep_alive
$sent_http_last_modified
$sent_http_location
$sent_http_transfer_encoding
```

# 5. upstream（负载均衡）

## 一，upstream 内参数

| 书写方式          | 含义                                                                                                                                                                                                                                                                                        |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| server xxxxxxx    | real server地址，不写端口默认80，高并发情况下IP可以换位域名                                                                                                                                                                                                                                 |
| weight=数字       | 服务器的权重，默认值是 1， 数字越大表示接受的请求比例越大                                                                                                                                                                                                                                   |
| max_fails=数字    | Nginx 尝试连接后端主机失败的次数，这个值是配合proxy_next_upstream、 fastcgi_next_upstream 和memcached_next_upstream 这三个参数来使用的。当 nginx 接收后端服务器返回这三个参数定义的状态码时，会将这个请求转发给正常工作的后端服务器                                                         |
| fail_timeout=time | 与max_fails配合使用，在max_fails 定义的失败次数后，距离下次检查的间隔时间，默认是 10s 。 如果 max_fails是 5 , 它就检测 5 次，如果 5 次都是 502, 那么，它就会根据 fail_timeout的值，等待 10s 再去检查，还是只检查一次，如果持续 502, 在不重新加载 Nginx 配置的情况下，每隔 10s都只检查一次。 |
| backup            | 热备配置（ RS 节点的高可用），当前面激活的 RS 都失败后会自动启用热备 RS这标志着这个服务器作为备份服务器，若主服务器全部宕机了，就会向它转发请求。注意：当负载调度算法为 ip_hash 时，后端服务器在负载均衡调度中的状态不能是 weight 和 backup                                                 |
| down              | 标志着服务器永远不可用，这个参数可配合 ip_hash 使用                                                                                                                                                                                                                                         |

## 二，默认模式

```
upstream proxy1 {
    server 192.168.56.12:10002;
    server 192.168.56.13:10002;
}
```

我们默认的模式都是按照上述这种写法（默认是**轮询模式**）。

## 三，加权

```

upstream remo {
    server 192.168.2.191 weight=1;
    server 192.168.2.160 weight=3;
}

```

> 通过weight指定轮询的权重比率（与访问率成正比），应对后端服务器性能不一的情况，性能高的服务器可以设置较高权重，反之则设置较低。这个方式是按照平滑加权轮询算法进行分配，权重值越高被分配到的几率就高

## 四，ip_hash

> 每个用户发出的请求会按照ip_hash的记过进行分配，分配后的结果即每个访问者固定了的服务器了（可以有效解决动态网页中的连接共享问题）

```
upstream remo {
    ip_hash;
    server 192.168.2.191;
    server 192.168.2.160;
}
```
