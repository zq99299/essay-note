# vuex

## 入门使用


一个简单的Store
> 参考文档：http://vuex.vuejs.org/zh-cn/getting-started.html

```javascript

import Vue from 'vue';
import Vuex from 'vuex';
Vue.use(Vuex); // 在模块化构建系统中，确保在开头调用了 Vue.use(Vuex)

const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++;
    }
  }
});

export default store;


// 把该段代码包装成 一个 es6 模块
```

> 参考文档： http://vuex.vuejs.org/zh-cn/state.html

在es6里使用Import，在同一个环境里面，该文件只会被导入一次。
所以你在使用该文件的时候。先导入该js文件，然后当次计算属性使用。这样其他地方修改了。这里也会跟着变化
```javascript
import store from '../vuexdemo.js'
// 创建一个 Counter 组件
const Counter = {
  template: `<div>{{ count }}</div>`,
  computed: {
    count () {
      return store.state.count
    }
  }
}
// store.commit('increment') 通过commit 调用方法来触发改变值。
```
现在 使用也会了。但是有一个问题。如果所有组件都想使用怎么办，那么就得所有组件都导入吗？
官方提供的vuex是一个插件吧，在不同组件中使用同一个。可以使用如下办法：


```javascript
import store from '../vuexdemo.js'
// 在根组件注册store
const app = new Vue({
  el: '#app',
  // 把 store 对象提供给 “store” 选项，这可以把 store 的实例注入所有的子组件
  store,
  components: { Counter },
  template: `
    <div class="app">
      <counter></counter>
    </div>
  `
})

// 子组件通过：
this.$store.state.count  使用
```