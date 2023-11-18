# proxy_pass 如何传递自定义header

## 场景：

1. 我们的前端产品分别是：平台 以及编辑器。平台部署在 `166`服务上，编辑器以及后端代码都部署在 `167`上。
2. 我们前端往后端传递token的时候，都是通过自定义header(**certificat**e)，来进行传递的
3. 因为我们平台是有登录功能的，所以登录后我们可以将token保存起来，每次平台发送请求都可以添加自定义header(**certificate**)来进行发送
4. 但是我们在平台点击编辑的时候，会跳转编辑器。编辑器并没有登录功能的，所以需要拿到我们的自定义header
5. 所以想用 `proxy_pass` 的方式将自定义header **certificate** 传递过去

## 解决方案：

### 第一步 启动配置

根据官网的要求，我们需要开启 `underscores_in_headers on;` 的功能。 开启允许使用下划线。[官网地址](http://nginx.org/en/docs/http/ngx_http_core_module.html#underscores_in_headers)

官网具体解释如下：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1681981903015/2851d5ab151a48d6a6d237bda7b73eff.png)

nginx 配置如下：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1681981903015/3e68c2aa44554e07b29afba859f7e736.png)

### 第二步 设置代理

在设置代理的时候，添加自定义header，但是自定义header有规范要求，必须是 `http_` 开头的。

具体的配置如下：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1681981903015/6e01421c97014701902412af2616af7a.png)

其实我们传递的header 是 `certificate` . 但是我们必须以 `http_` 开头

### 第三步 结果验证

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1681981903015/c25d9fee868d489b997e0ab390b16614.png)

我们可以通过设置log 来打印进行验证

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1681981903015/8976ebc39ae441c5a04bf516e30e1ad6.png)
