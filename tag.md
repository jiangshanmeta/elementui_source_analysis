tag这个组件与badge很像，都是CSS为主的组件，element-ui将tag封装成了组件，但是大部分的属性都是用来控制样式的，比如type、hit、color。

tag组件提供一个关闭事件：

```javascript
methods: {
    handleClose(event) {
        this.$emit('close', event);
    }
}
```

具体判断是否要关闭是由我们这群二次开发者自行决定。

element-ui为tag进入和退出做了过渡效果：

```css
.el-zoom-in-center-enter-active,
.el-zoom-in-center-leave-active {
  transition: all .3s cubic-bezier(.55,0,.1,1);
}
.el-zoom-in-center-enter,
.el-zoom-in-center-leave-active {
  opacity: 0;
  transform: scaleX(0);
}
```

效果不复杂，就是透明度的改变和水平方向上的缩放。