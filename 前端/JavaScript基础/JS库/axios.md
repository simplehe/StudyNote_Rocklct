## Axios
Axios 是一个基于 promise 的 HTTP 库，可以用在浏览器和 node.js 中。

Features:

 - 从浏览器中创建 XMLHttpRequests
 - 从 node.js 创建 http 请求
 - 支持 Promise API
 - 拦截请求和响应
 - 转换请求数据和响应数据
 - 取消请求
 - 自动转换 JSON 数据
 - 客户端支持防御 XSRF

安装

``` js
npm install axios
```

详细可见文档

### 创建实例
我们可以通过创建实例axios实例来发送请求

``` js
var instance = axios.create({
  baseURL: 'https://some-domain.com/api/',
  timeout: 1000,
  headers: {'X-Custom-Header': 'foobar'}
});
```

因此我们就可以调用instance.get等方法，下面请求等方法都可以用这个实例来做。

### 请求

``` js
// 为给定 ID 的 user 创建请求
axios.get('/user?ID=12345')
  .then(function (response) {
    console.log(response);
  })
  .catch(function (error) {
    console.log(error);
  });

axios.post('/user', {
      firstName: 'Fred',
      lastName: 'Flintstone'
    })
    .then(function (response) {
      console.log(response);
    })
    .catch(function (error) {
      console.log(error);
    });

```

还可以向axios传参

``` js
// 发送 POST 请求
axios({
  method: 'post',
  url: '/user/12345',
  data: {
    firstName: 'Fred',
    lastName: 'Flintstone'
  }
});
```
