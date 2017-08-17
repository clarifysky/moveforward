# Introduction

https://laravel.com/docs/5.2/facades

Facade提供针对在应用的服务容器中可用的类的静态接口。Laravel运行着很多facades。Laravel的facades就像服务容器中的潜在类的“静态代理”一样。它提供简要的，富有表达力的语法，和传统静态方法比起来，它兼顾诊断性和灵活性。

# 使用门面

在Laravel的上下文中，门面是一个提供了到一个容器中的对象的访问功能的类。这个装置在`Facade`类中工作。Laravel的门面，还有任何你自定义的门面，都会继承基类`Illuminate\Support\Facades\Facade`类。

门面类只需要实现一个方法：`getFacadeAccessor`，它的职责是决定需要从容器中获取什么。`Facade`基类利用`__callStatic()`魔术方法来延迟来自你的门面到要解决的对象的调用。

下面的例子中，有个到Laravel的缓存系统的调用，查看下面这段代码，你会觉得静态方法`get`是在`Cache`类上调用的。

```php
<?php

namespace App\Http\Controllers;

use Cache;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
    /**
     * Show the profile for the given user.
     *
     * @param  int  $id
     * @return Response
     */
    public function showProfile($id)
    {
        $user = Cache::get('user:'.$id);

        return view('profile', ['user' => $user]);
    }
}
```

注意，靠近文件顶部的位置，我们“导入”了`Cache`类，这个门面提供了一个访问潜在的`Illuminate\Contracts\Cache\Factory`接口的代理。任何我们使用这个门面发出的调用都会被传递给潜在的Laravel缓存系统。

查看`Illuminate\Support\Facade\Cache`类，你会看到没有静态方法`get`:

```php
class Cache extends Facade
{
    /**
     * Get the registered name of the component.
     *
     * @return string
     */
    protected static function getFacadeAccessor() { return 'cache'; }
}
```

相反，`Cache`门面继承`Facade`基类并定义了`getFacadeAccessor`方法。记住，这个方法的职责是返回服务容器绑定的名字。当一个用户引用了`Cache`门面上的任意静态方法，Laravel都会从服务容器中解决`cache`绑定（获得对象）并在那个对象上运行亲请求的方法（在这个例子中，是`get`)。



