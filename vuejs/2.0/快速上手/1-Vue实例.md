# Vue实例

## Vue实例创建语法

```js
var app5 = new Vue({
  el: '#app-5',
  data: {
    message: 'Hello Vue.js!',
    total: 0
  },
  methods: {
    reverseMessage: function () {
      this.message = this.message.split('').reverse().join('')
    }
  },
  computed: {
    total: function () {
        this.total++;
    }
  }
})
```

## 属性和方法

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
```

`Vue`的一些实例属性：

- `$data`访问数据。
- `$el`访问el对象。

## 实例生命周期钩子

`created`在实例被创建后调用,其他的还有`mounted`,`updated`,`destroyed`。



