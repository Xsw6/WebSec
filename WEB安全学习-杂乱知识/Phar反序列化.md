# Phar反序列化

[学些了一些基本的东西](https://tttang.com/archive/1732/)

懂了什么是phar反序列化。
触发原因：Phar文件会以序列化的形式存储用户自定义的`meta-data`，在特定函数操作该phar文件时会触发反序列化。

![在这里插入图片描述](https://img-blog.csdnimg.cn/48c1e4dac97b44969a0d8dc6e2bff1ce.png)

并且这些函数中的参数必须能够保证传入：`pahr://`
**相关绕过：**

```php
1、php://filter/read=convert.base64-encode/resource=phar://test.phar
//即使用filter伪协议来进行绕过
2、compress.bzip2://phar:///test.phar/test.txt
//使用bzip2协议来进行绕过
3、compress.zlib://phar:///home/sx/test.phar/test.txt
//使用zlib协议进行绕过
```

