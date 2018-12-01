栅格系统是布局常用元素，关于其对应的CSS实现，我在讲述[bootstrap3的栅格](https://github.com/jiangshanmeta/jiangshanmeta.github.io/issues/4)和[bootstrap4的栅格](https://github.com/jiangshanmeta/jiangshanmeta.github.io/issues/16)时已经讲过了，我这里只说明element-ui是如何把栅格系统组件化的。

## row

element-ui直接采用了render函数的写法，毕竟这个组件不是很复杂：

```javascript
export default {
  name: 'ElRow',
  // 用于给col组件判断是否是row组件的
  componentName: 'ElRow',
  props: {
    // 可以自定义标签类型
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
      const ret = {};

      if (this.gutter) {
        ret.marginLeft = `-${this.gutter / 2}px`;
        ret.marginRight = ret.marginLeft;
      }

      return ret;
    }
  },

  render(h) {
    return h(this.tag, {
      // 其实class也可以作为computed属性
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

row组件的render函数用jsx可以这么写：

```javascript
render(h){
    const Tag = this.tag;
    return (
        <Tag
            class={this.cls}
            style={this.style}
        >
            {this.$slots.default}
        </Tag>
    );
}
```




## col

对于col组件来说，一个难题是如何获取gutter，因为gutter是row组件的prop而不是col组件的prop。

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

这个有点类似于provide/inject的实现，就是不断向上查找到最近的row组件，然后获取row组件的gutter属性。如果要用provide/inject的话，row组件provide gutter属性并不是一个好主意，因为即使利用getter ，inject的时候也只是一个简单值，并不是响应式的，一个比较常见的解决方案是provide整个组件实例。

```javascript
provide(){
  return {
    // provide整个组件实例
    row:this
  }
}
```

考虑到col组件只是一个布局展示性质的组件，没有自身状态和生命周期，可以考虑改写成functional component，但是此时parent指向和有状态组件是不一致的，改写的时候需要把gutter作为prop传递给col组件。









