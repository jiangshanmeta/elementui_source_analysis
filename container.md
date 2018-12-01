element-ui提供的布局容器组件有el-container、el-header、el-aside、el-main、el-footer。我其实比较想吐槽这几个组件，他们本身只是布局性质的，对应实现就是一个div 加个class 再加上slot，真的有必要单独写个组件出来吗？暴露几个CSS类名就够了，而且非要写组件的话用functional component实现其实更合适。

比较有水平的是在el-container组件，根据子元素的情况给direction默认值

```javascript
computed: {
    isVertical() {
        if (this.direction === 'vertical') {
            return true;
        } else if (this.direction === 'horizontal') {
            return false;
        }
        // props没指定direction，根据slot中的节点决定direction默认值
        return this.$slots && this.$slots.default
            ? this.$slots.default.some(vnode => {
                // 拿到组件的tag
                const tag = vnode.componentOptions && vnode.componentOptions.tag;
                return tag === 'el-header' || tag === 'el-footer';
            })
            : false;
    }
}

```