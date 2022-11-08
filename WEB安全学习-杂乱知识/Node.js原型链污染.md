# Node.js原型链污染

[学习1](https://www.anquanke.com/post/id/236182#h3-10)

[学习2](https://xz.aliyun.com/t/10809#toc-8)

[学习3](https://xz.aliyun.com/t/7182#toc-2)

值得注意的一点是区别于java的继承关系，Node.js当我们修改上层的原型的时候,底层的实例会发生动态继承从而产生一些修改。（`这也是漏洞的一个关键因素`）

理解`__proto__`和`prototype`的关系

![image-20220926112633625](https://cdn.jsdelivr.net/gh/zx-creat/myblog@master/img/202209261126724.png)

理解同步和异步的区别：
`同步：先输出文件内容，再输出一段话
异步：先输出一段话，后输出文件内容`