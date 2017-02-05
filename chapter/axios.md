# axios 请求器
https://github.com/mzabriskie/axios
安装
```bash
npm install axios --dev
```

## 请求方式

### get 使用示例：
```javascript
 import axiosfrom 'axios';
 
 // get 可以这样添加参数，但是发出去之后，该框架会自动把参数拼接到url后面
  axios.get('/api/test/list',{
    username:'xxx'
  })
      .then(function (response) {
        console.log(response);
      })
      .catch(function (error) {
        console.log(error);
      });
```
`then`: 请求回来后会调用该方法
`catch` : 请求错误 包括 在 then 中处理抛出异常 都会在这里回调

### post 使用示例：
```javascript
 import axiosfrom 'axios';
 import qs from 'qs';
  axios.post('/api/login', qs.stringify({
    account: username,
    pwd: password
    }
    //,
   // {
   //   headers: {'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8'}
   // }
  )
    .then(function (response) {
      console.log(response);
    })
    .catch(function (error) {
      console.log(error);
    });
  
```
如果不用`qs.stringify` 来转换的话，发出去的是字符串，而不是 平时时候 `jqajax`发出去的是`Form data`类型的。

