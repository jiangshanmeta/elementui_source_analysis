button也是页面布局常见元素，然而原生的button大家都懂，为此element-ui做了大量样式上的美化工作，这个层面上看看就好。我想说一些感觉是亮点的地方。


## 事件处理

根据vue的文档，为组件绑定原生事件需要使用```native```修饰符，但这样使用不是很方便，而且有时会忘记这一点。为了解决这个问题，element-ui监听了原生的```click```事件，然后```$emit```了一个```click```事件，这种设计其实挺常见的，尤其是在表单组件中时常会$emit一个input事件。

还有一个亮点是loading状态下，采用了CSS ```pointer-events: none```，这样就不触发鼠标事件。一个应用场景是这样的：点击按钮 首先标记为loading，然后向后端请求，这样即使用户点按钮也不会调用click事件的回调函数，因此我们也不用在回调里判断是否正在请求。


## slot处理

在element-ui的button.vue中有这么一段：

```html
<span v-if="$slots.default"><slot></slot></span>
```

这里的亮点是```$slots.default```的应用，通过它我们可以判断是否有内容通过默认slot分发。曾经见人为了实现同样的目的，手动获取DOM然后判断innerHTML的内容。