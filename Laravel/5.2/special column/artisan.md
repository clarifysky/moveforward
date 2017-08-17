# artisan

https://laravel.com/docs/5.2/artisan

1、启动Laravel服务
	php artisan serve
2、数据迁移
	//创建数据迁移文件
	php artisan make:migration create_tasks_table --create=tasks
	//生成对应的数据表
	php artisan migrate
3、创建模型
	//创建模型文件
	php artisan make:model Task
4、创建验证模块，包括注册，登陆模版和指向验证控制器的路由。
	php artisan make:auth --views
5、创建控制器
	php artisan make:controller TaskController
	//在Admin目录下创建控制器
	php artisan make:controller Admin/AdminBaseController
6、创建Policy
	php artisan make:policy TaskPolicy
7、显示所有artisan命令
	php artisan list
8、查看帮助
	//查看命令migrate的帮助文档
	php artisan help migrate
9、编写命令
	//创建SendEmails命令的存根，这条命令会创建一个app/Console/Commands/SendEmails.php类文件
	php artisan make:console SendEmails
	//--command选项可以用来指定终端命令名称
	php artisan make:console SendEmails --command=emails:send
	//当你的命令创建后，你应当为那个类的signature和description属性赋值，它们会当你的命令在list命令下显示出来时显示
10、创建中间件
	php artisan make:middleware AgeMiddleware
	若想创建全局中间件，即所有HTTP请求都被该中间件过滤，则需要将该中间件加入app/Http/Kernel.php类中的$middleware属性中。
	在app/Http/Kernel.php的$routeMiddleware属性中可以指定特定路由的中间件。
11、创建服务供应者
	php artisan make:provider CatsProvider


