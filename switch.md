switch效果其实移动端见得比较多，element-ui本身是针对pc设计的，感觉有点奇怪。不过从代码角度来看这个组件实现的很优雅。

首先要讲的是模板部分，之前我讲其它组件的时候主要是关注model层，尤其是关注model层的数据流动，view层只是捎带一提，但是switch的模板实现的太优雅了。整体是一个label标签，label 包裹着一个checkbox，这样点击这个组件任何一部分都会改变checkbox的状态。```el-switch__core```是这个组件的主要展示区，它负责switch不同状态下不同的背景颜色，其下的```el-switch__button```对应通过位置反映状态的圆球。两个```el-switch__label```对应不同状态下的文字区域。

然后就是重点数据流动了。我们首先自上而下看：

```javascript
created() {
    if (!~[this.onValue, this.offValue].indexOf(this.value)) {
        this.$emit('input', this.offValue);
    }
},
```

父组件通过value属性传递值，这个值会被校验是否是```onValue```和```offValue```中的一个，校验失败则默认采用```offValue```。注意这里特地采用了位运算符```~```，因为indexOf方法如果找不到值则返回-1，-1经过按位非运算会变成0，似乎这样比判断是否等于-1要有点性能上的优势，这里我其实更想用includes方法。

拿到父元素的值之后我们就可以判断当前的选中状态了：

```javascript
checked() {
    return this.value === this.onValue;
},
```

通过判断当前的选中状态，我们可以设定checkbox的状态、控制背景颜色以及控制小球的位置：

```javascript
watch: {
    checked() {
        this.$refs.input.checked = this.checked;
        if (this.onColor || this.offColor) {
            this.setBackgroundColor();
        }
    }
},
transform() {
    return this.checked ? `translate(${ this.coreWidth - 20 }px, 2px)` : 'translate(2px, 2px)';
}
```

然后我们自下而上看：当我们点击组件任何一部分界面时，由于label标签的特性checkbox会反转选中状态，然后触发```change```事件。当监听到```change```事件事件的时候我们通知父组件：

```javascript
handleChange(event) {
    this.$emit('input', !this.checked ? this.onValue : this.offValue);
    this.$emit('change', !this.checked ? this.onValue : this.offValue);
    this.$nextTick(() => {
        this.$refs.input.checked = this.checked;
    });
},
```

当父组件接受switch组件传递过来的值的时候，会改变value值，从而改变switch的选中状态。