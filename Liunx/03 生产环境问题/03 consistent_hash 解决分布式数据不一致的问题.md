# 一，需求

因公司的产品需要，该项目(此处可以理解为A项目)是分布式部署（多台Java服务是平行部署的），由于没有第三方中间件，所以每次用户登录后将用户信息保存到服务器中。

服务器之间的负载均衡是用nginx来做的，虽然使用了 `ip_hash` 模式，但是如果用户处于登录状态，但是IP切换了很有可能导致用户的请求被转发到别的服务器上，恰巧该服务器上并没有存储用户的登录信息，所以导致用户登录失效了。

虽然这种方式的后果不严重，而且是非常小的概率会发生，但是我们开发是非常严谨的。我们应该如何解决这个问题呢

# 二，解决

## 1，原负载均衡方式

```
upstream proxy1 {
    ip_hash;
    server 192.168.56.12:10002;
    server 192.168.56.13:10002;
}
```

以上这种方式是将客户端的IP hash化，通过hash值映射到某个服务器上。但是问题就是我们上述所说的内容

## 2，新的方式

> 我们可以想想，什么东西是用户登陆后一定不会发生变化呢。对了，我们使用userId/ token都可以。所以我们可以将token等hash化，通过hash后的值来选择服务器。这样只要userId/ token不会发生变化，请求访问到的服务器就不会发生变化，怎么做呢？？？ 我们来看如下处理。

### 一，**ngx_http_upstream_consistent_hash** 模块

我们可以使用一致性hash的这个模块来 将自定义信息hash化，这个模块不是nginx自带的模块，需要我们手动添加

---

首先我们需要从[GitHub](https://github.com/replay/ngx_http_consistent_hash)中将压缩包下载到某个特定目录

```
# 如果没有unzip命令的话，可以注册的方式
[root@localhost soft]# yum install unzip -y

[root@localhost soft]# ll
total 1192
drwxr-xr-x. 9 1001 1001     186 Oct 22 11:13 nginx
-rw-r--r--. 1 root root 1213919 Jun 13 16:33 nginx-1.25.1.tar.gz
drwxr-xr-x. 2 root root      87 Sep  3  2018 ngx_http_consistent_hash-master
-rw-r--r--. 1 root root    3787 Oct 22 11:08 ngx_http_consistent_hash-master.zip

# 在执行配置文件的时候，将模块添加到其中
[root@localhost soft]# ./configure --prefix=/opt/install/nginx --with-http_stub_status_module --with-http_stub_status_module --add-module=../ngx_http_consistent_hash-master
```

### 二，Nginx 配置

```
upstream proxy1 {
    consistent_hash $http_user_id;
    server 192.168.56.12:10002;
    server 192.168.56.13:10002;
}
```

设置一致性hash `consistent_hash` 标识。

### 三，测试

```
$ curl -H "user_id:1111111111" 192.168.56.11/user/str
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    18  100    18    0     0   5439      0 --:--:-- --:--:-- --:--:--  9000我是服务器222

$ curl -H "user_id:111111111111" 192.168.56.11/user/str
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    18  100    18    0     0   5976      0 --:--:-- --:--:-- --:--:--  9000我是服务器111
```

# 三，参考文献

- [ngx_http_consistent_hash GitHub 地址](https://github.com/replay/ngx_http_consistent_hash)
- [Nginx 官网地址](https://www.nginx.com/resources/wiki/modules/consistent_hash/)
- [csdn 他人博客](https://blog.csdn.net/guowenyan001/article/details/51305941)
- [一致性hash简述（csdn版）](https://blog.csdn.net/weixin_34284188/article/details/89663041)
- [一致性hash简述（自我笔记）](https://cloud.fynote.com/share/d/JfRMWAlh "自我")
