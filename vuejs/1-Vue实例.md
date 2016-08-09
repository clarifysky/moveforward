# Vue实例

## 构造器

任何一个Vue.js应用都是从使用`Vue`构造器方法创建一个`根Vue实例`开始：

```js
var vm = new Vue({
  // options
})
```

一个Vue实例基本上是一个定义在[MVVM pattern](https://en.wikipedia.org/wiki/Model_View_ViewModel)中的**ViewModel**，因此你会在文档各处看到变量名`vm`。

当你要实例化一个Vue实例，你需要传入一个**选项对象**，它可能包含了数据，模版，要载入的元素，方法，生命周期回调选项等等。完整的选项列表可以在[API手册](http://vuejs.org/api)中找到。

Vue构造器可以被扩展来创建带有预定义选项的**组件构造器**：

```js
var MyComponent = Vue.extend({
  // extension options
})

// all instances of `MyComponent` are created with
// the pre-defined extension options
var myComponentInstance = new MyComponent()
```

尽管你可以创建一个扩展的实例，在多数情况下你将注册一个组件构造器作为自定义元素并在模版中声明式地组织它们。我们将在后面的细节中讨论组件系统。现在，你只需要知道所有的Vue.js组件都基本上是扩展自Vue实例。

## 属性和方法

每个Vue实例代理着所有在它的`data`对象中找到的属性：

```js
var data = { a: 1 }
var vm = new Vue({
  data: data
})

vm.a === data.a // -> true

// setting the property also affects original data
vm.a = 2
data.a // -> 2

// ... and vice-versa
data.a = 3
vm.a // -> 3
```

应当注意只有这些代理的属性是**回馈的**。如果你在实例被创建后附加一个新的属性给它，它不会触发任何视图更新。我们会在后面的细节中讨论这个反应系统。

除了数据属性，Vue实例揭露了许多有用的实例属性和方法，这些属性和方法以`$`作为前缀以和代理数据属性区分开。例如：

```js
var data = { a: 1 }
var vm = new Vue({
  el: '#example',
  data: data
})

vm.$data === data // -> true
vm.$el === document.getElementById('example') // -> true

// $watch is an instance method
vm.$watch('a', function (newVal, oldVal) {
  // this callback will be called when `vm.a` changes
})
```

查看[API手册(http://vuejs.org/api)来得到属性和实例的完整列表。

## 实例生命周期

每一个Vue实例都要经过一系列实例化步骤，例如，它需要设置数据检测，编译模版，创建必要的数据绑定。沿着这条路，它会调用一些**生命周期钩子**，这给了我们机会来执行自定义的逻辑。例如，`created`钩子会在实例被创建后调用：

```js
var vm = new Vue({
  data: {
    a: 1
  },
  created: function () {
    // `this` points to the vm instance
    console.log('a is: ' + this.a)
  }
})
// -> "a is: 1"
```

还会有别的钩子会在实例生命周期的不同阶段被调用，例如`compiled`，`ready`和`destroyed`。所有的生命周期钩子由指向调用它的Vue实例的`this`句柄调用。有些用户可能疑惑Vue.js世界中的"控制器"概念在哪，答案是：在Vue.js中，没有控制器，你的某个组件的自定义逻辑会被分成多个部分散布于这些生命周期钩子中。

## 生命周期图解

下面是一个实例的生命周期的图解，你现在不需要彻底理解运行中的每件事，但这个图解会在将来有用。
![lifecyle](lifecycle.png)
