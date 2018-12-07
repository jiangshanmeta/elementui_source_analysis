card这个组件其实主要也是负责渲染展示层面的，所以比价合适改写为functional component

```javascript
export default{
    name:"metaCard",
    functional:true,
    props:{
        header: {},
        bodyStyle: {},
        shadow:{
            type:String,
            default:"always",
        },
    },
    render(h,{props,data,slots}){
        const slotsContent = slots();
        let header = null;
        if(slotsContent.header || props.header){
            header = (
                <div
                    class="el-card__header"
                >
                    {slotsContent.header || props.header}
                </div>
            );
        }

        return (
            <div
                class={['el-card',`is-${props.shadow}-shadow`]}
                {...data}
            >
                {header}
                <div
                    class="el-card__body"
                    style={props.bodyStyle}
                >
                    {slotsContent.default}
                </div>
            </div>
        );
    },
}
```