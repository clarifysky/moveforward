# 使用事件管理器

https://framework.zend.com/manual/2.2/en/tutorials/tutorial.eventmanager.html

这篇教程探索了`Zend\EventManager`的多种特性。

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

`Zend\EventManager\SharedEventManagerInterface`描述了一个为附加到带有特定*标识符*的对象上的事件聚合监听器的对象。它自身不会触发事件。相反，一个组织了`SharedEventManager`的`EventManager`实例会为在它感兴趣的标识符上的监听器查询`SharedEventManger`，然后触发那些监听器。

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

这是`EventManager`的`setIdentifiers()`参数开始工作的地方。这个方法允许传入一个字符串，或是一个字符串数组，来定义给定的实例感兴趣的运行上下文或target的名字。如果给了一个数组，那么任意给出targets上的任意监听器都会被通知。

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

最终，用作上下文或目标的名字比必要是类名字；它们可以是一些仅在你的应用中有意义的名字。例如，你可能有一系列响应“log”或“cache”的类，这些上的监听器会被它们中的任何一个通知到。

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

## 监听器聚合

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
这种情况，当然，在使用触发模型的时候搞出了一些歧义，如你不能够再确定最后一个结果是否符合你搜寻的标准。因此，我们建议将一种或别的实现标准化。


