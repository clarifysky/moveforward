# 构建第一个Angular 2 应用

> 翻译自：https://angular.io/docs/ts/latest/quickstart.html

> translated from here: https://angular.io/docs/ts/latest/quickstart.html

> 这是TypeScript版本。｜This is the TypeScript version.

- 前置条件：安装Node.js。
- 步骤1：创建一个应用目录，定义包依赖和额外的设置。
- 步骤2：创建应用的根组件。
- 步骤3：添加`main.ts`，使Angular能够辨别出根组件。
- 步骤4：添加`index.html`，这个页面用来承载这个应用。
- 步骤5：构建并运行应用。

## 前置条件：Node.js
如果你还没有安装[Node.js® 和 npm](https://nodejs.org/en/download/)，那么请安装它。

> 请确保你正在运行最新的node`v4.x.x`和npm`3.x.x`。

## 步骤1：创建和配置项目
在这个步骤中，我们将：

1. 创建项目文件夹
1. 添加包定义和配置文件
1. 安装包

### 创建项目文件夹

```js
mkdir angular2-quickstart
cd    angular2-quickstart
```

### 添加包定义和配置文件

添加下面的包定义和配置文件到项目文件夹中：

- `package.json`列出了QuickStart应用所依赖的包并定义了一些有用的脚本。查看[Npm包配置][5]来获取细节。
- `tsconfig.json`是TypeScript编译器的配置文件，[查看TypeScript配置][6]来获取细节。
- `typings.json`标识了TypeScript定义文件。查看[TypeScript配置][7]来获取细节。
- `systemjs.config.js`，是SystemJS配置文件。查看[下面][8]的讨论。

> 你可以在本翻译的当前目录下找到上面列出的那几个文件。

### 安装包

```js
npm install
```

> `typings`目录可能在`npm install`执行完之后没有出现，如果是这样，你需要手动安装他们：

> ```js
npm run typings install
> ```


> **有用的脚本**

> 在我们建议的`package.json`中包含了一些npm脚本来帮助处理常见的开发任务：

> ```js
{
  "scripts": {
    "start": "tsc && concurrently \"npm run tsc:w\" \"npm run lite\" ",
    "lite": "lite-server",
    "postinstall": "typings install",
    "tsc": "tsc",
    "tsc:w": "tsc -w",
    "typings": "typings"
  }
}
> ```

> 我们通常按照如下方式执行npm脚本：`npm run`后接脚本名。

> 这个脚本这要有以下用途：

> - `npm start` - 同时运行编译器和服务，同时处于"监视状态"。
- `npm run tsc` - 运行一次TypeScript编译器。
- `npm run tsc:w` - 以监视模式运行TypeScript编译器，这个进程会持续运行，等到发现TypeScript文件发生变化后会重新编译它们。
- `npm run lite` - 运行
[lite-server][1]，一个轻量的，静态文件服务器，对于使用路由的Angular应用具有优秀的支持。
- `npm run typings` - 单独运行[typings工具][2]。
- `npm run postinstall` - 在npm成功完成包安装后，由它自动调用。这个脚本安装在`typings.json`中定义的[TypeScript定义文件][3]。

## 步骤2：我们的第一个Angular组件
我们需要创建一个文件夹来存放应用程序，然后添加一个超简单的Angular组件。

创建一个项目根目录下的app子目录：

```js
mkdir app
```

创建一个内容如下的组件文件`app/app.component.ts`（在新创建的目录中）：

```js
import { Component } from '@angular/core';
@Component({
  selector: 'my-app',
  template: '<h1>My First Angular 2 App</h1>'
})
export class AppComponent { }
```

> **AppComponent是应用的根**

> 每一个Angular应用至少具有一个**根组件**，一般命名为`AppComponent`，它负责处理客户端的用户体验。组件是Angular应用的几本构成单元，组件通过与它关联的模版控制着屏幕的一部分——一个视图。

> 这个QuickStart只有一个组件，但它具有我们要编写的每个组件都需要的基本结构：

> - 一个或多个import声明来关联我们需要的东西。
- 一个@Component修饰符来告诉Angular我们要使用什么模版和如何创建组件。
- 一个组件类，通过它的模版来控制一个视图的行为和外观。

> **导入**

> Angular应用是模块化的，它们由许多文件组成，每个文件表达了一个目的。Angular自身也是模块化的，它是一个类库模块集合，其中的每个由多个相关的特性组成。

> 当我们需要一个模块或类库中的东西，就要导入它。

> ```js
import { Component } from '@angular/core';
```
> **@Component修饰符**

> `Component`是一个修饰符方法，它接收一个元数据对象作为参数，我们通过在方法前面加上`@`符号来将它应用给组件类，使用元数据对象激活它。（它需要放在组建类上方？）

> `@Component`是一个修饰符，它允许我们将元数据关联到组建类上。这个元数据告诉了Angular如何创建和使用组件。

> ```js
@Component({
  selector: 'my-app',
  template: '<h1>My First Angular 2 App</h1>'
})
```

> **组件类**

> 下面是一个空的，什么也不做的名为`AppComponent`的类。
> ```js
export class AppComponent { }
```

> **导出**`AppComponent`可以使我们在应用的其他地方**导入**它。

## 步骤3:添加main.ts

现在需要告诉Angular来载入根组件，创建包含下列信息的`app/main.ts`文件：
```js
import { bootstrap }    from '@angular/platform-browser-dynamic';
import { AppComponent } from './app.component';
bootstrap(AppComponent);
```

> 我们导入了两个我们需要来启动应用的东西：

> 1. Angular的浏览器`bootstrap`方法
2. 应用的根组件，`AppComponent`

> 然后调用`bootstrap`，传以`AppComponent`

## 步骤4:添加index.html
在项目跟目录，创建`index.html`文件，它的内容如下：
```js
<html>
  <head>
    <title>Angular 2 QuickStart</title>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="stylesheet" href="styles.css">
    <!-- 1. Load libraries -->
     <!-- Polyfill(s) for older browsers -->
    <script src="node_modules/core-js/client/shim.min.js"></script>
    <script src="node_modules/zone.js/dist/zone.js"></script>
    <script src="node_modules/reflect-metadata/Reflect.js"></script>
    <script src="node_modules/systemjs/dist/system.src.js"></script>
    <!-- 2. Configure SystemJS -->
    <script src="systemjs.config.js"></script>
    <script>
      System.import('app').catch(function(err){ console.error(err); });
    </script>
  </head>
  <!-- 3. Display the application -->
  <body>
    <my-app>Loading...</my-app>
  </body>
</html>
```

> 值得一提的HTML区域为：
> 1. JavaScript[类库][4]。
1. SystemJS的配置文件，还有一个用来导入并运行指向刚刚创建的`main`文件的`app`模块的脚本。
1. `<body>`区域中的`<my-app>`标签，它是应用生存的地方。

## 步骤5:构建和运行应用
打开命令终端，执行以下命令：

```js
npm start
```


[1]:https://www.npmjs.com/package/lite-server
[2]:https://angular.io/docs/ts/latest/guide/typescript-configuration.html#!#typings
[3]:https://angular.io/docs/ts/latest/guide/typescript-configuration.html#!#typings
[4]:https://angular.io/docs/ts/latest/quickstart.html#libraries
[5]:https://angular.io/docs/ts/latest/guide/npm-packages.html
[6]:https://angular.io/docs/ts/latest/guide/typescript-configuration.html#tsconfig
[7]:https://angular.io/docs/ts/latest/guide/typescript-configuration.html#!#typings
[8]:https://angular.io/docs/ts/latest/quickstart.html#systemjs
