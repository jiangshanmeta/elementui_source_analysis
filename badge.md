徽章/标签这一种设计元素一般是作为CSS组件来用，element-ui把它封装成了js组件，添加了一些小功能。

## 徽章类型

element-ui为badge这一个元素提供了两种设计，一种是显示文字带圆角的badge，另一种是显示小红点的badge，这两种类型的切换是通过```is-dot```这个布尔属性控制的。这里布尔属性的使用方法自己需要学习一下。


## 确定显示内容

当徽章内容为数字的时候，element-ui提供了一个feature用来设定显示最大值：

```javascript
content() {
  if (this.isDot) return;

  const value = this.value;
  const max = this.max;

  if (typeof value === 'number' && typeof max === 'number') {
    return max < value ? `${max}+` : value;
  }

  return value;
}
```

## 样式

对于显示文字的带圆角的badge，又可以根据是否有slot内容分为两种：当有slot内容时badge应该在右上角，当没有内容时badge按照inline形式展示。判断默认slot是否被填充的方法我们已经见多了：```$slots.default```，在button/radio/checkbox这些组件中我们都见过了这种用法，通过这种方式我们控制一个css类```is-fixed```。

把badge放在右上角不难实现，大体思路就是绝对定位定上去，element-ui在细节上做的不错：

```css
.el-badge__content.is-fixed {
    top: 0;
    right: 10px;
    position: absolute;
    -ms-transform: translateY(-50%) translateX(100%);
    transform: translateY(-50%) translateX(100%)
}
```

这样设置使.el-badge__content距离.el-badge右边框距离一定，.el-badge__content关于.el-badge上边框对齐。