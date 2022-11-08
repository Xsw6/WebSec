# Laravel

## 环境

php版本:7.3.4
Larvel5.4:composer create-project --prefer-dist laravel/laravel laravel5.4 "5.4.*"

学习这篇文章：https://www.anquanke.com/post/id/258264#h3-11

## pop1


因为这篇文章中的__destruct()方法中存在删除文件的方法。

## pop2

依旧是上面文章中的pop2，但是存在一些问题。
因为作者忽略了，在反序列化需要进行的wakeup方法。
![image-20221002191425254](https://cdn.jsdelivr.net/gh/zx-creat/myblog@master/img/202210021914305.png)

那么接下来绕过wakeup方法就行了吗？答案是不行。原因如下：

![image-20221002191542288](https://cdn.jsdelivr.net/gh/zx-creat/myblog@master/img/202210021915363.png)

## pop3

注意点：isset($this->extensions[$rule])根据`key判断`
is_callable() 函数用于检测函数在当前环境中是否可调用。

## pop4

注意点：Manager是个抽象类，找了 ChannelManager来实现。而ChannelManager刚好也充当了其关键部分！也是其他实现类无法替代得。

## pop5

讲的很清楚。

## pop6

关键点：
![image-20221003093536811](https://cdn.jsdelivr.net/gh/zx-creat/myblog@master/img/202210030935876.png)

![image-20221003093616187](https://cdn.jsdelivr.net/gh/zx-creat/myblog@master/img/202210030936276.png)

进入$this->dispatchToQueue($command);
![image-20221003093707089](https://cdn.jsdelivr.net/gh/zx-creat/myblog@master/img/202210030937189.png)

## pop7

关键点：

利用$queue = call_user_func($this->queueResolver, $connection);其中两个参数均可控，寻找到EvalLoader.php中得eval方法。

至此以上利用都是在PendingBroadcast得__destruct()方法中触发得。

## pop8

看上去绕！
有几个关键触发点。
`is_file`将对象当作字符串处理，调用任意对象的`toString`。
![image-20221003112715057](https://cdn.jsdelivr.net/gh/zx-creat/myblog@master/img/202210031127181.png)

## pop9

其实就是将`$this->rfc = new ValidGenerator($payload);`替换成了：`$this->headers = new ValidGenerator($payload);`
后续就没接着学习了。

## 总结

其实就是要搞清楚函数之间的调用。以及一些php语法。对比java可以说是非常简单了。