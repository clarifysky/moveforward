# Events

# 介绍

Laravel的事件提供了一个简单的观测者实现，允许你订阅和监听事件。事件类通常存储在`app/Events`目录中，他们的监听器存储在`app/Listeners`中。

# 注册事件/监听器

Laravel中的`EventServiceProvider`提供了一个方便的位置来注册所有的事件监听器。`listen`属性包含了一个所有事件（作为键）和其监听器（作为值）的数组。

```php
/**
 * The event listener mappings for the application.
 *
 * @var array
 */
protected $listen = [
    'App\Events\PodcastWasPurchased' => [
        'App\Listeners\EmailPurchaseConfirmation',
    ],
];
```

## 生成事件/监听器类

手动创建时间和监听器是麻烦的，你可以把事件和监听器加入`EventServiceProvider`里面，然后使用`event:generate`命令。这个命令会生成所有在`EventServiceProvider`中列出的事件或监听器。当然，已经存在的事件和监听器不会收到影响。

```
php artisan event:generate
```

## 手动注册事件

通常，事件应当通过`EventServiceProvider``$listen`数组来注册，不过，你也可以通过事件派遣器来手动注册，可以使用`Event`门面或者`Illuminate\Contracts\Events\Dispatcher`协议实现：

```php
/**
 * Register any other events for your application.
 *
 * @param  \Illuminate\Contracts\Events\Dispatcher  $events
 * @return void
 */
public function boot(DispatcherContract $events)
{
    parent::boot($events);

    $events->listen('event.name', function ($foo, $bar) {
        //
    });
}
```

**通配符事件监听器**

你甚至可以使用`*`作为通配符来注册监听器，这允许你在同一个监听器内捕捉到多个事件。通配符监听器接收整个事件数据数组作为唯一的参数（单一的参数）。

```php
$events->listen('event.*', function (array $data) {
    //
});
```

# 定义事件

事件类是一个简单的持有事件相关信息的数据容器。例如，假定我们生成的`PodcastWasPurchased`事件接收一个[Eloquent ORM](https://laravel.com/docs/5.2/eloquent)对象：

```php
<?php

namespace App\Events;

use App\Podcast;
use App\Events\Event;
use Illuminate\Queue\SerializesModels;

class PodcastWasPurchased extends Event
{
    use SerializesModels;

    public $podcast;

    /**
     * Create a new event instance.
     *
     * @param  Podcast  $podcast
     * @return void
     */
    public function __construct(Podcast $podcast)
    {
        $this->podcast = $podcast;
    }
}
```

如你所见，这个事件类没有逻辑，他只是一个被购买的`Podcast`对象的容器。这个事件使用`SerializesModels`trait来优雅地序列化任何Eloquent模型，如果这个事件对象使用PHP的`serialize`函数被序列话的话。

# 定义监听器

事件监听器在他们的`handle`方法中接收事件实例。`event:generate`命令会自动导入正确的事件类，并在`handle`方法上正确地类型提示那个事件。在`handle`方法內，你可以实施任意必要的逻辑来响应事件。

```php
<?php

namespace App\Listeners;

use App\Events\PodcastWasPurchased;

class EmailPurchaseConfirmation
{
    /**
     * Create the event listener.
     *
     * @return void
     */
    public function __construct()
    {
        //
    }

    /**
     * Handle the event.
     *
     * @param  PodcastWasPurchased  $event
     * @return void
     */
    public function handle(PodcastWasPurchased $event)
    {
        // Access the podcast using $event->podcast...
    }
}
```

事件监听器可以类型提示任何他所需要的依赖。所有的事件监听器都由Laravel的服务容器解决，所以，依赖会被自动注入：

```php
use Illuminate\Contracts\Mail\Mailer;

public function __construct(Mailer $mailer)
{
    $this->mailer = $mailer;
}

```

**停止一个事件的传播**

有时候，你希望停止事件传播到其他监听器那里。你可以通过在你的监听器的`handle`方法中返回`false`来达到这个目的。

## 队列化事件监听器

需要将一个事件加入[队列](https://laravel.com/docs/5.2/queues)？这不能再简单了。只需要简单地将`ShouldQueue`接口加到监听器类上。由`event:generate`Artisan命令生成的监听器就已经把这个接口导入到当前命名空间中了，你可以直接使用它：

```php
<?php

namespace App\Listeners;

use App\Events\PodcastWasPurchased;
use Illuminate\Contracts\Queue\ShouldQueue;

class EmailPurchaseConfirmation implements ShouldQueue
{
    //
}
```

这样就好了，当这个监听器由于某个事件而被调起的时候，他就会被通过事件派遣器使用Laravel的[队列系统](https://laravel.com/docs/5.2/queues)来自动加入到队列中。队列中的监听器没有抛出异常的话，加入队列中的工作会在它被处理完之后自动被删除。

**手动访问队列**

如果你想要手动访问潜在队列工作的`delete`和`release`方法，你需要`Illuminate\Queue\InteractsWithQueue`trait，他会为你提供到这些方法的访问能力，他在自动生成监听器的时候就被导入了:

```php
<?php

namespace App\Listeners;

use App\Events\PodcastWasPurchased;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Contracts\Queue\ShouldQueue;

class EmailPurchaseConfirmation implements ShouldQueue
{
    use InteractsWithQueue;

    public function handle(PodcastWasPurchased $event)
    {
        if (true) {
            $this->release(30);
        }
    }
}
```

# 触发事件

要触发事件，你需要使用`Event`门面，然后将一个事件实例传递给`fire`方法。`fire`方法会将这个事件派遣到所有它的已注册的监听器中。

```php
<?php

namespace App\Http\Controllers;

use Event;
use App\Podcast;
use App\Events\PodcastWasPurchased;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
    /**
     * Show the profile for the given user.
     *
     * @param  int  $userId
     * @param  int  $podcastId
     * @return Response
     */
    public function purchasePodcast($userId, $podcastId)
    {
        $podcast = Podcast::findOrFail($podcastId);

        // Purchase podcast logic...

        Event::fire(new PodcastWasPurchased($podcast));
    }
}
```

或者，你也可以使用全局的`event`助手函数来激活事件：

```php

event(new PodcastWasPurchased($podcast));
```

# 广播事件

在许多现代应用中，websocket用来实现实时的，热更新的用户接口。当服务器上的一些数据被更新的时候，通常有个消息通过一个websocket链接被发送到客户端，并且被客户端处理。

为了帮助你构建这种类型的应用，Laravel让通过websocket连接“广播”你的事件变得容易。广播你的Laravel事件将会在你的服务端代码和客户端JavaScript框架之间共享相同的事件名字。

## 配置

所有的事件广播配置选项都存储在`config/broadcasting.php`配置文件中。Laravel支持几种拆箱即用的广播驱动：[Pusher](https://pusher.com/),[Redis](https://laravel.com/docs/5.2/redis)，还有一个`log`驱动用于本地开发和debug。每种驱动都包含一个配置例子。

**广播前提**

下面的依赖是事件广播需要的

- Pusher: `pusher/pusher-php-server ~2.0`
- Redis: `predis/predis ~1.0`

**队列前提**

在广播事件之前，你还需要配置和运行一个[队列监听器](https://laravel.com/docs/5.2/queues)。所有的事件广播都由队列工作完成，所以你的应用的响应时间不会受到严重的影响。

## 标记事件用于广播

要告诉Laravel一个事件应当被广播，需要在事件类上实现`Illuminate\Contracts\Broadcasting\ShouldBroadcast`接口。`ShouldBroadcast`接口要求你实现一个方法`broadcastOn`。这个方法应当返回一个这个事件应当被广播到的"频道"的名称数组：

```php
<?php

namespace App\Events;

use App\User;
use App\Events\Event;
use Illuminate\Queue\SerializesModels;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;

class ServerCreated extends Event implements ShouldBroadcast
{
    use SerializesModels;

    public $user;

    /**
     * Create a new event instance.
     *
     * @return void
     */
    public function __construct(User $user)
    {
        $this->user = $user;
    }

    /**
     * Get the channels the event should be broadcast on.
     *
     * @return array
     */
    public function broadcastOn()
    {
        return ['user.'.$this->user->id];
    }
}
```

然后，你只需要正常[激活事件](https://laravel.com/docs/5.2/events#firing-events)。一旦事件被激活了，一个广播工作就会自动通过你指定的广播驱动广播这个事件。

## 广播数据

当一个事件被广播，所有它的`public`属性都被自动序列化并作为这个事件的载荷被广播，这允许你通过你的js应用访问它的任意公开数据。所以，例如，如果你的事件又一个公开的`$user`属性，它包含了一个Eloquent模型，那么这个广播载荷就将是：

```php
{
    "user": {
        "id": 1,
        "name": "Jonathan Banks"
        ...
    }
}
```

不过，如果你想要对你的广播载荷有更好成色的控制的话，你需要给你的事件添加一个`broadcastWith`方法。这个方法应当返回你希望和事件一起被广播的数据数组：

```php
**
 * Get the data to broadcast.
 *
 * @return array
 */
public function broadcastWith()
{
    return ['user' => $this->user->id];
}
```

## 事件广播自定义

**自定义事件名字**

默认情况下，广播的事件名字是事件的全类名，所以，如果事件类名是`App\Events\ServerCreated`，那么广播事件也会是`App\Events\ServerCreated`。你可以使用通过在你的事件类上自定义一个`broadcastAs`方法来自定义这个广播事件名字。

```php
/**
 * Get the broadcast event name.
 *
 * @return string
 */
public function broadcastAs()
{
    return 'app.server-created';
}
```

**自定义队列**

默认情况下，每一个需要广播的事件都被放在在你的`queue.php`配置文件中的默认队列连接的默认队列中。你可以通过在你的事件类中添加一个`onQueue`方法来自定义由事件广播器使用的队列。这个方法应当返回你希望使用的队列的名字：

```php
 /**
 * Set the name of the queue the event should be placed on.
 *
 * @return string
 */
public function onQueue()
{
    return 'your-queue-name';
}

```

## 消耗事件广播

**Pusher**

你可以方便地使用`[Pusher](https://pusher.com/)`驱动使用Pusher的JavaScript SDK来消耗事件广播。让我们来消耗之前例子中的`App\Events\ServerCreated`事件。

```js
this.pusher = new Pusher('pusher-key');

this.pusherChannel = this.pusher.subscribe('user.' + USER_ID);

this.pusherChannel.bind('App\\Events\\ServerCreated', function(message) {
    console.log(message.user);
});
```

**Redis**

如果你在使用Redis广播器，你需要编写你自己的Reids发布/订阅消耗者来接收和广播他们，并由你来决定要使用的websocket技术。例如，你可以使用流行的[Socket.io](http://socket.io/)库，它由Node写成：

```js
var app = require('http').createServer(handler);
var io = require('socket.io')(app);

var Redis = require('ioredis');
var redis = new Redis();

app.listen(6001, function() {
    console.log('Server is running!');
});

function handler(req, res) {
    res.writeHead(200);
    res.end('');
}

io.on('connection', function(socket) {
    //
});

redis.psubscribe('*', function(err, count) {
    //
});

redis.on('pmessage', function(subscribed, channel, message) {
    message = JSON.parse(message);
    io.emit(channel + ':' + message.event, message.data);
});
```

# 事件订阅者

事件订阅者是一些你希望在其内部订阅多个事件的类，允许你在一个类中定义多个事件处理器。订阅者必须定义一个`subscribe`方法，这个会被传递给一个事件派遣器实例：

```php
<?php

namespace App\Listeners;

class UserEventListener
{
    /**
     * Handle user login events.
     */
    public function onUserLogin($event) {}

    /**
     * Handle user logout events.
     */
    public function onUserLogout($event) {}

    /**
     * Register the listeners for the subscriber.
     *
     * @param  Illuminate\Events\Dispatcher  $events
     */
    public function subscribe($events)
    {
        $events->listen(
            'App\Events\UserLoggedIn',
            'App\Listeners\UserEventListener@onUserLogin'
        );

        $events->listen(
            'App\Events\UserLoggedOut',
            'App\Listeners\UserEventListener@onUserLogout'
        );
    }

}
```

**注册一个事件订阅者**

一旦订阅者定义好了，他就可以和事件派遣器一起被注册。你可以使用`EventServiceProvider`上的`$subscribe`属性注册订阅者。例如，让我们添加一个`UserEventListener`.

```php
<?php

namespace App\Providers;

use Illuminate\Contracts\Events\Dispatcher as DispatcherContract;
use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;

class EventServiceProvider extends ServiceProvider
{
    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        //
    ];

    /**
     * The subscriber classes to register.
     *
     * @var array
     */
    protected $subscribe = [
        'App\Listeners\UserEventListener',
    ];
}
```


