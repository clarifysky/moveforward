# quick reference.

## 移动优先

为确保正确的渲染和触摸缩放，将下面的**viewport meta tag**加入到`<head>`中：

```html
<meta name="viewport" content="width=device-width, initial-scale=1">
```

可以在移动设备上禁用缩放能力，不过不建议在每个站点上都这么搞，需要慎用。

```html
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">
```

## 文字和链接

bootstrap设置了一些基本的全局的外观，文字图形和链接样式，这些都可以在`scaffolding.less`中找到。

## Normalize.css

为提升跨浏览器渲染，使用了[Normalize.css](http://necolas.github.io/normalize.css/)。

## 容器

bootstrap提供了两种容器，由于`padding`和更多别的因素，容器是不能嵌套的。

一个响应式的固定宽度的容器。

```html
<div class="container">
  ...
</div>
```

完整宽度的容器，扩展到你的视窗的整个宽度。

```html
<div class="container-fluid">
  ...
</div>
```

## 栅格系统

bootstrap包含了一个响应式的，移动优先的流式栅格系统，它在设备或视窗尺寸增加时会适当地增加到12列。

## 介绍

栅格系统用来通过一系列的行和列来构建页面布局，这些行和列中可以容纳你的页面内容。

- 行必须放在`.container`(固定宽度)或者`.container-fluid`(完全宽度)容器中以便产生正确的对齐和边距。
- 使用行来创建列的水平组。
- 内容应当被放置在列中，只有列才能成为行的直接子级。
- 预定义的类，如`.row`和`.col-xs-4`可以用来快速生成栅格布局。Less mixins也可以用于许多语义化的布局。
- 列通过`padding`创建了槽沟（列内容间的间距）。`That padding is offset in rows for the first and last column via negative margin on .rows.`
- 反向的margin就是下面的例子外凸的原因，所以在栅格列里的内容被以非栅格内容的形式排成一列。
- 栅格列由你在可用的12列中指定的列数来创建。例如，三个等宽列应当是`.col-xs-4`。
- 如果在单行内放置了超过12个列，则每个额外列祖会被当作一个单元，转到下一行。
- 栅格类应用于宽度大于或等于拐点尺寸的设备，并且在较小的设备上重写栅格类。因此，给一个元素应用`.col-md-*`类不知会影响它在中型设备上的样式，还会影响它在大型设备上的样式，如果`.col-lg-*`没有出现的话。

## 栅格选项

|  | 极小(extra small)设备,<br>手机 (<768px) | 小设备,<br>平板(≥768px)|中型设备，<br>桌面电脑(≥992px) | 大型设备，<br>桌面电脑(≥1200px)|
| --- | --- | --- | --- | --- |
| 栅格行为 |  总是水平排列 | 开始是折叠的，<br>当大于上面的拐点<br>时变成水平排列  | 同左 | 同左 |
| 容器宽度 | 没有(自动) | 750px | 970px | 1170px |
| 类前缀 | `.col-xs-` | `.col-sm-` | `.col-md-` | `.col-lg-` |
| 列数 | 12 | 12 | 12 | 12 |
| 列宽 | 自动 | ~62px | ~81px | ~97px |
| 槽沟宽度 | 30px(列每边15px) | 同左 | 同左 | 同左 |
| 可嵌套 | 是 | 是 | 是 | 是 |
| 偏移 | 是 | 是 | 是 | 是 |
| 列排序 | 是 | 是 | 是 | 是 |

## 响应式列重置

借助那四个东西，你可能会遇到一个问题，在确定的拐点，由于一个比别的高，你的列不会清除右侧，要解决这个问题，使用一个`.clearfix`和我们的[响应式组件类](http://getbootstrap.com/css/#responsive-utilities)的集合来解决这个问题。

```html
<div class="row">
  <div class="col-xs-6 col-sm-3">.col-xs-6 .col-sm-3</div>
  <div class="col-xs-6 col-sm-3">.col-xs-6 .col-sm-3</div>

  <!-- Add the extra clearfix for only the required viewport -->
  <div class="clearfix visible-xs-block"></div>

  <div class="col-xs-6 col-sm-3">.col-xs-6 .col-sm-3</div>
  <div class="col-xs-6 col-sm-3">.col-xs-6 .col-sm-3</div>
</div>
```

## 偏移列

使用`.col-md-offset-*`类将列向右偏移。这些类会增加一个列左边的间距，间距值为`*`指定的列数。

## 嵌套列

你需要在一个新`.row`里面开启嵌套列。如：

```html
<div class="row">
  <div class="col-sm-9">
    Level 1: .col-sm-9
    <div class="row">
      <div class="col-xs-8 col-sm-6">
        Level 2: .col-xs-8 .col-sm-6
      </div>
      <div class="col-xs-4 col-sm-6">
        Level 2: .col-xs-4 .col-sm-6
      </div>
    </div>
  </div>
</div>
```

## 列排序

使用我们内置的栅格系统的`.col-md-push-*`和`.col-md-pull-*`修饰类可以轻松地更改列的顺序。

| .col-md-3 .col-md-pull-9 | .col-md-9 .col-md-push-3 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; |
| --- | --- |


```
<div class="row">
  <div class="col-md-9 col-md-push-3">.col-md-9 .col-md-push-3</div>
  <div class="col-md-3 col-md-pull-9">.col-md-3 .col-md-pull-9</div>
</div>
```

> 猜想：`push`是向右推，`pull`是向左拉，两个数值需要匹配。

## 文字图形

### 头标签

所有的HTML头标签，`<h1>`到`<h6>`都可以用。`.h1`到`.h6`类也可以用。

在任意heading中要创建更轻小的，次级文字，使用`<small>`标签或`.small`类。

```html
<h1>h1. Bootstrap heading <small>Secondary text</small></h1>
<h2>h2. Bootstrap heading <small>Secondary text</small></h2>
<h3>h3. Bootstrap heading <small>Secondary text</small></h3>
<h4>h4. Bootstrap heading <small>Secondary text</small></h4>
<h5>h5. Bootstrap heading <small>Secondary text</small></h5>
<h6>h6. Bootstrap heading <small>Secondary text</small></h6>
```

## body拷贝

全局默认的`font-size`是14px，`line-height`是1.428，这被应用在`<body>`和所有的段落上，除此之外,`<p>`接收一个底部边界，边界值是它们计算出的行高的一般（默认10px）。

### body突出

要使一个段落突出出来，加上`.lead`类。

```html
<p class="lead">...</p>
```

## 成行文字元素

### 高亮文本

```html
You can use the mark tag to <mark>highlight</mark> text.
```

### deleted 文本 (文本被删除了)

```html
This line of text is meant to be treated as deleted text.
```

### 划线文本 (文本不再使用)

```html
<s>This line of text is meant to be treated as no longer accurate.</s>
```

### 插入的文本

`<ins>`

### 下划线文本

`<u>`

### 小文本

`<small>`或`.small`

### 粗体

`<strong>`

### 斜体

`<em>`

> 备选元素，可以随意在HTML5中使用`<b>`和`<i>`。`<b>`意味着高亮，但不表达重要性，`<i>`大多用于声音，技术条目等等。

## 对齐类

- `text-left`
- `text-center`
- `text-right`
- `text-justify`
- `text-nowrap`

## 变形类

- `text-lowercase`
- `text-uppercase`
- `text-capitalize`

## 短语

`<abbr>`用于短语和首字母缩略词，在鼠标悬停的时候显示展开的版本，带有`title`属性的`<abbr>`标签在鼠标悬停的时候会出现一个问号，并在鼠标悬停位置提供一个辅助信息提示框。类似于操作系统中的鼠标悬停提示。

```html
<abbr title="attribute">attr</abbr>
```
给短语上添加`.initialism`类来产生一个更轻更小的字体。

```html
<abbr title="HyperText Markup Language" class="initialism">HTML</abbr>
```

## 地址

`<address>`

## 块引用

### 默认的块引用

`<blockquote>`

### 块引用选项

块引用内可以使用下面的标签

- `<footer>`
- `<cite>`

`.blockquote-reverse`会以和块引用相反的对齐方式来显示

## 列表

### 无序列表

```html
<ul>
  <li>...</li>
</ul>
```

### 有序列表

```
<ol>
  <li>...</li>
</ol>
```

### 无样式

使用`.list-unstyled`来移除默认的`list-style`样式，这个支队长直接下级有效。

```html
<ul class="list-unstyled">
  <li>...</li>
</ul>
```

### 成行

```html
<ul class="list-inline">
  <li>...</li>
</ul>
```

### 描述

带有相关描述的条款列表。

```html
<dl>
  <dt>...</dt>
  <dd>...</dd>
</dl>
```

#### 水平描述

```html
<dl class="dl-horizontal">
  <dt>...</dt>
  <dd>...</dd>
</dl>
```

> 自动截断
> 水平描述列表可以使用`text-overflow`来截断超长的条款。在狭窄的视窗内，这个会改变默认的堆叠布局








