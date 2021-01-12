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
