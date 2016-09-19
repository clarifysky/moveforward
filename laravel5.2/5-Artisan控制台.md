# 介绍

Artisan是Laravel里面的命令行接口的名字。它在你开发应用的以后提供了一些有用的命令。它由强力的Symfony Console组件驱动。要查看可用的Artisan命令列表，你可以使用`list`命令。

```
php artisan list
```

每一条命令都包含一个“帮助”屏幕，它会显示并描述一些可用的参数和选项，要查看帮助屏幕，只需要简单地在命令前面加上`help`就可以：

```
php artisan help migrate
```

# 编写命令

除了Artisan提供的命令之外，你还可以创建你自己的自定义命令。你可以把你的自定义命令存储在`app/Console/Commands`目录中，然而，你可以随意决定你自己的存储位置，因为你的命令都可以自动加载，基于你的`composer.json`设置。

要创建一条新命令，你可以使用`make:console`Artisan 命令，它会生成一条命令存根来帮助你开始：

```
php artisan make:console SendEmails
```

上面的命令会生成一个类`app/Console/Commands/SendEmails.php`。当创建这个命令的时候，`--command`选项可用于指派终端命令名称：

```
php artisan make:console SendEmails --command=emails:send
```

## 命令结构

一旦你的命令生成好了，你应当填充那个类的`signature`和`description`属性，这些将会在把你的命令显示到`list`屏幕上时用到。

`handle`方法会在你的命令被执行时调用。你应当将任何的命令逻辑放在这个方法内。让我们看一个例子。

注意，我们可以注入任何需要的依赖到命令的构造器内。Laravel的服务容器会自动注入人和类型提示的依赖到构造器中。为了最大化代码重用，保持你的控制台命令轻量并使它们延迟到应用服务中来完成它们的任务是一个好的练习。

```php
<?php

namespace App\Console\Commands;

use App\User;
use App\DripEmailer;
use Illuminate\Console\Command;

class SendEmails extends Command
{
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user}';

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = 'Send drip e-mails to a user';

    /**
     * The drip e-mail service.
     *
     * @var DripEmailer
     */
    protected $drip;

    /**
     * Create a new command instance.
     *
     * @param  DripEmailer  $drip
     * @return void
     */
    public function __construct(DripEmailer $drip)
    {
        parent::__construct();

        $this->drip = $drip;
    }

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $this->drip->send(User::find($this->argument('user')));
    }
}
```

# 命令I/O

## 定义输入预期

当编写控制台命令时，通过参数或选项来聚集输入是很常见的事。Laravel在你的命令上使用`signature`属性来很方便地定义你希望从用户那得到的输入。`signature`允许你为命令定义名字，参数还有选项，以一种单一，富于表现力，路由类似的语法形式。

所有的用户提供的参数和选项都被包裹在大括号内。在下面的例子中，那个命令定义了一个**必须**的参数：`user`：

```php
/**
 * The name and signature of the console command.
 *
 * @var string
 */
protected $signature = 'email:send {user}';
```

你也可以让参数变成可选，并为其指定默认值：

```
// Optional argument...
email:send {user?}

// Optional argument with default value...
email:send {user=foo}
```

选项，和参数一样，也是一种用户输入形式。然而当它们被在命令行指定时，它们前面带有两个破折号（--）。我们可以在签名中这样定义选项：

```php
/**
 * The name and signature of the console command.
 *
 * @var string
 */
protected $signature = 'email:send {user} {--queue}';
```

在这个例子中，`--queue`开关可能会在调用Artisan命令时被指定。如果`--queue`开关通过了，选项的值就是`true`，否则，就是`false`:

```
php artisan email:send 1 --queue
```

你也可以指定用户必须给选项指派一个值，通过在它后面缀上`=`符号来表明必须提供一个值：

```php
/**
 * The name and signature of the console command.
 *
 * @var string
 */
protected $signature = 'email:send {user} {--queue=}';
```

在这个例子中，用户可能像这样给选项传递一个值：

```
php artisan email:send 1 --queue=default
```

你也可以为选项指派默认值：

```
email:send {user} {--queue=default}
```

要在定义选项的时候给它指定缩略名，你可以在选项前面指定缩略名，并在缩略名和选项之间用|分隔符将它俩分开：

```
email:send {user} {--Q|queue}
```

如果你希望参数或选项接收数组输入，你可以使用`*`符号:

```
email:send {user*}

email:send {user} {--id=*}
```

### 输入描述

你可以为通过用冒号将描述和参数/选项隔开的方式来给输入参数和选项指定描述信息：

```php
/**
 * The name and signature of the console command.
 *
 * @var string
 */
protected $signature = 'email:send
                        {user : The ID of the user}
                        {--queue= : Whether the job should be queued}';
```

### 拉取输入

当你的命令被执行时，很显然你需要访问参数和选项的值，所以，你需要使用`argument`和`option`方法：

```php
/**
 * Execute the console command.
 *
 * @return mixed
 */
public function handle()
{
    $userId = $this->argument('user');

    //
}
```

如果你需要获取所有的参数作为数组，调用`argument`而不要带参数:

```php
$arguments = $this->argument();
```

获取选项和获取参数一样简单，需要使用`option`方法。如同`argument`方法，你可以不带参数调用`option`方法来获取所有的选项作为一个数组：

```php
// Retrieve a specific option...
$queueName = $this->option('queue');

// Retrieve all options...
$options = $this->option();
```

如果参数或选项不存在，`null`会被返回。

## 提示输入

除了显示输出，你还有可能要用户在你的命令执行期间提供输入。`ask`方法会带着给定的问题提示用户，接收他们的输入，并将用户的输入返回给你的命令。

```php
/**
 * Execute the console command.
 *
 * @return mixed
 */
public function handle()
{
    $name = $this->ask('What is your name?');
}
```

`secret`方法和`ask`类似，但是它们的输入不会显示在控制台上。这个方法在向用户征求敏感信息（如密码）时非常有用：

```php
$password = $this->secret('What is the password?');
```

### 征求确认

如果你需要用户进行一个简单的确认，你可以使用`cnofirm`方法，默认情况下，这个方法会返回`false`，然而，如果用户输入了`y`作为提示响应，这个方法就会返回`true`。

```php
if ($this->confirm('Do you wish to continue? [y|N]')) {
    //
}
```

### 给用户一个主意

`anticipate`方法可以用来给可能的主意提供自动完成。用户仍旧可以选择别的答案，不管自动完成提示：

```php
$name = $this->anticipate('What is your name?', ['Taylor', 'Dayle']);
```

如果你要给用户一个预设的主意集合，你可以使用`choice`方法，用户选择答案的索引，但是答案的值会被返回给你。你也可以设置默认值作为返回，当用户没有做出选择的时候：

$name = $this->choice('What is your name?', ['Taylor', 'Dayle'], $default);

## 编写输出


要发送输出到控制台，使用`line`, `info`, `comment`, `question`和`error`方法。这里面的每一个方法都会使用合适的ANSI颜色来表达它们的意图。

要给用户显示一个信息，使用`info`方法。通常，这会在控制台上显示为绿色的文本：

```php
/**
 * Execute the console command.
 *
 * @return mixed
 */
public function handle()
{
    $this->info('Display this on the screen');
}
```

要显示一条错误信息，使用`error`方法，错误信息通常显示为红色文本：

```php
$this->error('Something went wrong!');
```

如果你要显示平白的控制台输出，使用`line`方法，`line`方法不会接收任何唯一的自然色彩：

```php
$this->line('Display this on the screen');
```

### 表格布局

`table`方法使得正确地格式化多行/多列数据变得容易。仅仅给这个方法传入头部和行就可以了。宽度和高度会基于给定的数据动态计算：

```php
$headers = ['Name', 'Email'];

$users = App\User::all(['name', 'email'])->toArray();

$this->table($headers, $users);
```

### 进度条

对于长时间执行的任务，显示一个进度指示器是很有用的。使用输出对象，我们可以开始，前进和停止进度条。你必须在开始进度的时候定义步数，然后在每步之后向前推进进度条。

```php
$users = App\User::all();

$bar = $this->output->createProgressBar(count($users));

foreach ($users as $user) {
    $this->performTask($user);

    $bar->advance();
}

$bar->finish();
```

关于更多的选项，查看[Symfony进度条组件文档](http://symfony.com/doc/2.7/components/console/helpers/progressbar.html)

# 注册命令

一旦你的命令完成了，你需要将它和Artisan注册到一起，以便于可以使用。这个在`app/Console/Kernel.php`文件中完成了。

在这个文件中，你会在`commands`属性中找到一个命令列表。要注册你的命令，简单地将命令类名称加入进去。当Artisan启动时，所有在这里面列出的命令都会被服务容器解决并和Artisan注册在一起：

```php
protected $commands = [
    Commands\SendEmails::class
];
```

# 通过代码调用命令

有时你可能希望在CLI之外执行一个Artisan命令。例如，你也许希望从一个路由或控制器处触发一条Artisan命令。你可以在`Artisan`门面上调用`call`方法来完成这个。`call`方法接收命令的名字作为第一个参数，还有一个命令参数数组作为第二个参数。退出代码会被返回：

```php
Route::get('/foo', function () {
    $exitCode = Artisan::call('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);

    //
});
```

在`Artisan`门面上使用`queue`方法，通过你的[队列workers](https://laravel.com/docs/5.2/queues)，你甚至可以将Artisan命令加入队列以便于它们可以在后台执行：

```php
Route::get('/foo', function () {
    Artisan::queue('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);

    //
});
```

如果你需要给一个不接受字符串值的选项指定值，例如`migrate:refresh`命令上的`--force`标识，你可以传递一个布尔的`true`或`false`:

```php
$exitCode = Artisan::call('migrate:refresh', [
    '--force' => true,
]);
```

## 从其它命令调用命令

有时，你可能希望从一个现存的Artisan命令中调用其它命令。你可以用`call`方法做这个。这个`call`接收命令名和一个命令参数数组：

```php
/**
 * Execute the console command.
 *
 * @return mixed
 */
public function handle()
{
    $this->call('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);

    //
}
```

如果你想要调用别的命令并压制它的所有输出，你可以使用`callSilent`方法，`callSilent`方法和`call`方法具有相同的签名：

```php
$this->callSilent('email:send', [
    'user' => 1, '--queue' => 'default'
]);
```

