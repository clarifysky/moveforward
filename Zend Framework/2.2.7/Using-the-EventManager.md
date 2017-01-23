# 使用事件管理器

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


