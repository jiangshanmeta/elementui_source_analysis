类似于row和col组件，breadcrumb组件和breadcrumb-item组件也是成对出现的，breadcrumb-item组件也需要从breadcrumb组件获取必要的信息。在说前者的时候，我已经提到了利用provide/inject处理这种有父子关系的组件的通信是很有用的，breadcrumb就是采用了这一方式。

```javascript
// breadcrumb组件把自身向下注入,item就能获取seperator的信息了
provide() {
    return {
        elBreadcrumb: this
    };
},
```

关于这两个组件组件，有几个地方的实现我不太理解：

```javascript
//breadcrumb
mounted() {
  // 获取最后一个item的dom节点，添加属性
  const items = this.$el.querySelectorAll('.el-breadcrumb__item');
  if (items.length) {
    items[items.length - 1].setAttribute('aria-current', 'page');
  }
}
```

一方面是考虑到breadcrumb的内容可能是动态的，意味着最后一个item可能是变化的，这种只处理一次的做法不见得合适。另一方面是为什么要操作dom，操作vnode就可以完成这一点：

```javascript
render(h){
    const children = this.$slots.default;
    if(!Array.isArray(children)){
        return null;
    }
    // 筛选breadcrumb-item的vnode
    const breadcrumbItemVNodes = children.filter((vnode)=>vnode.componentOptions && vnode.componentOptions.tag === 'meta-breadcrumb-item')

    if(breadcrumbItemVNodes.length === 0){
        return null;
    }
    // 最后一个添加属性
    const lastBreadcrumnItemVNode = breadcrumbItemVNodes[breadcrumbItemVNodes.length-1];
    const attrs = lastBreadcrumnItemVNode.data.attrs || (lastBreadcrumnItemVNode.data.attrs={});
    attrs['arial-current'] = 'page';

    return (
        <div
            class="el-breadcrumb"
            aria-label="Breadcrumb"
            role="navigation"
        >
            {breadcrumbItemVNodes}
        </div>
    )
},
```


另一个不太理解的是在breadcrumb-item组件上：

```javascript
const link = this.$refs.link;
link.setAttribute('role', 'link');
link.addEventListener('click', _ => {
  const { to, $router } = this;
  if (!to || !$router) return;
  this.replace ? $router.replace(to) : $router.push(to);
});
```

一般而言是尽可能不要利用ref碰DOM节点，而且这里拿到DOM节点仅仅是为了添加事件，为啥不用vue提供的语法绑定事件而是要手动绑定事件？