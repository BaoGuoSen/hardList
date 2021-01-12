## 修改ui组件的样式
- 要考虑的问题，避免全局污染
1. 因为一般用的样式是scope作用于当前样式，使用样式覆盖就不起作用，这时用/deep/就行，而且不会污染其他地方。
- 步骤
1. 打开开发者工具，查看要修改组件的作用样式
2. 根据css样式优先级，对它的父组件加一个空的作用样式，使用样式覆盖就行了
3. 详情https://juejin.cn/post/6877452643145351176

## 使用vuex时，踩坑总结
1. 在刷新页面时，vuex会重新发送请求，state的数据会重新初始化；
2. vuex数据的请求在dom渲染完成后，意味着在vue的mouted及之前都无法获取vuex数据，computed在vue实例初始化阶段执行（beforecreate & created之间）computed能输出store的某个对象，但不能输出具体指，因为对象是引用。
3. 使用vuex数据应该用vue的watch函数，根据具体的场景结合immediately使用。

## overflow踩坑
- 场景：现需要在一个表单的外部区域加上区域标识，用绝对定位将标识定位在表单外部；
1. 所以需要在表单水平方向的overflow设置为visible，垂直方向为scroll
2. 但水平方向不生效，原因是overflow设置为visible，不为bfc，其他值为dfc所以会有冲突
3. 解决方法：拓展左右的padding；将区域标识放在padding区域，能得到理想的效果；