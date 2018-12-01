button组件是很基础的组件了，实现一个button组件其实并没有什么太大的难度，稍微有经验的开发看element-ui的button实现应该没有太大的压力。

考虑到button组件并没有自身的状态，而且也并不需要生命周期之类的，我把element-ui的button组件改造成了functional component。

```javascript
export default{
    name:"metaButton",
    functional:true,
    props:{
        type: {
            type: String,
            default: 'default'
        },
        size: String,
        icon: {
            type: String,
            default: ''
        },
        nativeType: {
            type: String,
            default: 'button'
        },
        loading: Boolean,
        disabled: Boolean,
        plain: Boolean,
        autofocus: Boolean,
        round: Boolean,
        circle: Boolean
    },
    render(h,{slots,data,props,parent}){
        const buttonSize = props.size || (parent.$ELEMENT || {}).size;
        const cls = [
            'el-button',
            props.type ? 'el-button--' + props.type : '',
            buttonSize ? 'el-button--' + buttonSize : '',
            {
                'is-disabled': props.disabled,
                'is-loading': props.loading,
                'is-plain': props.plain,
                'is-round': props.round,
                'is-circle': props.circle
            }
        ];
        return (
            <button
                class={cls}
                disabled={props.disabled || props.loading}
                autofocus={props.autofocus}
                type={props.nativeType}
                {...data}
            >
                {props.loading && <i class="el-icon-loading"></i>}
                {props.icon && !props.loading && <i class={props.icon}></i>}
                {slots().default}
            </button>
        )
    },
}
```

这里并没有完全复现element-ui的button组件的功能，因为在[slot中的functional component和正常的component parent指向并不一致](https://github.com/vuejs/vue/issues/4550)，这意味着基于基于节点树的功能(如provide/inject)表现可能不一致。


顺便还有elButtonGroup组件，它本身仅仅是个容器，所以也是适合改造成functional component的：

```javascript
export default{
    name:"metaButtonGroup",
    functional:true,
    render(h,{data,slots}){
        return (
            <div 
                class="el-button-group"
                {...data}
            >
                {slots().default}
            </div>
        )
    },
}
```

css层面在disable情况下使用了```pointer-events: none```，这样就不会触发鼠标事件了。