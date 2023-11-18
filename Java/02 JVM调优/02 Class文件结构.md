### 一，Class 文件结构

## 1，一个简单的类

我们从一个非常简单的类开始，看看编译后的类是啥样的

```
package classes.file.format;

public class Main01 {
}
```

编译后的Class

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1699703836066/5d7411562a7e48f7822b519a50793a03.png)

> 其实我们可以看到，当我们写一个类时没有写构造方法的同时，会存在一个默认的构造方法

## 2，Class 字节码结构

> 这里我们使用xmind来详细的列举下 Class文件的结构

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1699703836066/6172e70a8d434e4e84e124feb7af691a.png)

上述结构中有特别多的内容，接下来我们使用真实的案例来分析下。

（转换工具介绍）

不过我们在分析之前，我们先来介绍下我们使用的转换工具

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1699703836066/c79ac0f0835d4088b49f89679506fa87.png)

## 3，结构简单分析

我们还是以上述xmind中的内容简单分析下，到底Class结构包含哪些东西

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1699703836066/26d8efef341049c18e91b6bb6f5d7625.png)

---

- 每个Class中都会锁定编译的版本号，以及JDK的版本号
  - Magic Number/ Minor Version/ Major Version
- 记录着每个Class中使用的常量
  - constant_pool_count/ constant_pool
- 包括记录着 Class的修饰符。例如是被public/ final等修饰
  - access_flags
- this_class/ super_class/ methods/ interfaces 等

## 4，字节码分析

> 我们通过通过进制分析工具来看下，最简单的Class被分析后长啥样

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1699703836066/65644ab6c46a484083e61f9c4e65a577.png)

但是上述这种结果让我们难以入手去分析，我们结合字节码分析工具来看下

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1699703836066/261d67cebefb45b698adff6360400404.png)

（一般信息）

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1699703836066/abe21ef482344488b554098115033e32.png)

其实一般信息这个目录结构，跟我们上述xmind中分析的Class结构完全保持一致的。

无非是列举了常量池中个数/ 本类索引/ 父类索引/ 方法个数/ 属性个数等

那么接下来我们核心分析下：【常量池】

### 一，常量池

#### 1，CONSTANT_Methodref_Info

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1699703836066/c97f1c1e379b40918d703c16a49dc6f9.png)

上述截图中包含了【类名】以及【名称描述符】，其指向的别的地址。

- cp_info #2  指向的是基础对象【java/lang/Object】
- cp_info #3 表示无返回值的构造方法

#### 2，CONSTANT_Class_Info

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1699703836066/0bb03b3a8b984bcba910942847709129.png)

- cp_info #4 其实就是一个Object对象

#### 3，CONSTANT_NameAndType_Info

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1699703836066/a1f1c6016e644d5cbb9e401424280601.png)

- cp_info #5  一个默认的构造方法
- cp_info #6  无参，无返回值的方法

#### 4，CONSTANT_Utf8_info

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1699703836066/e388adfabc514d908f25a1c3ceb998f0.png)

其实是一个固定的常量，常量池中类似这种XXX_Utf8的描述都是一些固定值常量

### 二，方法

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1699703836066/bef573ad576f4c9f8092567d612b0dff.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1699703836066/5e61ebedd61146e58db0e0c9d039e27f.png)

可以将之理解为 一个空参数的构造方法

### 三，属性

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1699703836066/4d5be66cafa04579a1ae5dd0c250d697.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1699703836066/d03950e0ef73427f9f45e77487f8735d.png)

其表示文件本身 以及文件名称

## 5，增量分析

### 一，增量分析①

```
package classes.file.format;

public class Main02 {
  public void eat() {}
}
```

我们在简单类的基础上添加了一个方法【eat】，我们分析下该Class字节码

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1699703836066/c482f041c8ee47d4b0090342f3ebd7f4.png)

### 二，增量分析②

```
package classes.file.format;

import classes.file.format.inter.ExtendInterface;

public class Main03 implements ExtendInterface {
}
```

上述示例代码中添加了接口，那么我们看下分析出来的字节码，关于接口是如何表示的呢

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1699703836066/cef07e9aa7384c1898ab18f01ac86f51.png)
