checkbox对应三个组件：checkbox、checkbox-group、checkbox-button。其实这三个组件和radio系列的radio、radio-group、radio-button基本是对应的。稍微复杂一点的是checkbox的值可能有多种情况，比如数组(最常用的情况)、布尔值，使得对是否选中稍微有些麻烦，但其实也只是多几个if else的事。

以及，和radio系列同样的问题，用provide/inject关联checkbox-group和checkbox更合适。