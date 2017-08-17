# ServiceProvider

介绍
服务供应者是所有Laravel应用引导的重要位置。你自己的应用，也包括所有Laravel的核心服务，都是由服务供应者启动。

但是，我们所说的“引导”是什么意思呢？通常，我们说是注册某些事情，包括注册服务容器绑定，事件监听，中间件，甚至是路由。服务提供者对于配置你的应用来说是重要的东西。

如果你打开了Laravel中的config/app.php文件，你会看到一个providers数组。这些是所有的会为你的应用加载上的服务提供者类。当然，它们中的一些是“延迟”的提供者，意味着它们不会为每个请求都载入，仅仅在它们所提供的服务真正被需要的时候。

在这个概览中，你将学习到如果编写你自己的服务提供者并将它们注册到你的Laravel应用。

编写服务提供者
所有的服务提供者都扩展自Illuminate\Support\ServiceProvider类。这个抽象类要求你至少为你的提供者定义一个方法：register。在register方法中，你应当仅仅将东西绑定在服务容器上。你应当永远不要试图注册任何事件监听，路由，或在register方法中注册功能的任意片段。

Artisan CLI可以通过make:provider命令来方便地生成新的服务提供者：
php artisan make:provider RiakServiceProvider

Register方法
如上所述，在register方法内，你应当仅仅绑定东西到服务容器上。你永远别试图在register方法内注册任何事件监听器，路由，或者任何功能的片段。要不然，你可能会意外地用到一个由服务提供者提供，但是还没有载入的服务。

现在，我们来看一个基本的服务提供者：
<?php
namespace App\Providers;
use Riak\Connection;use Illuminate\Support\ServiceProvider;
class RiakServiceProvider extends ServiceProvider{    /**     * Register bindings in the container.     *     * @return void     */    public function register()    {        $this->app->singleton(Connection::class, function ($app) {            return new Connection(config('riak'));        });    }}
这个服务提供者仅仅定义了一个register方法，并用那个方法在服务容器中定义了一个Riak\Connection实现，如果你不知道服务容器怎么工作，检查它的文档。

Boot方法
所以，如果我们需要在我们的服务供应者里面注册一个视图创作者又会怎样？这应该在boot方法中完成。这个方法会在所有服务供应者注册完毕之后被调用，意味着你可以访问别的所有已经被框架注册的服务：
<?php
namespace App\Providers;
use Illuminate\Contracts\Events\Dispatcher as DispatcherContract;use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;
class EventServiceProvider extends ServiceProvider{    // Other Service Provider Properties...
   /**     * Register any other events for your application.     *     * @param  \Illuminate\Contracts\Events\Dispatcher  $events     * @return void     */    public function boot(DispatcherContract $events)    {        parent::boot($events);
      view()->composer('view', function () {            //        });    }}
Boot方法依赖注入
你可以对你的服务提供者的boot方法进行类型提示依赖项（对依赖项进行类型提示）。服务容器会自动注入你所需的任何依赖项：
use Illuminate\Contracts\Routing\ResponseFactory;
public function boot(ResponseFactory $factory){    $factory->macro('caps', function ($value) {        //    });}
注册提供者
所有的服务提供者都注册在config/app.php配置文件中。这个文件包含了一个你可以列出你的服务提供者名字的providers数组。默认情况下，一系列Laravel核心服务提供者会被列在这个数组中。这些提供者引导着核心的Laravel组件，例如mailer, queue, cache, 和其他的一些。

要注册你自己的提供者，简单地将它加进去：
'providers' => [    // Other Service Providers
    App\Providers\AppServiceProvider::class,],
延迟提供者
如果你的提供者仅仅注册绑定在了服务容器上，你可以选择延迟它的注册直到一个注册的绑定真正被需要。延迟这种供应者的载入会提高你的应用的性能，因为它不会在每次请求到来的时候都从文件系统上载入。

要延迟一个提供者的载入，将defer属性设置为true，并定义一个provides方法。这个provides方法返回提供者注册的服务容器绑定：
<?php
namespace App\Providers;
use Riak\Connection;use Illuminate\Support\ServiceProvider;
class RiakServiceProvider extends ServiceProvider{    /**     * Indicates if loading of the provider is deferred.     *     * @var bool     */    protected $defer = true;
   /**     * Register the service provider.     *     * @return void     */    public function register()    {        $this->app->singleton(Connection::class, function ($app) {            return new Connection($app['config']['riak']);        });    }
   /**     * Get the services provided by the provider.     *     * @return array     */    public function provides()    {        return [Connection::class];    }
}
Laravel编译并存储了一个由延迟服务供应者提供的所有服务的列表，和它的服务提供者类的名字一起。然后，仅在你试图决定这些服务中的一个的时候，Laravel会载入这个服务提供者。

