# The Root Directory
app：包含你的应用的核心代码
	Console/Http：提供进入应用核心的API。HTTP协议和CLI（Command Line Interface）都是和你的应用交互的装置，但它们实际上不包含任何应用逻辑。换句话说，它们只是两种发送命令给应用的方式。Console目录包含所有的Artisan命令，Http目录包含控制器，中间件，和请求。
	Events：安置着事件类，比如当某个动作被触发时警告应用的其它部分。
	Exceptions：包含应用异常处理器，同时也是存储应用抛出的异常的好选择。
	Jobs：存储队列化的工作。这些工作可能由应用加入队列或者在当前请求周期内同步执行。
	Listeners：存储事件的处理类，这些处理类响应正在处理的事件：接收事件，响应逻辑。
	Policies：包含认证政策类。这些政策用来检查一个用户是否可以针对某个资源执行一个给定的动作。
bootstrap：包含启动框架，配置自动加载的一些文件，cache文件夹中也包含一些框架生成的用于优化启动性能的文件。
config：包含所有的你的应用的配置文件。
database：包含你的数据库迁移和种子，如果你愿意，你可以使用这个目录来保存SQLite数据库。
public：前台控制器和资源集（图片，JavaScript，CSS等）。
resources：包含视图，原始资源（LESS, SASS, CoffeeScript）和本地化文件。
storage：包含已编译的Blade模版，基于session的文件，缓存文件和一些由框架生成的其它文件。这个目录被分成了app，framework，logs这些目录。
	app：存储任何可能被应用用到的文件
	framework：存储由框架生成的文件和缓存
	logs：存储应用的日志文件
tests：存储自动化测试
vendor：包含Composer依赖项。

