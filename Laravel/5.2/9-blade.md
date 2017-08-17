# blade
## 介绍
Blade是由Laravel提供的简单又给力的模版引擎。不像其他流行的PHP模版引擎，Blade不限制你在你的视图中使用纯PHP代码。所有的Blade视图都被编译成PHP代码并且缓存起来，直到它们被修改，意味着Blade在本质上给你的应用添加了0开销。Blade视图文件使用.blade.php文件后缀，一般存储在resources/views目录下。
模版继承
扩展布局
使用Blade @extends符号来指定子页面需要继承的布局。@extends了Blade布局的视图会使用@section符号将内容注入到布局的section中。
使用@parent符号来给布局的区域中加入内容，而不是替换，@parent符号会在视图被渲染时被布局的内容替换。
显示数据
你可以将变量包含在大括号内传递给你的Blade视图。例如，给定下面的路由：
Route::get('greeting', function () {    return view('welcome', ['name' => 'Samantha']);});
然后就可以在视图中这么显示$name变量：{{ $name }}
也可以直接在Blade echo声明中执行任意PHP代码：The current UNIX timestamp is {{ time() }}.
Blade {{}}声明自动由PHP的htmlentities方法发送来防止XSS攻击。
资源载入
资源需要放在public目录下：
<link href="{{ URL::asset('ckeditor/samples/toolbarconfigurator/lib/codemirror/neo.css') }}" rel='stylesheet' type='text/css'>
Blade和JavaScript框架
目前也有许多JavaScript框架使用双花括号来显示内容（{{}}），你可以使用@符号来通知Blade渲染引擎一个表达式需要被保留，例如：
Hello, @{{ name }};
这个时候，Blade会移除@符号，但是{{ name }}表达式会被保留，允许它被你的JavaScript框架渲染。

