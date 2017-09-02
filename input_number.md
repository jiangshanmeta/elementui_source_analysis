数字输入框是基于输入框构建的特异输入框，它能够限定输入的内容是数字类型，并且能够限定数字的范围。

要实现一个数字输入框，我们需要一个基础输入框，然后去监听输入事件，在输入事件的回调中我们去验证输入的内容是否是合法数字。

```javascript
handleInput(value) {
    if (value === '') {
        return;
    }
    const newVal = Number(value);
    if (!isNaN(newVal)) {
        this.setCurrentValue(newVal);
    } else {
        this.$refs.input.setCurrentValue(this.currentValue);
    }
}
```

我们理一下数据的流动方向。el-input元素emit一个input事件，传入文本框的值。el-input-number先简单校验，如果不能转换为数字，就将el-input的值赋为el-input-number缓存的合法的数值，如果能转换为数值，就进一步判断数值的范围：

```javascript
setCurrentValue(newVal) {
    const oldVal = this.currentValue;
    if (newVal >= this.max) newVal = this.max;
    if (newVal <= this.min) newVal = this.min;
    if (oldVal === newVal) {
        this.$refs.input.setCurrentValue(this.currentValue);
        return;
    }
    this.$emit('change', newVal, oldVal);
    this.$emit('input', newVal);
    this.currentValue = newVal;
},
```

max和min的默认值分别是Infinity和-Infinity，正常数值都会在这一范围内。数值限定后el-input-number组件触发input事件，通知el-input-number的父组件更新数据，然后更新自身缓存值currentValue，el-input组件由于watch了该缓存值，因而也会更新。

现在我们明白了el-input-number和el-input之间的数据流，我们也知道了el-input-number组件如何向上传递数据，最后一个问题是el-input-number组件的父组件如何向el-input-number传递数据：

```javascript
watch: {
    value: {
        immediate: true,
        handler(value) {
            let newVal = Number(value);
            if (isNaN(newVal)) return;
            if (newVal >= this.max) newVal = this.max;
            if (newVal <= this.min) newVal = this.min;
            this.currentValue = newVal;
            this.$emit('input', newVal);
        }
    }
},
```

el-input-number的父组件通过props的value向下传递数据，el-input-number监听这一值的变化，并在输入值为合理值的情况下更新自身的缓存值currentValue。在初始化阶段，如果父组件传递的值为合理的数值，将更新缓存值currentValue。

关于数据流动还有一点是debounce方法的使用，它是用来限定handleInput执行频率的，这个方法我会在underscore源码解读中讲解underscore的实现。

关于数字输入框最后一点要说的是一个指令：

```javascript
directives: {
    repeatClick: {
        bind(el, binding, vnode) {
            let interval = null;
            let startTime;
            const handler = () => vnode.context[binding.expression].apply();
            const clear = () => {
                if (new Date() - startTime < 100) {
                    handler();
                }
                clearInterval(interval);
                interval = null;
            };

            on(el, 'mousedown', () => {
                startTime = new Date();
                once(document, 'mouseup', clear);
                clearInterval(interval);
                interval = setInterval(handler, 100);
            });
        }
    }
},
```

这个指令是作用在数字输入框的加减号上的，当我们按住加减号不动时，会按照一定频率持续执行加减操作。这里通过```vnode.context```获取到相应的vue实例，然后调用实例上相应的方法。