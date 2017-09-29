
---
title: axios使用说明
tags: ['http']
categories : Tools
---

基于Promise的浏览器和node.js的HTTP 客户端

## Features - 特征

- 从浏览器发送[XMLHttpRequests](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest)
- 从node.js发送[http](http://nodejs.org/api/http.html)请求
- 支持 [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) API
- 拦截请求及响应
<!-- more -->
- 改造请求和响应数据
- 取消请求
- 自动转换JSON数据
- 防范客户端[XSRF](http://en.wikipedia.org/wiki/Cross-site_request_forgery)

## Installing

Using npm:

```bash
$ npm install axios
```

Using bower:

```bash
$ bower install axios
```

Using cdn:

```html
<script src="https://unpkg.com/axios/dist/axios.min.js"></script>
```

## Example

执行`GET`请求

```js
// 向具有指定ID的用户发出请求
axios.get('/user?ID=12345')
  .then(function (response) {
    console.log(response);
  })
  .catch(function (error) {
    console.log(error);
  });

// 上述请求也可以作为
axios.get('/user', {
    params: {
      ID: 12345
    }
  })
  .then(function (response) {
    console.log(response);
  })
  .catch(function (error) {
    console.log(error);
  });
```

执行`POST`请求

```js
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

执行多个并发请求

```js
function getUserAccount() {
  return axios.get('/user/12345');
}

function getUserPermissions() {
  return axios.get('/user/12345/permissions');
}

axios.all([getUserAccount(), getUserPermissions()])
  .then(axios.spread(function (acct, perms) {
    // Both requests are now complete
  }));
```

## axios API

请求可以通过相关设置`axios`.

##### axios(config)

```js
// Send a POST request
axios({
  method: 'post',
  url: '/user/12345',
  data: {
    firstName: 'Fred',
    lastName: 'Flintstone'
  }
});
```

```js
// GET request for remote image
axios({
  method:'get',
  url:'http://bit.ly/2mTM3nY',
  responseType:'stream'
})
  .then(function(response) {
  response.data.pipe(fs.createWriteStream('ada_lovelace.jpg'))
});
```

##### axios(url[, config])

```js
// Send a GET request (default method)
axios('/user/12345');
```

## 请求方法的别名

为方便起见，已为所有支持的请求方法提供别名.

##### axios.request(config)
##### axios.get(url[, config])
##### axios.delete(url[, config])
##### axios.head(url[, config])
##### axios.options(url[, config])
##### axios.post(url[, data[, config]])
##### axios.put(url[, data[, config]])
##### axios.patch(url[, data[, config]])

###### 注意

当使用别名方法`url`，`method`和`data`属性不需要在配置中指定。

## 并发

处理并发请求的辅助函数。

##### axios.all(iterable)
##### axios.spread(callback)

## 创建一个实例

你可以用一个自定义的配置创建可信的一个新实例

##### axios.create([config])

```js
var instance = axios.create({
  baseURL: 'https://some-domain.com/api/',
  timeout: 1000,
  headers: {'X-Custom-Header': 'foobar'}
});
```

## 实例方法

可用的实例方法如下。指定的配置将与实例配置合并。

##### axios#request(config)
##### axios#get(url[, config])
##### axios#delete(url[, config])
##### axios#head(url[, config])
##### axios#options(url[, config])
##### axios#post(url[, data[, config]])
##### axios#put(url[, data[, config]])
##### axios#patch(url[, data[, config]])

## 请求配置

这些是用于生成请求的可用配置选项。 只有 `url` 是必须的. 如果请求的 `method`没有指定默认是`GET`.

```js
{
  // `url` 是将用于请求的服务器URL
  url: '/user',

  // `method` 是在请求时使用的请求方法
  method: 'get', // default

  // `baseURL`将被添加到`url`，除非`url`是绝对的
  // 可以方便的设置`baseURL`为一个实例的axios传递相对URL   
  //到该实例的方法。
  baseURL: 'https://some-domain.com/api/',

  // `transformRequest` 允许在发送到服务器之前更改请求数据
  // 这仅适用于请求方法 'PUT', 'POST', and 'PATCH'
  // 数组中的最后一个函数必须返回一个字符串或Buffer的实例, ArrayBuffer
  // FormData or Stream
  // 可以修改标题对象
  transformRequest: [function (data, headers) {
    //  Do whatever you want to transform the data

    return data;
  }],

  // `transformResponse` 允许在它被传递给then/catch之前对响应数据进行更改
  transformResponse: [function (data) {
    // Do whatever you want to transform the data

    return data;
  }],

  // `headers` 要被发送的请求头信息
  headers: {'X-Requested-With': 'XMLHttpRequest'},

  // `params` 是要与请求一起发送的URL参数
  // 必须是纯对象或URLSearchParams对象
  // (Must be a plain object or a URLSearchParams object)
  params: {
    ID: 12345
  },

  // `paramsSerializer` 是一个可选的函数，负责序列化`params`
  // (e.g. https://www.npmjs.com/package/qs, http://api.jquery.com/jquery.param/)
  paramsSerializer: function(params) {
    return Qs.stringify(params, {arrayFormat: 'brackets'})
  },

  // `data` 是作为请求体将被发送的数据
  // 仅适用于请求方式 'PUT', 'POST', and 'PATCH'
  // 当没有设置“transformRequest”时，必须是以下类型之一:
  // - string, plain object, ArrayBuffer, ArrayBufferView, URLSearchParams
  // - Browser only: FormData, File, Blob
  // - Node only: Stream, Buffer
  data: {
    firstName: 'Fred'
  },

  // `timeout` 指定请求超时的毫秒数.
  // 如果请求比“timeout”长，请求将被中止
  timeout: 1000,

  // `withCredentials` 表示是否支持跨域请求
  // 应使用凭证
  withCredentials: false, // default

  // `adapter` 允许自定义处理请求，使测试更容易.
  // 返回 promise 并提供有效的回复
  // (Return a promise and supply a valid response).
  adapter: function (config) {
    /* ... */
  },

  // `auth` 表示应使用HTTP Basic认证，并提供凭据。
  // 这将设置一个 `Authorization` 头, 覆盖任何现有的
  // 使用`header`设置的`Authorization`自定义头文件。
  auth: {
    username: 'janedoe',
    password: 's00pers3cret'
  },

  // `responseType` 表示服务器将响应的数据类型   
  //选项是'arraybuffer'，'blob'，'document'，'json'，'text'，'stream
  responseType: 'json', // default

  // `xsrfCookieName` is the name of the cookie to use as a value for xsrf token
  // `xsrfCookieName` 是携带xsrf token的http标头的名称
  xsrfCookieName: 'XSRF-TOKEN', // default

  // `xsrfHeaderName` is the name of the http header that carries the xsrf token value
  xsrfHeaderName: 'X-XSRF-TOKEN', // default

  // `onUploadProgress` allows handling of progress events for uploads
  // `onUploadProgress` 监听上传进度的事件
  onUploadProgress: function (progressEvent) {
    // Do whatever you want with the native progress event
  },

  // `onDownloadProgress` allows handling of progress events for downloads
  // `onDownloadProgress` 监听下载进度的事件
  onDownloadProgress: function (progressEvent) {
    // Do whatever you want with the native progress event
  },

  // `maxContentLength` defines the max size of the http response content allowed
  // `maxContentLength` 定义允许响应文件的最大值
  maxContentLength: 2000,

  // `validateStatus` defines whether to resolve or reject the promise for a given
  // HTTP response status code. If `validateStatus` returns `true` (or is set to `null`
  // or `undefined`), the promise will be resolved; otherwise, the promise will be
  // rejected.
  // `validateStatus` 定义resolve or reject 的 promise返回的HTTP response状态码
  validateStatus: function (status) {
    return status >= 200 && status < 300; // default
  },

  // `maxRedirects` defines the maximum number of redirects to follow in node.js.
  // `maxRedirects` 在node.js中定义要重定向的最大数量.
  // If set to 0, no redirects will be followed.
  maxRedirects: 5, // default

  // `httpAgent` and `httpsAgent` define a custom agent to be used when performing http
  // and https requests, respectively, in node.js. This allows options to be added like
  // `keepAlive` that are not enabled by default.

  // `httpAgent` and `httpsAgent`在node.js中分别执行 http 和 https 请求时定义要使用的自定义代理
  // 允许添加类似`keepAlive`的选项，默认情况下未启用。
  httpAgent: new http.Agent({ keepAlive: true }),
  httpsAgent: new https.Agent({ keepAlive: true }),

  // 'proxy' 定义代理服务器的主机名和端口
  // Use `false` to disable proxies, ignoring environment variables.
  // 使用`false`来禁用代理，忽略环境变量
  // `auth` indicates that HTTP Basic auth should be used to connect to the proxy, and
  // supplies credentials.
  // `auth`表示应该使用HTTP Basic auth连接到代理，提供凭证。
  // This will set an `Proxy-Authorization` header, overwriting any existing
  // `Proxy-Authorization` custom headers you have set using `headers`.
  // 这将设置一个 `Proxy-Authorization` 头, 覆盖任何现有的
  // 使用`header`设置的`Proxy-Authorization`自定义头文件。
  proxy: {
    host: '127.0.0.1',
    port: 9000,
    auth: {
      username: 'mikeymike',
      password: 'rapunz3l'
    }
  },

  // `cancelToken` 指定可以用于取消请求的取消token
  // (有关详情，请参阅下面的取消部分)
  cancelToken: new CancelToken(function (cancel) {
  })
}
```

## 响应模式

一个请求的响应中包含以下信息。

```js
{
  // `data` is the response that was provided by the server
  data: {},

  // `status` is the HTTP status code from the server response
  status: 200,

  // `statusText` is the HTTP status message from the server response
  statusText: 'OK',

  // `headers` the headers that the server responded with
  // All header names are lower cased
  headers: {},

  // `config` is the config that was provided to `axios` for the request
  config: {},

  // `request` is the request that generated this response
  // It is the last ClientRequest instance in node.js (in redirects)
  // and an XMLHttpRequest instance the browser
  request: {}
}
```

当使用 `then`, 您将收到如下的响应:

```js
axios.get('/user/12345')
  .then(function(response) {
    console.log(response.data);
    console.log(response.status);
    console.log(response.statusText);
    console.log(response.headers);
    console.log(response.config);
  });
```

当使用 `catch`, 或者通过一个 [rejection callback](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/then) 作为`then`的第二个参数, 响应将获取 `error` 对象在处理错误部分做出解释.

## 配置默认

您可以指定将应用于每个请求的配置默认值

### Axios全局默认值

```js
axios.defaults.baseURL = 'https://api.example.com';
axios.defaults.headers.common['Authorization'] = AUTH_TOKEN;
axios.defaults.headers.post['Content-Type'] = 'application/x-www-form-urlencoded';
```

### 自定义实例的默认值

```js
// Set config defaults when creating the instance
var instance = axios.create({
  baseURL: 'https://api.example.com'
});

// Alter defaults after instance has been created
instance.defaults.headers.common['Authorization'] = AUTH_TOKEN;
```

### 优先级配置顺序

将按优先顺序合并配置。 该顺序为库默认值在 `lib/defaults.js`里, then `defaults` property of the instance, and finally `config` argument for the request. The latter will take precedence over the former. Here's an example.

```js
// Create an instance using the config defaults provided by the library
// At this point the timeout config value is `0` as is the default for the library
var instance = axios.create();

// Override timeout default for the library
// Now all requests will wait 2.5 seconds before timing out
instance.defaults.timeout = 2500;

// Override timeout for this request as it's known to take a long time
instance.get('/longRequest', {
  timeout: 5000
});
```

## Interceptors

You can intercept requests or responses before they are handled by `then` or `catch`.

```js
// Add a request interceptor
axios.interceptors.request.use(function (config) {
    // Do something before request is sent
    return config;
  }, function (error) {
    // Do something with request error
    return Promise.reject(error);
  });

// Add a response interceptor
axios.interceptors.response.use(function (response) {
    // Do something with response data
    return response;
  }, function (error) {
    // Do something with response error
    return Promise.reject(error);
  });
```

If you may need to remove an interceptor later you can.

```js
var myInterceptor = axios.interceptors.request.use(function () {/*...*/});
axios.interceptors.request.eject(myInterceptor);
```

You can add interceptors to a custom instance of axios.

```js
var instance = axios.create();
instance.interceptors.request.use(function () {/*...*/});
```

## Handling Errors

```js
axios.get('/user/12345')
  .catch(function (error) {
    if (error.response) {
      // The request was made and the server responded with a status code
      // that falls out of the range of 2xx
      console.log(error.response.data);
      console.log(error.response.status);
      console.log(error.response.headers);
    } else if (error.request) {
      // The request was made but no response was received
      // `error.request` is an instance of XMLHttpRequest in the browser and an instance of
      // http.ClientRequest in node.js
      console.log(error.request);
    } else {
      // Something happened in setting up the request that triggered an Error
      console.log('Error', error.message);
    }
    console.log(error.config);
  });
```

You can define a custom HTTP status code error range using the `validateStatus` config option.

```js
axios.get('/user/12345', {
  validateStatus: function (status) {
    return status < 500; // Reject only if the status code is greater than or equal to 500
  }
})
```

## Cancellation

You can cancel a request using a *cancel token*.

> The axios cancel token API is based on the withdrawn [cancelable promises proposal](https://github.com/tc39/proposal-cancelable-promises).

You can create a cancel token using the `CancelToken.source` factory as shown below:

```js
var CancelToken = axios.CancelToken;
var source = CancelToken.source();

axios.get('/user/12345', {
  cancelToken: source.token
}).catch(function(thrown) {
  if (axios.isCancel(thrown)) {
    console.log('Request canceled', thrown.message);
  } else {
    // handle error
  }
});

// cancel the request (the message parameter is optional)
source.cancel('Operation canceled by the user.');
```

You can also create a cancel token by passing an executor function to the `CancelToken` constructor:

```js
var CancelToken = axios.CancelToken;
var cancel;

axios.get('/user/12345', {
  cancelToken: new CancelToken(function executor(c) {
    // An executor function receives a cancel function as a parameter
    cancel = c;
  })
});

// cancel the request
cancel();
```

> Note: you can cancel several requests with the same cancel token.

## Using application/x-www-form-urlencoded format

By default, axios serializes JavaScript objects to `JSON`. To send data in the `application/x-www-form-urlencoded` format instead, you can use one of the following options.

### Browser

In a browser, you can use the [`URLSearchParams`](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams) API as follows:

```js
var params = new URLSearchParams();
params.append('param1', 'value1');
params.append('param2', 'value2');
axios.post('/foo', params);
```

> Note that `URLSearchParams` is not supported by all browsers (see [caniuse.com](http://www.caniuse.com/#feat=urlsearchparams)), but there is a [polyfill](https://github.com/WebReflection/url-search-params) available (make sure to polyfill the global environment).

Alternatively, you can encode data using the [`qs`](https://github.com/ljharb/qs) library:

```js
var qs = require('qs');
axios.post('/foo', qs.stringify({ 'bar': 123 }));
```

### Node.js

In node.js, you can use the [`querystring`](https://nodejs.org/api/querystring.html) module as follows:

```js
var querystring = require('querystring');
axios.post('http://something.com/', querystring.stringify({ foo: 'bar' }));
```

You can also use the `qs` library.

## Semver

Until axios reaches a `1.0` release, breaking changes will be released with a new minor version. For example `0.5.1`, and `0.5.4` will have the same API, but `0.6.0` will have breaking changes.

## Promises

axios depends on a native ES6 Promise implementation to be [supported](http://caniuse.com/promises).
If your environment doesn't support ES6 Promises, you can [polyfill](https://github.com/jakearchibald/es6-promise).

## TypeScript
axios includes [TypeScript](http://typescriptlang.org) definitions.
```typescript
import axios from 'axios';
axios.get('/user?ID=12345');
```

## Resources

* [Changelog](https://github.com/mzabriskie/axios/blob/master/CHANGELOG.md)
* [Upgrade Guide](https://github.com/mzabriskie/axios/blob/master/UPGRADE_GUIDE.md)
* [Ecosystem](https://github.com/mzabriskie/axios/blob/master/ECOSYSTEM.md)
* [Contributing Guide](https://github.com/mzabriskie/axios/blob/master/CONTRIBUTING.md)
* [Code of Conduct](https://github.com/mzabriskie/axios/blob/master/CODE_OF_CONDUCT.md)

## Credits

axios is heavily inspired by the [$http service](https://docs.angularjs.org/api/ng/service/$http) provided in [Angular](https://angularjs.org/). Ultimately axios is an effort to provide a standalone `$http`-like service for use outside of Angular.

