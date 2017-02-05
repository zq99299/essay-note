# 使用vue-router
官方文档：https://router.vuejs.org/zh-cn/installation.html

## 安装
```bash
npm i vue-router -S
```

## 配置
一个基础的嵌套路由
```javascript
import Vue from 'vue';
import VueRouter from 'vue-router';
Vue.use(VueRouter);

// 普通加载，所有文件都打包到一个js文件中
// import Table from 'components/table/Table.vue'
// import Form from 'components/form/Form.vue'
// 按需加载
const Table = resolve => require(['components/table/Table.vue'], resolve);
const Form = resolve => require(['components/form/Form.vue'], resolve);

// 2. 定义路由
// 每个路由应该映射一个组件。 其中"component" 可以是
// 通过 Vue.extend() 创建的组件构造器，
// 或者，只是一个组件配置对象。
// 嵌套路由： 在Home.vue 里面有一个路由，所以要把路由都放到该节点下面。
// app 里面设定了一个路由：用来渲染全屏页面。 所以这里就造成了 嵌套路由

// 嵌套路由 : 寻找规则是：按配置规则查找页面，所以：路由路径不要重复，否则会出现问题
// 比如：如果下面的配置，将sign_in放到 / 后面，那么访问 sign_in的时候则会被匹配到 / 子路由的 path: '*'中去
const routes = [
  {path: '/sign_in', component: require('pages/signin/Signin.vue')},
  {
    path: '/',
    component: require('pages/Home.vue'),
    children: [
      {path: '/table', component: Table},
      {path: '/form', component: Form},
      {path: '/sign_in', component: Form}
      // {path: '*', component: require('pages/Home404.vue')}
    ]
  },
  {path: '*', component: require('pages/404.vue')}
];

// 3. 创建 router 实例，然后传 `routes` 配置
// 你还可以传别的配置参数, 不过先这么简单着吧。
const router = new VueRouter({
  routes // （缩写）相当于 routes: routes
});

// router.beforeEach((to, from, next) => {
//   // 全局钩子：用来做loading...
//
//   console.log('路由切换');
//   next();
// });
//
// router.afterEach(route => {
//   console.log('路由切换完成');
// });
export default router;
```

总结几点：

1. 路由的路径，在一个路由入口实例（包括嵌套子路由）里面的路径是唯一的，且配置按照从上倒下的顺序查找
2. 嵌套子路由：

```javascript
--- 组件A -------          在组件A里面点击路径 /home.那么组件A中的router-view被渲染成 home组件的内容
|<router-view>  |          --- 组件 home----
|               |          |<router-view>  |    在组件home里面想要跳转到 home的 router-view
|               |          |               |    那么就需要配置嵌套路由，如上面 路径为 “/” 中的 children



```