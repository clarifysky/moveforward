# 使用事件管理器

https://framework.zend.com/manual/2.2/en/tutorials/tutorial.eventmanager.html

这篇教程探索了`Zend\EventManager`的多种特性。

> 一个触发器会触发绑定在其上的所有监听器（使用`attach`来指定绑定在`event`上的`listener`）
> 若要短路监听器的执行，有两种做法：
> 1、给trigger增加第四个参数，这个参数是个回调，这个回调的参数是`listener`的执行结果，根据其结果作出`true`／`false`，如果在这个回调里面返回了`true`，则会短路`listener`的执行，后续的其他`listener`均不再执行。
> 2、在`listener`内中断事件传播。`$e->stopPropagation();`

## 术语

- **event（事件）**是一个命名的行为
- **Listener（监听器）**是一个对某个事件有回应的PHP回调
- **EventManager（事件管理器）***聚合*了一个或多个命名事件的监听器（listeners），并*触发*事件。

 通常，一个事件会被当成一个对象模型，包含有何时，怎样触发它的元数据，包括事件名字，什么对象触发了事件（the "target"），还有提供了什么参数。事件是命名的，它允许单个监听器基于事件来叉开逻辑。
 
## 开始
 
开始使用事件的最小必要事情有：

- 一个`EventManager` 实例
- 一个或多个在（一个或多个事件上）的监听器
- 一个`trigger()`调用来触发一个事件

下面是一个最简单的例子：

```php
use Zend\EventManager\EventManager;

$events = new EventManager();
$events->attach('do', function ($e) {
    $event = $e->getName();
    $params = $e->getParams();
    printf('Handled event "%s", with parameters %s', $event, json_encode($params));
});

$params = array('foo' => 'bar', 'baz' => 'bats');
$events->trigger('do', null, $params);
```

上面的代码会输出下面的东西：

```
Handled event "do", with parameters {"foo":"bar","baz":"bat"}
```

你会发现，上面的`null`是什么？

通常，你会在一个类内组织一个`EventManager`，来允许在方法中触发行为。`trigger()`中间的那个参数是"target"，如情形所描述，会是当前实例。这允许事件监听器访问调用对象（调用了事件的对象）。

```php
use Zend\EventManager\EventManager;
use Zend\EventManager\EventManagerAwareInterface;
use Zend\EventManager\EventManagerInterface;

class Example implements EventManagerAwareInterface {
    protected $events;
    
    public function setEventManager(EventManagerInterface $events) {
        $events->setIdentifiers(array(
            __CLASS__,
            get_class($this)
        ));
        $this->events = $events;
    }
    
    public function getEventManager() {
        if (!$this->events) {
            $this->setEventManager(new EventManager());
        }
        return $this->events;
    }
    
    public function do($foo, $baz) {
        $params = compact('foo', 'baz');
        $this->getEventManager()->trigger(__FUNCTION__, $this, $params);
    }
}

$example = new Example();

// 这里确定附加到`Example`实例上的`do`方法的原因是
// `setEventManager`这里设置了当前类为Identifier？
$example->getEventManager()->attach('do', function () {
    $event = $e->getName();
    $target = get_calss($e->getTarget());
    $params = $e->getParams();
    
    printf(
        'Handled event "%s" on target "%s", with parameters %s',
        $event,
        $target,
        json_encode($params)
    );
});

$example->do('bar', 'bat');
```

上面的例子基本上和第一个一样，唯一的区别就是用了中间那个参数来传递target，就是`Example`的实例，用在了监听器上。监听器现在可以拉取它（$e->getTarget()）并做一些事情。

如果你很慎重地阅读本文，你就会发现一个问题。`setIdentifier()`是干什么的？

## 共享的管理器

`EventManager`实现提供的一个层面是组织`SharedEventManagerInterface`实现的能力。

`Zend\EventManager\SharedEventManagerInterface`描述了一个为附加到带有特定*标识符*的对象上的事件*聚合*监听器的对象。它自身不会触发事件。相反，一个组织了`SharedEventManager`的`EventManager`实例会查询`SharedEventManger`来得到它所感兴趣的标识符上的监听器，然后触发那些监听器。

这到底是怎样工作的？

考虑下面的代码：

```php
use Zend\EventManager\SharedEventManager;

$sharedEvents = new SharedEventManager();
$sharedEvents->attach('Example', 'do', function($e) {
    $event = $e->getName();
    $target = get_class($e->getTarget());   // "Example"
    $params = $e->getParams();
    printf(
        'Handled event "%s" on target "%s", with parameters %s',
        $event,
        $target,
        json_encode($params);
    );
});
```

这个看起来几乎和上一个例子一样；主要的区别是这里在*开始*处有一个额外的参数`'Example'`。这段代码基本上在说，“监听'Example'目标上的'do'事件，当收到通知后，执行回调”。

这是`EventManager`的`setIdentifiers()`参数开始工作的地方。这个方法允许传入一个字符串，或是一个字符串数组，来定义名字或上下文名字或给定实例感兴趣的目标。如果给了一个数组，那么在给出的任意targets上的任意监听器都会被通知。

所以，回到上面的例子上，我们假定上面的共享监听器被注册了，并且`Example`类就如上面定义的。我们可以接下来执行下面的：

```php
$example = new Example();
$example->getEventManager()->setSharedManager($sharedEvents);
$example->do('bar', 'baz');
```

下面的内容将会被`echo`：

```
Handled event "do" on target "Example", with parameters {"foo":"bar","baz":"bat"}
```

现在，我们来扩展一下`Example`：

```php
class SubExample extends Example {
}
```

一个关于`setEventManager()`的有趣的方面是我们定义了它来同时监听`__CLASS__`和`get_class($this)`。这就意味着，调用`SubExample`上的`do()`同样会触发共享监听器！这也意味着，如果必要，我们可以附加到特定的`SubExample`上，这时候，只附加到`Example`目标上的监听器将不会被触发。

最终，用作上下文或目标的名字必须要是类名字；它们可以是一些仅在你的应用中有意义的名字。例如，你可能有一系列响应“log”或“cache”的类，这些上的监听器会被它们中的任何一个通知到。

> 我们建议使用类名，接口名，和／或抽象类名作为标识符。这样的话，确定哪些事件可用就变得容易了，这也包括找出哪些监听器可能被附加到那些事件上。接口造就了一个尤其好用的用例，它们允许一个操作附加（事件）给一组相关的类。

在任意点，如果你不想要通知共享的监听器，传递一个`null`给`setSharedManager()`

```php
$events->setSharedManager(null);
```

这样，它们就会被忽略。如果在任意点，你想要启用它们，传入`SharedEventManager`实例

```php
$events->setSharedManager($sharedEvents);
```

## 通配符

到目前为止，不管是正常的`EventManager`还是`sharedEventManager`实例，我们已经看到了我们想要附加上的，用单一字符串代表事件和目标名字的用法。如果我们想要给多个事件或目标附加一个监听器，该怎么办呢？

这就要使用通配符`*`

```php
// Multiple named events:
$events->attach(
    array('foo', 'bar', 'baz'), // events
    $listener
);

// All events via wildcard:
$events->attach(
    '*', // all events
    $listener
);

// Multiple named targets:
$sharedEvents->attach(
    array('Foo', 'Bar', 'Baz'), // targets
    'doSomething', // named event
    $listener
);

// All targets via wildcard
$sharedEvents->attach(
    '*', // all targets
    'doSomething', // named event
    $listener
);

// Mix and match: multiple named events on multiple named targets:
$sharedEvents->attach(
    array('Foo', 'Bar', 'Baz'), // targets
    array('foo', 'bar', 'baz'), // events
    $listener
);

// Mix and match: all events on multiple named targets:
$sharedEvents->attach(
    array('Foo', 'Bar', 'Baz'), // targets
    '*', // events
    $listener
);

// Mix and match: multiple named events on all targets:
$sharedEvents->attach(
    '*', // targets
    array('foo', 'bar', 'baz'), // events
    $listener
);

// Mix and match: all events on all targets:
$sharedEvents->attach(
    '*', // targets
    '*', // events
    $listener
);
```

指定多个目标和/或事件来附加的能力能够极大地苗条你的代码。

## 聚合监听器

另一个实现监听多个事件的方式是聚合监听器，由`Zend\EventManager\ListenerAggregateInterface`代表。通过这个实现，但一类可以监听多个事件，附加多个实例方法作为监听器。

这个接口定义了两个方法，`attach(EventManagerInterface $events)`和`detach(EventManagerInterface $events)`。通常，你需要传递一个`EventManager`给他们，然后就可以开始实现他们并决定如何做了。

一个例子：

```php
use Zend\EventManager\EventInterface;
use Zend\EventManager\EventManagerInterface;
use Zend\EventManager\ListenerAggregateInterface;
use Zend\Log\Logger;

class LogEvents implements ListenerAggregateInterface {
    protected $listeners = array();
    protected $log;
    
    public function __construct (Logger $log) {
        $this->log = $log;
    }
    
    public function attach(EventManagerInterface $events) {
        $this->listeners[] = $events->attach('do', array($this, 'log'));
        $this->listeners[] = $events->attach('doSomethingElse', array($this, 'log'));
    }
    
    public function detach(EventCollection $events) {
        foreach ($this->$listeners as $index => $listener) {
            if ($events->detach($listener)) {
                unset($this->listeners[$index]);
            }
        }
    }
    
    public function log(EventInterface $e) {
        $events = $e->getName();
        $params = $e->getParams();
        $this->log->info(sprintf('%s: %s', $event, json_encode($params)));
    }
}
```

你可以使用`attach()`或`attachAggregate()`来附加。

```php
$logListener = new LogEvents($logger);

$events->attachAggregate($logListener);
$events->attach($logListener);
```

当触发时，聚合起器附加上的任意事件都会被通知。

为什么烦恼？两个原因：

- 聚合器允许你拥有全状态的监听器。上面的例子通过logger证明了这个。另外一个例子是追踪配置选项。
- 聚合器使解除监听附加变得方便，当调用`attach()`时，你会收到一个`Zend\Stdlib\CallbackHandler`实例；解除（`detach()`）监听的唯一方式是把那个实例传回-这意味着，如果你想要在后面detach，你需要把哪个实例保存在某个地方。聚合器已经为你做了这些——如上面例子中你看到的一样。

### 结果反省

有时候你需要知道你的监听器返回了什么。你需要记住，你可能在一个事件上有多个监听器；对于结果的接口必须保持一致而不要管监听器的数量。

`EventManager`实现默认返回了`Zend\EventManager\ResponseCollection`实例。这个类继承了PHP的`SplStack`，允许你以反向遍历响应（由于最后一个被执行的有可能是你最感兴趣的额那个）。它还实现了下述方法：

- `first()`会拉取第一个接收到的结果
- `last()`会拉取最后一个接收到的结果
- `contains($value)`允许你测试所有的值来看是否一个给定的值已经被收到了，收到了就会返回布尔值`true`，否则返回`false`。

通常，你不需要担心从事件返回的结果，由于触发事件的对象不会真正了解有哪些监听器附加到它上面。然而，有时你需要在感兴趣的数据得到之后短路监听器执行。

### 短路监听器执行

在得到一个特别的结果后，或者在一个监听器发现有东西出错了，或者它可以比目标更快地返回一些结果，你可能希望短路监听器的执行。

如例子中所述，一个添加`EventManager`的基本原因是缓存装置。你可以在方法中较早地触发一个事件，当有缓存找到时，将其返回，在这个方法的稍后的地方触发另外一个事件来种下缓存。

`EventManager`组件提供两种处理这个问题的方式。第一种是传递一个回调作为`trigger()`的最后一个参数；如果那个回调返回了`true`，执行就中断了。

例如：

```php
public function someExpensiveCall($criteria1, $criteria2)
{
    $params  = compact('criteria1', 'criteria2');
    $results = $this->getEventManager()->trigger(
        __FUNCTION__,
        $this,
        $params,
        function ($r) {
            return ($r instanceof SomeResultClass);
        }
    );
    if ($results->stopped()) {
        return $results->last();
    }

    // ... do some work ...
}
```

借助这个范例，我们知道执行中断的可能原因是最后一个结果契合了回调的标准（要求）；因此，我们简单地返回最后一个结果。

另外一种中断执行的方式是在监听器内，在它接收的`Event`对象上搞事情。在这种情形下，监听器调用`stopPropagation(true)`，并且`EventManager`紧接着就会返回而不通知任何多余的监听器。

```php
$events->attach('do', function ($e) {
    $e->stopPropagation();
    return new SomeResultClass();
});
```
这种情况，当然，在使用触发模型的时候搞出了一些歧义，如你不能够再确定最后一个结果是否符合你搜寻的标准。因此，我们建议将一种实现或别的实现标准化。

### 保持有序

偶然情况下，你可能需要关心监听器执行的顺序。例如，你可能希望尽早地做一些日志，以便在短路发生时你已经做好了日志；或者，实现一个缓存，在缓存命中时尽早返回它，在保存缓存时稍晚点执行。

每一个`EventManager::attach()`和`SharedEventManager::attach()`接收一个额外的参数，一个*priority*。默认情况下，如果这个参数缺省了，监听器就会获得一个1的权限，然后按照它们在附加的时候顺序执行。然而，如果你提供了一个权限值，你就会影响这个执行顺序。

- 高权限值早执行
- 低权限值晚执行

例子

```php
$priority = 100;
$events->attach('Example', 'do', function ($e) {
    $event = $e->getName();
    $target = get_class($e->getTarget());
    $params = $e->getParams();
    
    printf(
        'Handled event "%s" on target "%s", with parameters %s',
        $event,
        $target,
        json_encode($params)
    );
}, $priority);
```

这个就会早执行，如果你将权限值改为了`-100`，他就会以一个低的权限执行，执行得比较迟。

应当避免设置权限，除非确实必要。


