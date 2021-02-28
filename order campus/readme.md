## 权限控制
- 纯前端控制，在每个路由设置meta信息，利用路由守卫，从Localstorage取出当前用户的userType，判断meta信息中是否包含当前用户的type，如果包含则正常跳转，不然则跳转到自己的提示界面。
```JavaScript
{
  //路由信息
      path: '/customer',
      name: 'customer',
      meta: {
        userTypes: ['3', '4'],
        requireAuth: true
      },
      component: customer,
      children: [
        {
          path: 'allDishs',
          name: 'allDishs',
          component: allDishs
        }
      ]
    }
  
  // 路由守卫
  router.beforeEach((to, from, next) => {
  const userType = localStorage.getItem('userType')
  if (to.meta.requireAuth) {
    if (to.meta && to.meta.userTypes.includes(userType)) {
      next()
    } else {
      console.log(to.meta.userTypes.includes(userType), userType, to.meta.userTypes)
      next({path: '/403'}) // 自己的提示组件
    }
  } else {
    next()
  }
})
```

## 支付宝接口接入 (面试讲述3)
- 主要是配置麻烦，不是企业应用，申请不到应用id，只能用沙箱环境替代；
- 比较复杂的点是，前端接收到返回的html代码怎么展现出来（form，script> fix
```JavaScript
document.body.innerHTML = res.data
          this.$nextTick(() => {
            document.forms[0].submit()
          })
```
doby内容直接用innerHTML替换，script用nextTick执行；

但是这样在后来支付成功回调后，需要给客户端发送消息，需要用到websocket，直接替换页面内容会断开websocket连接，发送失败。解决办法是新开一个窗口去支付。

## 计算每个骑手的位置 （面试讲述2）
- 支付成功后，需要从数据库中查询所有骑手，并根据经纬度计算离此地最近的一个骑手分配订单，但这个计算性能是很大的，十分影响性能，解决方法是新开一个webworker线程计算位置，这样需要的时候就可以直接使用，而不会等待时间过长。

## 高德地图接入
- amap undefined的问题，在eslint里配置好就完事了

## 订单无限列表优化处理  （面试讲述1）
- 只渲染可视区域内的订单组件（运用虚拟列表技术）
- 通过绑定scroll事件，计算scrollTop,单个元素高度，总高度，offset（偏移量）截取需要加载的那部分元素
1. 问题
- 渲染区域在顶部，而滚动到下方，数据消失
- 渲染组件只有几个，没法撑开滚动条，就没有滚动事件，一切就没有意义了
2. fix：
- 利用transform，将渲染区域向下平移offset（偏移量）
- 用一个空的div设置高度为列表项的总高度，绝对定位，设置z-index = -1,使它在渲染区域的里面，主要作用是撑开父元素，使之产生滚动条，这是为什么要用transform平移渲染区域的原因，因为真正的渲染区域的高度只是可视区域的组件撑开的。

## 支付成功后的回调
1. 问题描述
- 因为支付成功后，需要分配骑手配送，但客户端没办法知道支付是否成功，一般解决的方法是轮询或者websocket，因为websocket性能更好，所以选择了websocket解决
- 因为支付是返回的一个html form表单，之前是直接将body.innerHtml = res.data 在submit()；但这样会中断websocket的连接，导致无法推送消息；
- websocket在payController无法使用的原因
2. fix
- 前端在路由新建一个vue页面，收到返回的表单直接跳转到这个页面，利用query传递html，利用window新开一个窗口，这样websocket依然保持连接；
- 应该使用websocket的静态方法，（之前一直是使用的非静态方法，忽略了静态方法和静态属性，所以一直报错）
- 静态属性map表，存放userId，和websocket实例，静态方法群发消息（参数userId，message），这样之前的websocket的实例都存放在map表中，如果有userId就向该
用户发送消息，不然就向所有的连接用户发送消息；