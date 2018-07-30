---
title: 为angular的$http服务添加拦截器
date: 2018-07-28 20:57:58
tags: [AngularJs, http拦截器, 技术]
---

# 什么是拦截器–What are Interceptors?

$httpProvider 中有一个 interceptors 数组，而所谓拦截器只是一个简单的注册到了该数组中的常规服务工厂。
Interceptor(拦截器)在服务端框架中属于一种比较常见的机制,如 spring,Struts2 等 Java 框架中拦截器属于基本配置项.拦截器提供了一种机制可以使开发者可以定义在一个 action(动作)执行的前后执行的代码，这些代码可以是在一个 action 执行前阻止其执行的代码,也可以是修改目标动作某些行为的代码.(在 AOP（Aspect-Oriented Programming）中拦截器用于在某个方法或字段被访问之前，进行拦截然后在之前或之后加入某些操作。在 Spring 框架中比较常见。

<!--more-->

# $http 服务中的拦截器

查看 API 或是源码我们可以发现,Angular 的实现中通过\$httpProvider 提供了一个名为 interceptors 的数组.这个数组接受拦截器服务注册,通过过程次的注册最终会形成拦截器链.这样每次在调用\$http 服务的时候,angular 都会通过定义的拦截点(切面)进行相应的 Ajax 动作修改.

![](https://ws1.sinaimg.cn/large/0064OUUqly1fnty1o6bj2j30ms0hl40v.jpg)

现在要做的就是为每一次\$http 请求添加 token 信息，方便后台进行认证拦截。

## Angular 中如何声明一个拦截器

```javascript
// Interceptor declaration
module.factory('httpInterceptor', [
  '$log',
  function($log) {
    $log.debug(
      '$log is here to show you that this is a regular factory with injection'
    );

    return {
      // do something
    };
  }
]);
```

## 如何将声明的拦截器注册到$http 服务中

```javascript
// Add the interceptor to $httpProvider.interceptors
module.config([
  '$httpProvider',
  function($httpProvider) {
    $httpProvider.interceptors.push('httpInterceptor');
  }
]);
```

## $http 拦截点

\$http 服务的拦截器编写与添加.但是上面的代码片段并不能实际使用,要想真正的实现拦截操作,我们还需要遵循 http 服务暴露出来的可以进行拦截的点进行相关的代码编写.
每个拦截器都可以实现 4 个可选的处理函数，分别对应请求（成功/失败）和响应（成功/失败）的拦截

- request : request 方法可以实现拦截请求: 该方法会在 http 发送请求到服务器之前执行，因此我们可以在该方法的视线中修改配置或做其他的操作。该方法接收请求配置对象(requestconfigurationobject)作为参数，然后必须返回配置对象或者 promise。如果返回无效的配置对象或者 promise 则会被拒绝，导致 http 调用失败。
- requestError : requestError 方法可以实现拦截请求异常: 有时候一个请求发送失败或者被拦截器拒绝了。请求异常拦截器会俘获那些被上一个请求拦截器中断的请求。它可以用来恢复请求或者有时可以用来撤销请求之前所做的配置，比如说关闭进度条，激活按钮和输入框什么之类的。
- response : response 方法可以实现拦截响应: 该方法会在 http 接收到从服务器过来的响应之后执行，因此我们可以修改响应报文或做其他操作。该方法接收响应对象(responseobject)作为参数，然后必须返回响应对象或者 promise。响应对象包括了请求配置(requestconfiguration)，头(headers)，状态(status)和从后台过来的数据(data)。如果返回无效的响应对象或者 promise 会被拒绝，导致 http 调用失败。
- responseError : responseError 方法可以实现拦截响应异常: 有时候我们后台调用失败了。也有可能它被一个请求拦截器拒绝了，或者被上一个响应拦截器中断了。在这种情况下，响应异常拦截器可以帮助我们恢复后台调用。

代码规范：

```JavaScript
myApp.factory('MyInterceptor', function($q) {
	return {
    	// 可选，拦截成功的请求
    	request: function(config) {
      		// 进行预处理
      		// ...
      		return config || $q.when(config);
  		},

    	// 可选，拦截失败的请求
    	requestError: function(rejection) {
      		// 对失败的请求进行处理
      		// ...
      		if (canRecover(rejection)) {
      			return responseOrNewPromise
      		}
      		return $q.reject(rejection);
  		},

    	// 可选，拦截成功的响应
    	response: function(response) {
      		// 进行预处理
      		// ....
      		return response || $q.when(reponse);
  		},

    	// 可选，拦截失败的响应
    	responseError: function(rejection) {
      		// 对失败的响应进行处理
      		// ...
      		if (canRecover(rejection)) {
      			return responseOrNewPromise
      		}
      		return $q.reject(rejection);
  		}
	};
});
```

对于上面暴露出来的接口使用也是异常的简单的,我们可以像声明一个简单的服务一样声明一个$http 服务的拦截器,并交由 angular 的注入机制去使用我们配置的拦截器:

    myApp.config(function($httpProvider) {
    $httpProvider.interceptors.push(MyInterceptor);
    });

---

# 实战代码示例

## 配置

    angular.module('houseApp')  
    .config(["$stateProvider","$httpProvider",function ($stateProvider, $ionicConfigProvider,$httpProvider) {  
        //禁用所有缓存  
        //$ionicConfigProvider.views.maxCache(0);  
        //添加拦截器  
        $httpProvider.interceptors.push('sessionInteceptor');  

    })]);  

## 拦截器

```JavaScript
/**
* 查询条件服务  
*/  
angular.module('houseApp')  
   .factory('sessionInteceptor', ["WAP_CONFIG","$q","userInfoService",function(WAP_CONFIG,$q,userInfoService) {  

       var myInterceptor = {};  

       //该方法接收请求配置对象(request configuration object)作为参数，然后必须返回配置对象或者 promise 。  
       myInterceptor.request = function(requestConfig){  
           console.log("myInterceptor.request userInfoService.getUserKey(): " + userInfoService.getUserKey());  
           //为每一个请求添加token，每个请求都合法登录  
           if(requestConfig["data"] != "" && requestConfig["data"] != null && requestConfig["data"] != undefined ){  
               requestConfig["data"]["token"] = userInfoService.getUserKey();  
           }  
           return requestConfig;  
       };  

       //该方法接收响应对象(response object)作为参数，然后必须返回响应对象或者 promise。  
       myInterceptor.response = function(responseObject){  
           //判断服务器响应是否为999，如果是则表示没有登录  
           if(responseObject.data.status == 999){  
               //window.location.href = "/wap/tmpl/member/login.html";  
               console.log("responseObject.data.status == 999");  
           }  
           return responseObject;  
       };  

       myInterceptor.requestError  = function(rejectReason){  
           var deferred = $q.defer();  

           console.log("myInterceptor.requestError : " + responseObject);  
           return deferred.promise;  
       };  

       myInterceptor.responseError = function(responseError){  
           console.log("myInterceptor.responseError : " + responseObject);  
           return {};  
       };  

       return myInterceptor;  

	}]);
```

---

# 异步操作

部分场景下,我们希望在拦截器中能够执行一些异步操作.然后依据不同的处理结果进行不同的拦截操作,AngularJS 在设计的时候也很好的支持了这一特性.AngularJS 允许我们在拦截的方法中,我们可以返回一个 promise 对象.如在 http 服务中,我们如果返回一个 promise 对象时,http 服务将会延迟发起网络请求或是延迟响应请求结果

```JavaScript
module.factory('myInterceptor', ['$q', 'someAsyncService', function($q, someAsyncService) {
    var requestInterceptor = {
        request: function(config) {
            var deferred = $q.defer();
            someAsyncService.doAsyncOperation().then(function() {
                // Asynchronous operation succeeded, modify config accordingly
                ...
                deferred.resolve(config);
            }, function() {
                // Asynchronous operation failed, modify config accordingly
                ...
                deferred.resolve(config);
            });
            return deferred.promise;
        },
        response: function(response) {
                    var deferred = $q.defer();
                    someAsyncService.doAsyncOperation().then(function() {
                        // Asynchronous operation succeeded, modify response accordingly
                        ...
                        deferred.resolve(response);
                    }, function() {
                        // Asynchronous operation failed, modify response accordingly
                        ...
                        deferred.resolve(response);
                    });
                    return deferred.promise;
                }
    };

    return requestInterceptor;
    }]);
```

---

#参考博客文章
《为 angular 内置$http 服务添加拦截器》：

http://blog.csdn.net/shenlei19911210/article/details/52804004

《angular 拦截器每个请求传递用户的 token》：

http://hbiao68.iteye.com/blog/2310578

《Angular $http 拦截器介绍与使用》：

http://blog.csdn.net/u010730126/article/details/51770946
