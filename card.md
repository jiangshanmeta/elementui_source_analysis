card这个元素有时也被称之为panel(在bootstrap3是被称为panel，但是在bootstrap4改成了card)。

card元素一般分为header、body和footer三个部分，虽然element-ui将其封成了组件，但我并不是很认同这种做法，我觉得对于card组件应该只提供其基础CSS部分，然后这样使用：

```html
<div class="card">
    <div class="card-header"></div>
    <div class="card-body"></div>
    <div class="card-footer"></div>
</div>
```

批判完了然后看下element-ui对应的实现：

```html
<template>
  <div class="el-card">
    <div class="el-card__header" v-if="$slots.header || header">
      <slot name="header">{{ header }}</slot>
    </div>
    <div class="el-card__body" :style="bodyStyle">
      <slot></slot>
    </div>
  </div>
</template>
```

element-ui的card只有header和body两部分，没有footer的设计，而且```bodyStyle```这个props属性的存在也暴露了一个问题，就是封装成组件之后修改样式可能没那么简单。