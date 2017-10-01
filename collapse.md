实现collapse/according效果本身并不复杂，在我的[github 主页](http://jiangshanmeta.github.io/)上就实现了according效果。collapse/according效果实现上最大的问题是无法让```height```属性从0过渡到auto。我的解决方案是使用transform的scale属性，过渡的不是height，而是transform属性，从视觉效果上看结果是一致的。如果要过渡高度属性，就需要利用scrollHeight属性获取起始/目标高度，然后再进行过渡。element-ui的collapse就是这么一个思路。

具体实现上，element-ui采用了vue过渡的javascript钩子，在钩子函数中去获取scrollHeight属性，控制过渡效果。让我比较奇怪的是为什么要特地声明一个类来做这件事情，其实以mixin的形式来做这件事已经足够了。

让我感觉写得很好的地方是这里采用了函数式组件。函数式组件我其实没有用过，看了一下文档我觉得这句话说得很好：

> 在作为包装组件时它们也同样非常有用，比如，当你需要做这些时：
    * 程序化地在多个组件中选择一个
    * 在将 children, props, data 传递给子组件之前操作它们。

这里collapse-transition就相当于是一个包装组件，它对data的on属性做了特殊处理，挂上了一些过渡时用到的钩子函数，然后传递给子组件transition。这个点我还需要理解一段时间。

最后的一个问题是如何确定哪些item是展开状态哪些item是收缩状态。每一个item都有一个name属性作为唯一标识，我们通过通过value属性将哪些item需要展开传递给collapse组件，collapse将其规格化成为```activeNames```属性，每一个collapse-item通过访问父组件collaspe的```activeNames```属性判断自己是要展开还是要收缩:

```javascript
computed: {
    isActive() {
        return this.$parent.activeNames.indexOf(this.name) > -1;
    }
},
```

当我们点击每个标题的title时，collapse-item组件会通知collapse组件要toggle状态：

```javascript
handleHeaderClick() {
    this.dispatch('ElCollapse', 'item-click', this);
}
```

collapse组件一方面更新展开收缩状况列表，一方面通知我们状态更新：

```javascript
setActiveNames(activeNames) {
    activeNames = [].concat(activeNames);
    let value = this.accordion ? activeNames[0] : activeNames;
    this.activeNames = activeNames;
    this.$emit('input', value);
    this.$emit('change', value);
},
handleItemClick(item) {
    if (this.accordion) {
        this.setActiveNames(
            (this.activeNames[0] || this.activeNames[0] === 0) &&
            this.activeNames[0] === item.name
            ? '' : item.name
        );
    } else {
        let activeNames = this.activeNames.slice(0);
        let index = activeNames.indexOf(item.name);

        if (index > -1) {
            activeNames.splice(index, 1);
        } else {
            activeNames.push(item.name);
        }
        this.setActiveNames(activeNames);
    }
}
```

这一个组件的数据流动其实并不复杂，collapse组件相当于在我们和collpase-item之间拦了一层，它规格化了我们传递下去的展开状态列表，使collapse-item能通过一个统一的接口确定自身状态，同时它负责反转collpase-item的展开收缩状态，并通知我们展开列表更新。