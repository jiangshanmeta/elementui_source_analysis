breadcrumb在我看来是类似于card组件的CSS占主导的组件，所以就简单一提。

首先，breadcrumb元素由breadcrumb和breadcrumb-item两个组件构成，这两个组件的关系类似于button-group组件和button组件。前一个组件除了包了一层之外基本不做什么，后一个组件才负责主要功能。

然后，breadcrumb-item是真正的面包屑元素，它由两部分构成：item__inner 和 separator。

```html
<template>
  <span class="el-breadcrumb__item">
    <span class="el-breadcrumb__item__inner" ref="link"><slot></slot></span>
    <span class="el-breadcrumb__separator">{{separator}}</span>
  </span>
</template>
```

breadcrumb-item可以当做router-link元素使用，其本质上是利用了vue-router的编程式导航：

```javascript
var self = this;
if (this.to) {
    let link = this.$refs.link;
    link.addEventListener('click', _ => {
        let to = this.to;
        self.replace ? self.$router.replace(to)
                        : self.$router.push(to);
    });
}
```

这一段代码让我很费解的一个地方是为什么特意使用了self来引用this，考虑到这里使用了箭头函数没有什么this指向问题。

最后是CSS部分。breadcrumb-item是通过浮动实现基础布局的(其实这里用inline-block也可以)，最后一个breadcrumb-item通过:last-child选择器选中，这也是这个选择器中规中矩的使用方法。