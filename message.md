之前提及的组件调用模式都是通过模板，比如：

```html
<div>
    <el-input></el-input>
</div>
```

这是组件基本的调用模式，message组件(似乎叫插件更合适一点)调用的方式是通过原型上的方法：

```javascript
this.$message('这是一条消息提示');
```

显然，Vue原型对象上添加了一个新的方法$message，通过这个方法实现实例化一个组件。我们的重点是如何通过一个方法实例化一个组件，而不是组件内部的具体实现。

```javascript
let MessageConstructor = Vue.extend(require('./main.vue'));
```

main.vue文件中是组件的具体实现，我们基于它利用Vue.extend方法创造一个Vue的子类MessageConstructor，在$message方法中实例化了这个类。

```javascript
var Message = function(options) {
    if (Vue.prototype.$isServer) return;

    // 预处理参数
    options = options || {};
    if (typeof options === 'string') {
        options = {
            message: options
        };
    }
    let userOnClose = options.onClose;
    let id = 'message_' + seed++;
    options.onClose = function() {
        Message.close(id, userOnClose);
    };

    // 实例化 组件
    instance = new MessageConstructor({
        data: options
    });
    instance.id = id;

    // 允许写render函数控制message的内容
    // 然而一般用不到
    if (isVNode(instance.message)) {
        // $slots.default是一组VNode的集合
        instance.$slots.default = [instance.message];
        instance.message = null;
    }

    // 手动挂载未挂载实例到document.body下
    // $mount方法不是返回自身吗？这一步操作是为啥？
    instance.vm = instance.$mount();
    document.body.appendChild(instance.vm.$el);

    // 默认为隐藏的，控制它显示出来
    instance.vm.visible = true;

    // 这个类似于别名的操作是用来干啥的？
    instance.dom = instance.vm.$el;

    // 控制z-index，保证后实例化的显示上在上面
    instance.dom.style.zIndex = PopupManager.nextZIndex();
    instances.push(instance);
    return instance.vm;
};
```

其实使用模板模式调用也是有类似的过程，都有创建Vue子类，实例化Vue子类这一过程，只是通过模板调用这些相关操作是框架自己完成的。

message有不同的type，一方面可以通过参数进行设定，另一方面可以通过几个语法糖方法进行设定：

```javascript
['success', 'warning', 'info', 'error'].forEach(type => {
    Message[type] = options => {
        if (typeof options === 'string') {
            options = {
                message: options
            };
        }
        // 设定类型
        options.type = type;
        // 最终还是调用message方法
        return Message(options);
    };
});
```

在组件内部实现上要提及的是它的销毁方法：

```javascript
destroyElement() {
    // 解除事件监听
    this.$el.removeEventListener('transitionend', this.destroyElement);
    // 完全销毁这个实例
    this.$destroy(true);
    // 移除DOM节点
    this.$el.parentNode.removeChild(this.$el);
},
```

由于控制message弹出框显示隐藏使用的是v-show指令，因此message弹出框消失后只是显示上消失了，对应的DOM节点和Vue实例都是存在的，然而我们已经不需要这些节点了，因此需要手动销毁实例和DOM节点。