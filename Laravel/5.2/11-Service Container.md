# 介绍

> Laravel使用反射实现服务容器。

Laravel的服务容器是管理类依赖和实施依赖注入的强力工具。依赖注入是一个精致的短语，它基本上意味着：类的依赖项通过构造器，或者，在别的情形下，通过"setter"方法被“注入”到类中。

我们来看一个简单的例子：

```php
<?php
namespace App\Jobs;
use App\User;use Illuminate\Contracts\Mail\Mailer;use Illuminate\Contracts\Bus\SelfHandling;
class PurchasePodcast implements SelfHandling{    /**     * The mailer implementation.     */    protected $mailer;
    /**     * Create a new instance.     *     * @param  Mailer  $mailer     * @return void     */    public function __construct(Mailer $mailer)    {        $this->mailer = $mailer;    }
    /**     * Purchase a podcast.     *     * @return void     */    public function handle()    {        //    }}
```

在这个例子中，PurchasePodcast工作需要在一个podcast被购买后发送一封邮件。所以，我们会注入一个可以发送电子邮件的服务。一旦服务被注入了，我们可以简单地将它与另外一个实现交换（swap it out with another implemention）。我们也可以简单地“嘲弄”（mock），或在测试应用的时候创建一个虚假的邮件服务实现。

对Laravel服务容器的深入理解基本上就可以创建一个强力的，大型的应用，包括对Laravel的核心自身作贡献。

# 绑定

几乎所有的服务容器绑定都会在[service provider](https://laravel.com/docs/5.2/providers)内被注册，所以这些例子会在那种上下文中使用容器来演示。然而，如果没有依赖任何接口，就无需绑定类到容器中。容器不需要被通知如何创建这些对象，因为它可以使用PHP的反射服务自动解决这些“固结的”对象。

在一个服务提供者中，你总是可以通过`$this->app`实例变量来访问容器。我们可以使用`bind`方法来注册一个绑定，然后传入我们需要注册的类或者接口名，再包括一个返回这个类的实例的`Closure`:

```php
$this->app->bind('HelpSpot\API', function ($app) {    return new HelpSpot\API($app['HttpClient']);});
```

注意，我们将容器自身作为决定者的一个参数来接收，我们可以转而使用容器来解决我们要构建的对象的子依赖项。

**绑定一个单例**

`singleton`方法将类或接口绑定到容器中，但它应当只决定一次，然后在随后的对容器的调用中将返回相同的实例：

```php
$this->app->singleton('FooBar', function ($app) {    return new FooBar($app['SomethingElse']);});
```

**绑定实例**

你也可以使用`instance`方法来将一个现成的对象绑定到容器中。在随后的对容器的调用中将会返回这个给定的实例：

```php
$fooBar = new FooBar(new SomethingElse);
$this->app->instance('FooBar', $fooBar);
```

## 绑定接口到实现中

服务容器的一个强力特性是它可以绑定一个接口到一个给定的实现上的能力。例如，我们假定我们有一个`EventPusher`接口和一个`RedisEventPusher`实现，一旦我们编写好了这个接口的`RedisEventPusher`实现（`RedisEventPusher`是`EventPusher`的实现）的代码，我们就可以像这样将它注册到服务容器中：

```php
$this->app->bind('App\Contracts\EventPusher', 'App\Services\RedisEventPusher');
```

这会告诉容器它应当在一个类需要一个`EventPusher`的实现的时候将`RedisEventPusher`注入进去，然后我们可以在构造器中类型提示`EventPusher`接口，或者任意别的依赖项由服务容器注入的地方：

```php
use App\Contracts\EventPusher;
/** * Create a new class instance. * * @param  EventPusher  $pusher * @return void */public function __construct(EventPusher $pusher){    $this->pusher = $pusher;}
```

## 上下文的绑定

有时你可能有两个利用了同一个接口的类，但是你希望注入不同的实现给每一个类。例如，当我们的系统接收到一个新的订单时，我们可能希望通过`PubNub`而不是Pusher来发送一个事件。Laravel提供了一个简单的，流畅的接口来定义这个行为：

```php
$this->app->when('App\Handlers\Commands\CreateOrderHandler')          ->needs('App\Contracts\EventPusher')          ->give('App\Services\PubNubEventPusher');
```

你甚至可以传递一个闭包给`give`方法：

```php
$this->app->when('App\Handlers\Commands\CreateOrderHandler')          ->needs('App\Contracts\EventPusher')          ->give(function () {                  // Resolve dependency...              });
```

**绑定原始事物**

有时你可能有一个类接收一些注入的类，但也需要一个注入的原始东西的值，例如整数。你可以方便地使用上下文绑定来注入你的类需要的任意值：

```php
$this->app->when('App\Handlers\Commands\CreateOrderHandler')          ->needs('$maxOrderCount')          ->give(10);
```

**打标签**

偶然的情况下，你可能需要解决所有明确的“分类”的绑定。例如，也许你要构建一个接收许多不同`Report`接口实现的数组的报告集合器。在注册了`Report`实现之后，你可以使用`tag`方法来给它们指定一个tag:

```php
$this->app->bind('SpeedReport', function () {    //});
$this->app->bind('MemoryReport', function () {    //});
$this->app->tag(['SpeedReport', 'MemoryReport'], 'reports');
```

一旦服务被打上标签，你就可以很方便地通过`tagged`方法来解决它们所有：

```php
$this->app->bind('ReportAggregator', function ($app) {    return new ReportAggregator($app->tagged('reports'));});
```

# 确定/解决

在容器之外有很多确定某事的办法，首先，你可能要使用`make`方法，他接收你希望解决的类或者接口的名字：

```php
$fooBar = $this->app->make('FooBar');
```

第二步，你可以像数组那样访问容器，因为它实现了PHP的`ArrayAccess`接口：

```php
$fooBar = $this->app['FooBar'];
```

最后，但是最重要的，你可以简单地在一个由容器解决的类的构造器中“类型提示”依赖项，包括[controllers](https://laravel.com/docs/5.2/controllers)，[event listeners](https://laravel.com/docs/5.2/events)，[queue jobs](https://laravel.com/docs/5.2/queues)，[middleware](https://laravel.com/docs/5.2/middleware)，和更多。在练习中，这是由容器决定的你的大多数对象的方式。（In practice, this is how most of you objects resovled by the container.）

容器会自动为它解决出的类注入依赖项。例如，你可能在一个控制器的构造器中类型提示了一个由你的应用定义的仓库，这个仓库会自动被解决并注入这个类：

```php
<?php
namespace App\Http\Controllers;
use App\Users\Repository as UserRepository;
class UserController extends Controller{    /**     * The user repository instance.     */    protected $users;
    /**     * Create a new controller instance.     *     * @param  UserRepository  $users     * @return void     */    public function __construct(UserRepository $users)    {        $this->users = $users;    }
    /**     * Show the user with the given ID.     *     * @param  int  $id     * @return Response     */    public function show($id)    {        //    }}
```

# 容器事件

服务容器每次解决一个对象都会`fire`一个事件。你可以使用`resolving`方法来监听这个事件：

```php
$this->app->resolving(function ($object, $app) {    // Called when container resolves object of any type...});
$this->app->resolving(FooBar::class, function (FooBar $fooBar, $app) {    // Called when container resolves objects of type "FooBar"...});
```

如你所见，被解决的对象会被传递给回调，允许你在它被送达它的消耗者之前设置任意额外的属性。

