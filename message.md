一般的vue组件都是声明式(模板)调用，而message组件是通过命令式调用。这一种全新的使用方式算是该组件的一个实现难点了吧。


```javascript
// vue组件
import Main from './main.vue';
// 手动转化为vue子类 对于一般vue组件，这一层转换是vue内部实现的
let MessageConstructor = Vue.extend(Main);

// 实例化组件 此时还没有mount 某有dom
instance = new MessageConstructor({
    data: options
});
instance.id = id;
if (isVNode(instance.message)) {
    instance.$slots.default = [instance.message];
    instance.message = null;
}
// mount 有对应的DOM了，但是还没有挂载
instance.vm = instance.$mount();
// 手动挂载到body下面，进入文档流中
document.body.appendChild(instance.vm.$el);
// 从隐藏到显示 
instance.vm.visible = true;
```

我们调用```$message```方法，该方法会手动生成一个Message组件的实例，然后手动挂载到body下，剩下的就是组件内部维护的了。

组件内部在mounted时会开启一个定时器，到时关闭Message弹框。

```javascript
watch: {
    closed(newVal) {
        if (newVal) {
            // 从显示到隐藏 执行淡出动画
            this.visible = false;
            // 动画执行完 要做的
            this.$el.addEventListener('transitionend', this.destroyElement);
        }
    }
},

destroyElement() {
    this.$el.removeEventListener('transitionend', this.destroyElement);
    // 销毁组件实例
    this.$destroy(true);
    // 手动移除对应的DOM节点
    this.$el.parentNode.removeChild(this.$el);
},
```