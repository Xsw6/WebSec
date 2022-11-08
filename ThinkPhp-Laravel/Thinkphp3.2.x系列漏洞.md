# Thinkphp3.2.x系列漏洞

看了下简单的[框架分析](https://zhuanlan.zhihu.com/p/37054902)

## RCE

[学习](https://0xcreed.jxustctf.top/2021/07/ThinkPHP3-2-x-RCE%E5%A4%8D%E7%8E%B0/)（漏洞分析）

要知道其`控制器和对应方法创建`
![image-20220924101216279](https://cdn.jsdelivr.net/gh/zx-creat/myblog@master/img/202209241012549.png)

http://网址/index.php?m=模块名称&c=控制器&a=方法
关键利用点是走通利用链，最后走到include 文件包含。
中间关键函数例如`exce、run、include`函数

## Where注入

[学习](https://xz.aliyun.com/t/2812#toc-11)

### 创建数据库

```
create database thinkphp;
create table user (id int,
userName varchar(20),
password varchar(20)
);
INSERT INTO user(id,userName,password) values (1,'admin','passwd');
```

