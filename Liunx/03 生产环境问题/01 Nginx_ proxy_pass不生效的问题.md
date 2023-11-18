# proxy_pass不生效的问题

## 1. 需求 场景：

此时我们有两台服务器，分别是 `www.vagrant01.com` / `www.vagrant02.com` . 将用到nginx中 `proxy_pass` 以及 `rewrite` 字段实现一个代理 + 重写url功能。

此时我们从浏览器地址栏中输入 `www.vagarnt01.com/last`, 在页面不跳转的情况下，重写url + 代理，代理到站点 `www.vagrant02.com/json` 下，并显示站点 `/json` 的内容。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1682261186004/bce3c106226b40898b2a05d15ced6fb1.png)

## 2. 出现 问题：

```
# 第一种
location /last {
    proxy_pass http://www.vagrant02.com;
    rewrite /last /json last;
}

# 第二种
location /last {
    proxy_pass http://www.vagrant02.com;
    rewrite /last /json redirect;
}

```

无论是上述哪种配置，***都无妨访问 `www.vagrant02.com/json`, 访问的是 `www.vagrant01.com/json`*** . 但是我的01服务器中没有/json, 所以页面403了

## 3. 解决 方案：

```
    location /last {
        proxy_pass http://www.vagrant02.com;
        rewrite /last /json break;
    }
```

查阅了很多资料中遇发现了下面这句话，解决了这个问题。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1682261186004/bf3b5a013c014dcb86ed189835cffea7.png)
