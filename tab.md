tab 组件包括tabs、tab-pane、tab-nav三个基本组件，其中tab-nav不是我们显式使用，而是由tabs组件进行使用。于是我们所面临的第一个问题就来了，如何获取tab-nav需要的信息(label/nav)? element-ui的做法有点复杂：首先在tabs组件维护一个panes数组，这个数组的内容是tab-pane组件实例，然后把这个panes数组传递给tab-nav组件，tab-nav组件根据这个tab-pane数组获取必要的信息。合适吗？相对来说是个比较不错的解决方案了，我们仅仅是在tabs组件维护panes这个tab-pane组件实例数组时需要碰一下vnode，其他的都是我们很熟悉的组件相关的内容了。我的实现是碰vnode，tab组件分析slot中的子节点vnode，得到需要的label/name信息，把得到的数组传递给tab-nav组件，我的这个实现其实并不算好，访问vnode其实应该尽量避免。

tab-pane组件负责控制自身slot内容的显示和隐藏(display 样式层面)。element-ui的实现提供了一个lazy选项，tab-pane slot的内容一开始并不渲染(v-if控制)，直到第一次需要展示这个tab-pane的内容时，才利用v-show控制显示隐藏。这样做可以减少一些根本没用到的组件实例化。但是这样还不够，我更希望能让slot中的内容知道该tab-pane什么时候active 什么时候不active，这样可以做更多的操作，比如只有tab-pane为active时，才去轮询后台接口，对此我的一个实现是：

```javascript
render(){
    const style = {};
    if(!this.active){
        style['display'] = 'none';
    }
    
    return (
        <div
            class="el-tab-pane"
            style={style}
        >
            {
                this.$scopedSlots.default?
                    this.$scopedSlots.default({active:this.active}):
                    this.$slots.default
            }
        </div>
    )
},
```

看element-ui的tab组件，会发现其中的数据流也是乱的。其中的currentName应该作为computed属性，为了兼容value和activeName两种不同的写法，setCurrentName，应该只抛出input事件，那个beforeLeave其实也应该干掉，父组件决定可以切换tab，就改变props的value/activeName，父组件决定不切换tab，就不改变props的value/activeName。之前提过，element-ui的collapse也有类似的问题，又是违背了“单一数据来源”这一思路。