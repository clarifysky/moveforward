# Tips

### 如何处理未编译完成时显示`{{}}`的问题？

见https://cn.vuejs.org/api/#v-cloak：

使用`v-cloak`指令结合`{ display: none }`样式可以使未编译完的`{{}}`标签保持隐藏直到编译完成。

示例：

```css
[v-cloak] {
  display: none;
}
```

```
<div v-cloak>
  {{ message }}
</div>
```

上面这个`<div>`不会显示，直到编译结束。

