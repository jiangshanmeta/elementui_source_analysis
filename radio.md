radio涉及这么几个组件：radio、radio-group、radio-buttton。radio可以独立使用(虽然很少这么用)，也可以和radio-group一起使用，radio-button组件可以说和radio组件是等价的，但是必须和radio-group一起使用。

当radio和radio-group一起使用的时候，我们再次遇到了这种父子关联组件通讯的问题，现在更推荐的解决方案是使用provide/inject，然而算是历史原因吧，element-ui采用的沿着$parent向上查找(相当于手动实现了一遍provide/inject)，而且还有一个mixin emitter，其中的dispatch也是利用了这一思路调用父组件的方法。

还有一点用得很巧妙的是computed属性model

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

这个技巧在我写的[基于vue的通用后台](https://github.com/jiangshanmeta/vue-admin)被大量应用。

还有就是有些键盘相关的操作，像什么tabindex，其实我不是特别清楚多少人会有浏览网页时用键盘，尤其是tab的习惯。