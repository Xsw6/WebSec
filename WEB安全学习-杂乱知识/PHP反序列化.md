# PHP反序列化

就是简单写个总结：

1、清楚序列化后的字符串代表得含义

![image-20220926203029160](https://cdn.jsdelivr.net/gh/zx-creat/myblog@master/img/202209262030234.png)

2、魔法函数

```
__wakeup() //使用unserialize时触发
__sleep() //使用serialize时触发
__destruct() //对象被销毁时触发
__call() //在对象上下文中调用不可访问的方法时触发
__callStatic() //在静态上下文中调用不可访问的方法时触发
__get() //用于从不可访问的属性读取数据
__set() //用于将数据写入不可访问的属性
__isset() //在不可访问的属性上调用isset()或empty()触发
__unset() //在不可访问的属性上使用unset()时触发
__toString() //把类当作字符串使用时触发,返回值需要为字符串
__invoke() //当脚本尝试将对象调用为函数时触发
构造函数__construct()：当对象创建(new)时会自动调用。但在unserialize()时是不会自动调用的。
析构函数__destruct()：当对象被销毁时会自动调用。
```

