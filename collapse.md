实现collapse/according效果本身并不复杂，在我的[github 主页](http://jiangshanmeta.github.io/)上就实现了according效果。collapse/according效果实现上最大的问题是无法让```height```属性从0过渡到auto。我的解决方案是使用transform的scale属性，过渡的不是height，而是transform属性，从视觉效果上看结果是一致的。如果要过渡高度属性，就需要利用scrollHeight属性获取起始/目标高度，然后再进行两个具体高度之间的过渡。element-ui的collapse就是这么一个思路。

具体实现上是实现了functional component：

```javascript
export default {
  name: 'ElCollapseTransition',
  functional: true,
  render(h, { children }) {
    const data = {
      // 添加过渡需要的钩子，虽然不理解为啥不直接用一个具体的对象
      // 实现的思路就是上面的思路，利用scrollHeight获取高度  
      on: new Transition()
    };

    return h('transition', data, children);
  }
};
```


还有一个问题是如何确定哪些item是展开状态哪些item是收缩状态。这又涉及到这种有父子关系的组件的通讯的问题了，现在采用的是provide/inject这个方案，历史版本中有采用过手动$parent找collapse组件的情况。collapse组件维护一个数组activeNames，每个collapse-item组件看自己的唯一标识name属性是否在这个数组中就知道自己的状态了。


一个比较让人困惑的地方是activeNames的处理上。我们先看一下什么会影响activeNames，一个因素是props的value值，当value值变化时，activeNames也会变化，还有一个是下一层的collapse-item组件触发item-click事件，这回改变activeNames的值，同时也会让collapse发一个input事件，上层组件接受这个事件，一般而言会改变props的value值，于是又改变了activeNames的值。我们会发现，item-click事件改变activeNames是有点多余的，它最终会改变value值，进而影响到activeNames。我更倾向于把activeNames作为value值的一层包装(实现上是computed属性)，把数据格式统一为数组，它的唯一来源是value，只有value变activeNames才会变，其实是单一数据来源这一个思路。