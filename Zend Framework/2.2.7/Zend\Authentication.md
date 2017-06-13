# Zend\Authentication

https://framework.zend.com/manual/2.2/en/modules/zend.authentication.intro.html

`Zend\Authentication`只关注于**authentication**而不包括**authorization**。

> 没有`Zend\Authentication\Authentication`类，取而代之的，有个`Zend\Authentication\AuthenticationService`。这个类使用潜在的认证适配器和后台持久存储。

## 适配器

不同的认证有不同的适配器，但在这些认证适配器中，有些基本的东西是通用的。例如，接收认证凭证，实施基于认证服务的查询，返回对于`Zend\Authentication`适配器常见的结果。

每个`Zend\Authentication`适配器类都要实现`Zend\Authentication\Adapter\AdapterInterface`接口。这个接口定义了一个适配器类在实施认证查询时，必须实现的方法`authenticate()`。

`authenticate()`必须返回`Zend\Authentication\Result`的实例。如果出现了一些情况导致不可能实施认证查询，则`authenticate()`应当抛出一个`Zend\Authentication\Adapter\Exception\ExceptionInterface`异常。

## 结果

`Zend\Authentication`适配器通过`authenticate()`方法返回一个`Zend\Authentication\Result`来表明一个认证尝试结果。

下面的4个方法是针对`Zend\Authentication`适配器的结果的一些基本操作：

- `isValid()` 当且仅当结果表现出一个成功的认证尝试时返回true。
- `getCode()` 返回一个`Zend\Authentication\Result`常量识别符用来确定失败类型或是否成功了。
- `getIdentity()` 返回认证尝试的身份。
- `getMessages()` 返回一个基于失败的认证尝试的消息数组。

## 身份持久化

*HTTP*是无状态的协议，有必要支持维护认证过的身份而不是在每次请求的时候都提供认证凭证。

### 默认的在PHP Session中的持久化

完成一次成功的认证尝试后，`Zend\Authentication\AuthenticationService::authenticate()`会将认证结果中的身份存储到持久存储中。除非指定了别的方式。`Zend\Authentication\AuthenticationService`使用的是`Zend\Authentication\Storage\Session`，它进而使用`Zend\Session`。通过提供一个实现了`Zend\Authentication\Storage\StorageInterface`给`Zend\Authentication\AuthenticationService::setStorage()`的方式，也可以自定义这个存储的类。

### 链接存储

一个站点可以适当地有许多存储。`Chain`存储可以将这些粘合起来。

### 实现自定义的存储

要实现自定义的存储，开发者需要实现`Zend\Authentication\Storage\StoragetInterface`并提供一个这个类的实例给`Zend\Authentication\AuthenticationService::setStorage()`。

要使用一个不是`Zend\Authentication\Storage\Session`的身份持久存储类，开发者要实现`Zend\Authentication\Storage\StorageInterface`。

要使用这个自定义的存储类，需要在认证查询发起前调用`Zend\Authentication\AuthenticationService::setStorage()`

```php
use Zend\Authentication\AuthenticationService;

// Instruct AuthenticationService to use the custom storage class
$auth = new AuthenticationService();

$auth->setStorage(new My\Storage());

/**
 * @todo Set up the auth adapter, $authAdapter
 */

// Authenticate, saving the result, and persisting the identity on
// success
$result = $auth->authenticate($authAdapter);
```

## 用法

有两种使用`Zend\Authentication`适配器的方法：

- 不直接的，通过`Zend\Authentication\AuthenticationService::authenticate()`
- 直接的，通过*适配器*的`authenticate()`方法

要获取已经存储了的身份，

```php
use Zend\Authentication\AuthenticationService;

$auth = new AuthenticationService();

/**
 * @todo Set up the auth adapter, $authAdapter
 */

if ($auth->hasIdentity()) {
    // Identity exists; get it
    $identity = $auth->getIdentity();
}
```

在退出系统时清理身份，

```php
$auth->clearIdentity();
```



