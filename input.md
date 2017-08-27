input是表单基本元素，在element-ui中input组件是input-number、autocomplete等组件的基础。关于el-input组件，我想说以下三个方面：数据流动、限定textarea行数、input-group。

## 数据流动

父组件通过props的value属性向下传递要绑定的值（为了外界使用v-model语法），在初始化data的时候，currentValue被赋值为value值，并且监听value的变化，更新currentValue的值。同时el-input组件还在监听input元素的input事件，当监听到input事件时，首先```$emit```一个input事件通知父组件，然后设定currentValue的值，最后触发一个change事件。从父组件的角度来看，使用v-model语法相当于直接全盘接受el-input组件传递过来的值，也可以像el-input-number组件一样选择监听input事件，对值做一些特殊处理然后再接受值。


## 限定textarea行数

要限定textarea的行数，本质上是限定textarea的高度，为此我们要回答以下两个问题：一行所占高度是多少，总高度是多少。这两个问题根本上就是对DOM做一些操作，为此我们首先要获取DOM。element-ui直接利用了vue的ref属性。之前我只是用这个属性做组件的索引，在这里才知道当ref属性直接作用于普通DOM元素时，指向DOM元素。

element-ui采用如下方式计算单行高度：

```javascript
hiddenTextarea.value = '';
let singleRowHeight = hiddenTextarea.scrollHeight - paddingSize;
```

hiddenTextarea是一个高度为0 overflow:hidden的文本域，当它对的内容为空时，scrollHeight属性就对应了content-box的高度+上下padding。

要计算总高度也是利用了相同的原理，但是影响文字排版的影响很多，于是element-ui把textarea的font-fize、letter-spacing等属性的值赋给了hiddenTextarea，然后通过scrollHeight获得总高度。

最后就是限定高度了，这里稍微麻烦的一点是不同的box-sizing，设定height其含义是不一样的。


## input-group

input-group是基于input的常见设计了，element-ui是基于CSS table实现的，这个实现与bootstrap3的input-group的实现是一致的(bootstrap4是基于flex实现的)。