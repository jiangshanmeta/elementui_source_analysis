关于单选框，element-ui最出彩的地方并不是对其美化工作，而是radio-group这个概念上。这样就牵扯到了radio和radio-group之间通信问题了。在阅读了element-ui中grid相关代码之后，我们可以猜测到会使用到```$parent```这个属性。

## 判断是否在radio-group中

element-ui的radio可以单独使用，也可以配合radio-group使用(虽然实践中我基本上都是配合radio-group使用)，于是我们就面临这么一个问题：如何判断radio是否在radio-group中，解决方案和在element-ui的grid中判断col是否在row中是一致的，都是沿着组件树向上查找：

```javascript
isGroup() {
  let parent = this.$parent;
  while (parent) {
    if (parent.$options.componentName !== 'ElRadioGroup') {
      parent = parent.$parent;
    } else {
      // 保存最近的radio-group组件实例
      this._radioGroup = parent;
      return true;
    }
  }
  return false;
},
```

## 绑定值

我以前封装表单元素组件是这么做的：首先data中有一个currentValue属性，其初始值是通过props传递下来的value，然后我通过watch监听value值的变化，在回调中对currentValue重新赋值，表单元素的值绑定到currentValue上，我还要watch currentValue，在回调中```$emit```一个input事件，方便外部通过```v-model```语法使用。

element-ui的实现比我高明多了：

```javascript
model: {
  get() {
    return this.isGroup ? this._radioGroup.value : this.value;
  },

  set(val) {
    if (this.isGroup) {
      this.dispatch('ElRadioGroup', 'input', [val]);
    } else {
      this.$emit('input', val);
    }
  }
},

```

这里利用了 [计算setter](https://cn.vuejs.org/v2/guide/computed.html#计算-setter)这么一个特性，通过这一个get/set处理省掉了中间的currentValue。


还有一个问题是radio单独使用时自身```$emit```一个input事件，当包在radio-group时需要通过radio-group```$emit```一个input事件。对于后者element-ui特别封装了一个dispatch方法：

```javascript
dispatch(componentName, eventName, params) {
  var parent = this.$parent || this.$root;
  var name = parent.$options.componentName;

  while (parent && (!name || name !== componentName)) {
    parent = parent.$parent;

    if (parent) {
      name = parent.$options.componentName;
    }
  }
  if (parent) {
    parent.$emit.apply(parent, [eventName].concat(params));
  }
},
```

但我没怎么搞明白和直接调用```this._radioGroup.$emit('input',value)```有什么区别。

## 默认radio展示值

```html
<span class="el-radio__label">
  <slot></slot>
  <template v-if="!$slots.default">{{label}}</template>
</span>
```

如果让我实现的话我就直接```<slot>{{label}}</slot>```，关于slot的默认值，我猜测也是使用```!$slots.default```判断的，不太理解作者的意图。


## radio-group

radio-group中要注意的是这么一段：


```javascript
watch: {
  value(value) {
    this.$emit('change', value);
    this.dispatch('ElFormItem', 'el.form.change', [this.value]);
  }
}
```

这里要注意的是数据的流动顺序，input元素触发input事件，el-radio实例监听到该事件，然后通过el-radio-group实例抛出input事件，外界监听input事件，然后改变相应的值，这一系列就是“events up”的过程，然后el-radio-group实例监听到数据变换，抛出change事件。


## checkbox

在我看来radio和checkbox没有什么本质上的区别。element-ui为checkbox做了许多额外的工作，比如绑定true-value和false-value，还有一个indeterminate属性，但在我的日常使用中这些feature并没有怎么用到。