---
layout: post
title: 'uni-app 封装网络请求uni.request()并实现同步'
date: 2020-04-18
color: '#c8d6e5'
tags: 技术 uni-app
---
uni-app自带的API`uni.request()`可以获取接口的数据，但是直接使用无法完成数据在页面渲染的目的。查阅资料发现是因为网络请求是**异步操作**（因为网络操作的响应时间是不定的，无法直接按照我们的代码序列顺序执行）。

要想达成在页面渲染前获取数据的目的，需要进行**同步化操作**（把异步操作变成同步）。此外为了能在各个页面都方便地使用这个请求，还需要进行一个**封装并挂载到全局**的操作
## 需要的步骤
1. 新建js文件进行操作的封装，并在main.js中挂载全局
2. 通过async& await进行同步化操作

### JS文件配置

1. 建立.js文件，路径：`/components/my-request/api.js`
3. 主体分析
   * 因为要用async& await 进行同步化操作，而`await`后面只能跟`Promise`数据类型，所以要返回一个`Promise`数据
   * 因为不同的接口地址不同，传给接口的数据也有所不同，所以统一用一个变量`options`来表示（如`option.url`表示接口url）
   * 主体代码如下，视情况还可以再加上显示加载框的方法，目前展示的应该是最简单的封装（fail函数其实也可以删除）

```js
export const myRequest = (options) =>{ // 这是一个箭头函数，options是传入的参数
  return new Promise((resolve, reject)=>{ // 返回一个Promise对象
	  // 封装主体：网络请求API
    uni.request({
      url: options.url,
      data: options.data || {},
      method: options.method || 'GET',// 默认值GET，如果有需要改动，在options中设定其他的method值
      success: (res) => {
      	console.log(res.data);		// 控制台显示数据信息
      	resolve(res)
      },
      fail: (err) =>{
      	// 页面中弹框显示失败
      	uni.showToast({
          title: '请求接口失败'
      	})
      	// 返回错误消息
      	reject(err)
      }
    })
  }
  )
}
```

### main.js配置

添加代码

```js
// 接收api.js下的myRequst方法
import { myRequest } from './components/my-request/api.js'
// 挂载到全局，让所有的页面都能调用myRequest方法
Vue.prototype.$myRequest = myRequest
```

### 在页面中使用封装好的方法

```html
<script>
  export default {
    data() {
      return {
        backendData:[]
      }
    },
    onLoad() {
      this.initPage();
    },
    methods: {
      async initPage(){
        const res = await this.$myRequest({
          url: '接口地址',
          data: {
            // 接口的请求参数，可以为空
            }
        })
        // 给页面的数据赋值
        this.backendData =res.data;
      }
    }
  }
  // ...
</script>
```

代码分析：
1. 在`method`中新建一个方法，取名为`intitPage`，实现以下两个功能
   * 从接口获取数据`res`
   * 用`res`初始化页面数据`backendData`
1. async& await的使用
   * 语法
   ```js
   async 函数名 (){
     await 需要同步的方法（必须是Promise类型）
   }
   ```
   * 功能：实现同步，`await`后的方法return了才执行下一条语句
2. 数据获取应该在页面渲染之前，所以我们在`onLoad()`函数（页面加载时执行的函数）中执行`intitPage`
3. `res`的属性有很多，通过查看`res`，发现我们需要的值在`res.data`中，所以给`backendData`赋的值是`res.data`（注：有时需要的数据在`res.data.data`中，这里到底用几个data可以通过控制台查看`res`的值来判断）

## 其他的点

1. `uni.request()`的返回值是`res`，封装好后可以直接设一个变量`res = request（封装后）`来获得数据
5. 可以把要给接口的数据做成一个变量，方便修改
```html
<script>
  var data2backEnd {
    url: '',
    data: {

    },
    method: 'POST'
  }
  ...
        async initPage(){
        const res = await this.$myRequest(data2backEnd)
      }
  ...
```
