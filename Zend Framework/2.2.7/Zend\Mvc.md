# Zend\Mvc

https://framework.zend.com/manual/2.2/en/modules/zend.mvc.intro.html

## MVC层介绍

`Zend\Mvc`专注于灵活性和性能。

MVC层构建在以下组建的上层：

- `Zend\ServiceManager`-ZF在`Zend\Mvc\Service`中提供了一系列默认的服务定义设置。`ServiceManager`配置和创建你的应用实例和工作流。
- `Zend\EventManager`-这个MVC是事件驱动的，这个组件从应用初始启动之后就被用在任意地方，包括返回响应和请求调用，设置和拉去路由并匹配路由，还有渲染视图。
- `Zend\Http`-特定的请求和响应对象
- `Zend\Stdlib\DispatchableInterface`。所有的控制器都是简单的可派遣对象。

在MVC层中，一些子组件被释出了：

- `Zend\Mvc\Router`包含有路由请求的类，换句话说，他会将请求匹配到各自的控制器（或可派遣对象）上。
- `Zend\Http\PhpEnvironment`提供了一系列用于HTTP`Request`和`Response`对象的装饰物，这可以确保请求携带当前环境（包括查询参数（query parameters），POST参数，HTTP头部等等）被注入。
- `Zend\Mvc\Controller`，一系列抽象的控制器类，主要负责比如事件接通，行为派遣等等。
- `Zend\Mvc\Service`为默认的应用工作流提供了一系列`ServiceManager`工厂和定义。
- `Zend\Mvc\View`提供了默认的渲染器选择接通，视图脚本分辨，助手助层，还有更多；除此之外，它还提供了许多绑入MVC工作流的监听器，提供的特性例如自动模版名称分辨，自动视图模型创建和注入，等等。

到MVC的关口是`Zend\Mvc\Application`对象（以后对应为`Application`）。它的主要职责是**启动**资源，**路由**请求，在路由过程中拉取和**派遣**匹配到的控制器。一旦完成，它会**渲染**视图，**完成**请求，返回和发送响应。

### 基本的模块结构

推荐的模块结构如下：

```
module_root<named-after-module-namespace>/
    Module.php
    autoload_classmap.php
    autoload_function.php
    autoload_register.php
    config/
        module.config.php
    public/
        images/
        css/
        js/
    src/
        <module_namespace>/
            <code files>
    test/
        phpunit.xml
        bootstrap.php
        <module_namespace>/
            <test code files>
    view/
        <dir-named-after-module-namespace>/
            <dir-named-after-a-controller>/
                <.phtml files>
```

由于模块扮演着命名空间，所以模块根目录应当就是那个命名空间。

模块根目录下的直接文件`Module.php`应该在模块的命名空间下，就像下面的：

```php
namespace ZendUser;

class Module
{
}
```

当在`Modeul.php`中定义了`init()`方法，这个方法会在`Zend\ModuleManager`监听器载入模块类的时候触发，监听器默认会将自己（`Zend\ModuleManager`）的一个实例传递进来。这允许用户实施如设置模块特定的时间监听器的任务。但请注意，`init()`会在**每次**页面请求时为**每个**模块调用，并只能用于**轻量**的任务，如注册事件监听器。类似的，一个`onBootstrap()`方法（接受一个`MvcEvent`实例）也可能被定义；它也会在每次页面请求时被触发，也只能用于轻量的任务处理。

### 启动一个应用

`Application`有六个基本的应用：

- **configuration**，通常是一个数组或是实现了`Traversable`的对象
- **ServiceManager**实例
- **EventManager**实例
- **ModuleManager**实例
- **Request**实例
- **Response**实例

你对于整个工作流有很大的控制力。使用`ServiceManager`，你对于那个服务可用，它们是怎么例证的，什么依赖项注入了他们有着很好的细节控制。使用`EventManager`的权限系统，你可以在运行过程中的任意地方拦截应用的任意事件（"bootstrap", "route", "dispatch", "dispatch.error", "render"和"finish"），这允许你按需构建你自己的应用工作流。

### 引导一个模块化应用

一个大型应用工作的时候，它的配置从哪里来的？当我们创建一个模块应用，就会发现它来自模块自身，我们如何得到并聚合它们的呢？

答案是通过`Zend\ModuleManager\ModuleManager`。这个组件允许你指定模块的位置。然后，它会定位并初始化它。模块类会绑到`ModuleManager`上的多个监听器中以便于给应用提供配置，服务，监听器还有很多别的东西。

#### 配置模块管理器

我们需要将模块添加到`application.config.php`的`modules`数组中。

每一有要`Application`知道配置的`Module`类都需要定义一个`getConfig()`方法。那个方法应当返回一个数组或`Traversable`对象。例如： 
```php
namespace ZendUser;

class Module
{
    public function getConfig()
    {
        return include __DIR__ . '/config/module.config.php'
    }
}`
```

