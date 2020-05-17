# Promise用法

## 简介

promise和ajax功能类似，都是用来做异步请求的。promise的优势在于可以链式编程，在response还没回应时，then操作中可以把正常流程代码实现，catch捕获异常（而且可以放在最后进行统一捕获），可以不断then 下去；  前后端分离大背景下，ajax是基于原生的xhr，已经不能很好的适应mvvm这种前端模型了。

## 如何申明

  本章主要是用来描述vue项目开发前端页面的，为何先介绍promise呢？因为vue核心组件始终是围绕着Promise展开的；如vuex.store的  actions 中方法 都是用Promise来实现的；后端api交互用的axios，也是promise封装的。  

先简单申明一个Promise对象；

```javascript
new Promise(   function (resolve, reject) { 

​    // 一段耗时的异步操作

​     resolve(data) // 数据处理完成 

​    // reject('失败') // 数据处理出错   

} ) 
```

网上有说构造Promise时 就一个参数，有的说两个参数，都没错； 如果说一个参数，这个参数就是一个函数，如果说两个参数，指的是这个函数有两个参数。

## 状态变化

promise状态说明，从初始化时pending——》到resolved或rejected（只能是二者中某一种状态），在未调用resolve或reject方法时，promise都是pending状态，一旦调用了resolve或reject方法 promise就结束了，promise状态为resolved或rejected；其构造函数中不能进行其他操作了，只能进行后续的then或catch操作了（then 、catch也是promise类型对象）。

所以Promise 生命流程 只有两种状态 ；  

正常的： pending状态（未调用resolve方法前）---->resolved状态；

或者失败的 ： pending状态（未调用rejecte方法前）-------> rejected状态



## 链式处理

resolved状态promise能使用then进行后续处理，rejected状态promise对象可以用catch进行处理；在调用方法resolve()改变promise状态时，可以传递参数，如resolve(data)（只能接收一个参数，多个参数想办法合并成数组），后续接着用then处理；then的构造器中可以直接使用传递过来的data；这样无限的then搞下去，参数就通过resolve方法来传递。如果异步请求获取对象不符合要求，判断后，rejecte方法来改变promise状态 也可以通过它传递参数，最终变成rejected状态的promise，这种状态promise能调用catch，在其构造器中进行后续异常捕获或抛出操作。





## 结合项目的登录和获取用户信息

登录过程

```javascript
 this.$store.dispatch('Login', this.loginForm).then(() => {
 //dispatch方法前面是指向store中actions中方法，后面是要传递的参数；
 //登录成功后 页面进行重定向
 this.$router.push({ path: this.redirect || '/' })
 this.loading = false
 }).catch(() => {
 this.loading = false
 }) 
```

actions中Login方法 接收到传递过来参数，然后返回的是Promise对象，这个Promise对象里面 有执行axios的login方法，axios 的login  如果成功，就传递resolve携带的值，如果失败就传递reject携带的值。

```javascript
 Login({ commit }, userInfo) {
 const { username, password } = userInfo
 return new Promise((resolve, reject) => {
 login({ username: username.trim(), password: password }).then(response => {
 const { data } = response
 commit('SET_TOKEN', data.token)
 setToken(data.token)
 resolve()
 }).catch(error => {
 reject(error)
 })
 })
 }, 
```

resolved状态Promise会被then捕捉，所以登录成功后执行then，then构造器里面有登录后页面重定向操作等； rejected状态可以被catch捕捉。

通过上面代码 ，可以总结出利用Promise 构造登录的异步请求

```
new  Promise((resovle,reject)=>{
login(username,password).then(
resp=>{state.aa = resp.data; resovle(param)})
.catch(err=> {各种操作，reject(param)})
}) 
```



login请求调用axios提供的api，返回类型本质是promise；  如果登录成功后变成resolved状态，可以被then捕获，then中可以把login成功后的数据response（就是返回体） 拿来用，用完后resolve方法改变状态（可以传参数）； 如果login失败，判断后置为rejected状态 并且传递相应数据，然后catch来处理。

rejected状态promise与catch链式处理示例：

```javascript
promise对象（有调用reject方法的）.catch(() => {
reject(new Error("请核对账号或密码"));
}).catch(err => {
	alert(err);
     this.loading = false;//loading状态停止
}); 
```



## 项目开发中注意事项

axios 调用接口成功后，返回的response.data 才是报文体（后端传递过来的参数）response是包含了报文、报文头、认证信息等。如果后端封装的报文体由code、msg 、data 组成，需要用response.data.data 才能取到想要的数据。