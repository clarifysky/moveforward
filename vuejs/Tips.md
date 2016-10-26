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

### 生HTML

双大括号语法会将数据展示位平白的文本，要输出为真正的HTML，你需要使用`v-html`指令：

```html
<div v-html="rawHtml"></div>
```

内容会被当作平白HTML插入，数据绑定会被忽略。注意，你不能使用`v-html`来组织模版片段，Vue不是基于字符串的模版引擎。

