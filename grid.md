栅格系统是布局常用元素，关于其对应的CSS实现，我在讲述[bootstrap3的栅格](https://github.com/jiangshanmeta/jiangshanmeta.github.io/issues/4)和[bootstrap4的栅格](https://github.com/jiangshanmeta/jiangshanmeta.github.io/issues/16)时已经讲过了，我这里只说明element-ui是如何把栅格系统组件化的。

## row

element-ui实现的栅格有两种模式，一种是基于浮动的，另一种是基于flex，负责切换这两个的是```type```属性。通过gutter属性，栅格系统可以设定不同的槽宽度。通过tag属性，我们可以指定要渲染的元素标签。这个写起来其实没难度，我的一种实现是这样的：

```vue
<template>
    <component :is="tag" :style="computedStyle" :class="computedClass">
        <slot></slot>
    </component>
</template>

<script>
export default{
    name:'ElRow',
    props:{
        type:{
            type:String,
            default:'float'
        },
        gutter:{
            type:Number,
            default:0,
        },
        tag:{
            type:String,
            default:'div',
        },
    },
    computed:{
        computedStyle(){
            let data = {};
            if(this.gutter){
                data['marginLeft'] = `-${this.gutter/2}px`;
                data['marginRight'] = data['marginLeft'];
            }
            return data;
        },
        computedClass(){
            let data = ['el-row'];
            if(this.type === 'flex'){
                data.push('el-row--flex');
            }
            return data;
        },
    },
}
</script>
```

elementui直接采用了render函数的写法，毕竟这个组件不是很复杂：

```javascript
export default {
  name: 'ElRow',
  componentName: 'ElRow',
  props: {
    tag: {
      type: String,
      default: 'div'
    },
    gutter: Number,
    type: String,
    justify: {
      type: String,
      default: 'start'
    },
    align: {
      type: String,
      default: 'top'
    }
  },

  computed: {
    style() {
      var ret = {};

      if (this.gutter) {
        ret.marginLeft = `-${this.gutter / 2}px`;
        ret.marginRight = ret.marginLeft;
      }

      return ret;
    }
  },

  render(h) {
    return h(this.tag, {
      class: [
        'el-row',
        this.justify !== 'start' ? `is-justify-${this.justify}` : '',
        this.align !== 'top' ? `is-align-${this.align}` : '',
        { 'el-row--flex': this.type === 'flex' }
      ],
      style: this.style
    }, this.$slots.default);
  }
};
```

上面的代码和我的实现是等价的，一个要注意的点是这里有个自定义属性componentName，在讲col的实现时我们会看到这个参数的用途。

## col

对于col来说，占多少个格子，偏移多少这种都是很好实现的，一个难点在于如何获取gutter的大小，毕竟gutter是设定在row上的而不是col上的。对于这个问题element-ui是这么解决的：

```javascript
  computed: {
    gutter() {
      let parent = this.$parent;
      while (parent && parent.$options.componentName !== 'ElRow') {
        parent = parent.$parent;
      }
      return parent ? parent.gutter : 0;
    }
  },
```

我们向上寻找最近的ElRow，直接访问它的gutter属性即可。在row中提到的自定义属性componentName是为了表示这个组件是ElRow。

ElCol的其他feature的相关实现请自行阅读相关源码。