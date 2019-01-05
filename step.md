我们又遇到这种有父子关系的组件了，而且又是需要把父组件的数据传递给子组件。这里的实现是子组件访问$parent(话说element-ui迭代版本有点多，同一个问题的在不同组件下解决的方案有不一致情况，就不考虑统一一下吗)。

还一个问题，子组件需要自己是第几个子组件，因此el-steps维护了一个数组steps，当子组件实例化后push到这个数组中，由此确定是第几个子组件。这个实现真的对吗？对于step组件常见应用场景下是对的。我更倾向于在steps组件内操作vnode，把index作为prop传递下去。而且在step组件内部还应用了$children找前一个元素，这也是有问题的，$children根本保证不了顺序。[关于这一类问题的总结](https://github.com/jiangshanmeta/jiangshanmeta.github.io/issues/26)

剩下的基本就是样式问题了，不仔细看了。