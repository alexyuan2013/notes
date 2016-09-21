# 目录
1. [$broadcast $emit $on的用法](#broadcast-emit-on)
1. [$watch $apply $digest的用法](#watch-apply-digest)
1. [$q的用法](#q-promise)
1. [directive的用法](#directive)

<div id="broadcast-emit-on"></div>

## $broadcast $emit $on的用法

controller之间是层次关系，当在内层的controller中触发了一个事件，
需要在上层的controller中做出响应，由于作用域的隔离，在下层controller
中没办法完成上层controller的操作。angular给出的解决方案是通过$emit
向上层controller传递（向下层传递则使用$broadcast），而在目标controller
中监听到相应的$emit或$broadcast的事件，则需要使用$on方法。示例代码：

```javascript
angular.module('TestApp', ['ng'])
.controller('ParentCtrl', function($scope) {
 $scope.$on('dosomething', function(event,data){
	console.log('parent: do something ——' + data);	
 });
})
.controller('SelfCtrl', function($scope, $timeout) { 
  $scope.$emit('dosomething', 'bbb');
  $timeout(function(){$scope.$broadcast('dosomething', 'aaa')});
})
.controller('ChildCtrl', function($scope){
  $scope.$on('dosomething', function(event, data){
	  console.log('child: do something——' + data);
  });
})
;
```

```html
<body ng-app="TestApp">
    <div ng-controller="ParentCtrl">
      <div ng-controller="SelfCtrl">
        <div ng-controller="ChildCtrl">
          
        </div>
      </div>
    </div>
  <h1>Hello World!</h1>
</body>
```
> 注意：上面代码中，向下层$broadcast时，如果是在DOM生成的时就触发的话，
由于下层controller的DOM还没生成好，所以没有办法接受到上层的广播，使用$timeout
将其包裹，延迟广播，即可实现。

更多：
- [Stack Overflow](http://stackoverflow.com/questions/14502006/working-with-scope-emit-and-on)
- [Angularjs Developer Guide-Scope Events Propagation](https://docs.angularjs.org/guide/scope)


<div id="watch-apply-digest"></div>

## $watch $apply $digest的用法

Angularjs实现了双向绑定，所以大部分情况下，$watch()和$digest()都是angularjs自动调用的，
当我们想要关注某个变量的变化时，这时要用到$watch()方法：
```javascript
$scope.$watch(function(), function());
```
参数的第一个函数是个值函数，其必须有返回值，或者直接是一个变量亦可；
第二个函数为事件函数，用户处理监听到的变化，如下：
```javascript
$scope.$watch(
  function(scope){return scope.data.myValue;}, 
  function(newValue, oldValue){
    console.log(newValue);
  });
```

$digest()循环用户检查$scope中所有的对象，直到所有被监视的对象都未检测到变化时
（再多做10次检查），循环停止。当Angularjs认为有可以改变对象值的动作发生时，
如点击事件、ajax请求回调等，$digest()循环会自动触发，用于检测对象有没有发生变化。
但是，自动触发$digest()循环的事件必须在angularjs的框架下进行，当使用其他方式，
如直接通过getElementById操作DOM，并不会触发$digest()循环。这种情况下，
可以直接调用$digest()，也可以调用$apply()，其会自动触发$digest()。
```javascript
$scope.$apply(function(){
  $scope.data.myValue = "Another Value";
});
```
一个完整的例子：
```html
<div ng-controller="myController">
    {{data.time}}<br/>
    <button ng-click="updateTime()">update time - ng-click</button>
    <button id="updateTimeButton">update time</button>
</div>
<script>
    var module = angular.module("myapp", []);
    var myController1 = module.controller("myController", function($scope) {
 
        $scope.data = { time : new Date() };

        $scope.updateTime = function() {
            $scope.data.time = new Date();
        }

        document.getElementById("updateTimeButton")
                .addEventListener('click', function() {
            console.log("update time clicked");
            $scope.data.time = new Date(); //时间标签并没有更新
        });
    });
</script>
```
ng-click事件为angularjs框架内的事件，所以单击第一个按钮，时间标签会更新，
但是，第二个按钮是直接使用js绑定的事件，单击后时间标签并没有更新。
添加手动触发$digest()循环的代码，则时间标签会更新。
```javascript
document.getElementById("updateTimeButton")
        .addEventListener('click', function() {
    console.log("update time clicked");
    $scope.data.time = new Date();
    $scope.$digest(); //手动触发$digest()循环，标签更新
});
```
也可以使用$apply()：
```javascript
document.getElementById("updateTimeButton").addEventListener('click', function(){
  $scope.$apply(function(){ //在$apply()中更新时间，标签更新
    console.log("update time clicked");
    $scope.data.time = new Date();
  });
});
```
更多：
- [AngularJS $watch() , $digest() and $apply()](http://tutorials.jenkov.com/angularjs/watch-digest-apply.html)
- [Angularjs Developer Guide-Integration with the browser event loop](https://docs.angularjs.org/guide/scope)

<div id="q-promise"></div>

## $q的用法

JavaScript是单线程的，所以为了避免线程阻塞，其天生就是异步执行的。但有的时候我们需要等待一个动作的结果之后再执行另一个结果，
比如ajax请求之后根据返回的结果执行下一个动作，这个动作又可能是一个ajax请求。通常情况下，我们只能嵌套这些ajax请求，
在上一层的回调函数中嵌套执行ajax请求。这样的代码结构看起来很不清晰，更谈不上优雅。

于是，在angularjs中，引入了$q。$q是一个服务（service），它可以异步地执行方法，并且在完成处理后返回处理的结果。

### $q的构造

$q的构造方式有两种，一种是ES6的现代风格，一种是传统JS的一般风格：

```javascript
// for the purpose of this example let's assume that variables `$q` and `okToGreet`
// are available in the current lexical scope (they could have been injected or passed in).
function asyncGreet(name) {
  // perform some asynchronous operation, resolve or reject the promise when appropriate.
  return $q(function(resolve, reject) {
    setTimeout(function() {
      if (okToGreet(name)) {
        resolve('Hello, ' + name + '!');
      } else {
        reject('Greeting ' + name + ' is not allowed.');
      }
    }, 1000);
  });
}

var promise = asyncGreet('Robin Hood');
promise.then(function(greeting) {
  alert('Success: ' + greeting);
}, function(reason) {
  alert('Failed: ' + reason);
});
```
以上就是现代风格的构造方式，直接通过$q的构造函数生成对象。

```javascript
// for the purpose of this example let's assume that variables `$q` and `okToGreet`
// are available in the current lexical scope (they could have been injected or passed in).
function asyncGreet(name) {
  var deferred = $q.defer();

  setTimeout(function() {
    deferred.notify('About to greet ' + name + '.');

    if (okToGreet(name)) {
      deferred.resolve('Hello, ' + name + '!');
    } else {
      deferred.reject('Greeting ' + name + ' is not allowed.');
    }
  }, 1000);

  return deferred.promise;
}

var promise = asyncGreet('Robin Hood');
promise.then(function(greeting) {
  alert('Success: ' + greeting);
}, function(reason) {
  alert('Failed: ' + reason);
}, function(update) {
  alert('Got notification: ' + update);
});
```
传统的代码要相对好理解，首先是通过$q.defer()生成一个延时对象，然后再在延时函数中处理结果，
并通过延时对象的resolve和reject方法分别保存执行成功和失败的结果，最后返回延时的promise对象。
在外部使用promise对象，通过then()方法来获取相应的返回结果。个人比较倾向于使用这种构造方式。

### 延时对象deferred的接口

- 方法
  - resolve(value) 导出处理结果到promise对象
  - reject(reason) 导出处理失败原因到promise对象
  - notify(value) 提供promise对象当前的执行状态，可能会调用多次，至于其调用的频率，大概是一秒

- 属性
  - promise 延时对象关联的promise对象

### 许诺对象promise的接口

- 方法
  - then(successCallback, [errorCallback], [notifyCallback]) successCallback为必须的回调函数
  errorCallback和notifyCallback两个回调函数是可选的。
  - catch(errorCallback) 等同于then(null, errorCallback)
  - finally(callback, notifyCallback) 不管结果是resolve还是reject，均调用该方法

### promise对象的链式操作

通过then方法的调用，返回的依然是一个promise对象，因此promise具备了链式操作的功能：

```javascript
var details {  
   username: null,
   profile: null,
   permissions: null
};

$http.get('/api/user/name')
  .then(function(response) {
     // Store the username, get the profile.
     details.username = response.data;
     return $http.get('/api/profile/' + details.username);
  })
  .then(function(response) {
      //  Store the profile, now get the permissions.
    details.profile = response.data;
    return $http.get('/api/security/' + details.username);
  })
  .then(function(response) {
      //  Store the permissions
    details.permissions = response.data;
    console.log("The full user details are: " + JSON.stringify(details);
  });
  ```
上面$http.get()就是一个promise对象，链式操作依次获取了username, proifle, persmissions三个属性。

### $q.all()等待所有的promise对象，或promise链

有时候，我们需要等待所有的异步操作完成后，再执行某个动作，这个时候可以使用$q.all方法，
代码如下:

```javascript
app.controller('MainCtrl', function($scope, $q, $timeout) {
    //得到三个延时对象
    var one = $q.defer();
    var two = $q.defer();
    var three = $q.defer();  

    //对应的延时操作
    $timeout(function () {
        one.resolve("one done");
    }, Math.random() * 1000);

    $timeout(function () {
        two.resolve("two done");
    }, Math.random() * 1000);

    $timeout(function () {
        three.resolve("three done");
    }, Math.random() * 1000);

    //使用$q.all来监听所有的promise执行
    $q.all([one.promise, two.promise, three.promise]).
      then(function() {
        console.log("ALL INITIAL PROMISES RESOLVED");
    });

    //成功的回调函数
    function success(data) {
        console.log(data);
        return data + "Chained";
    }

    //定义三个promise链
    var oneChain = one.promise.then(success).then(success);
    var twoChain = two.promise.then(success);
    var threeChain = three.promise.then(success).then(success).then(success);
    //$q.all来监听所有的promise链
    $q.all([oneChain, twoChain, threeChain]).
      then(function(){
        console.log("All PROMISES CHAIN RESOLVED");
    });

});
```

更多：
- [Angularjs API Reference - $q](https://docs.angularjs.org/api/ng/service/$q)
- [Promises in angularjs - The definitive guid](http://www.dwmkerr.com/promises-in-angularjs-the-definitive-guide/)
- [Stackoverflow - Wait for all promises to resolve](http://stackoverflow.com/questions/21759361/wait-for-all-promises-to-resolve)


<div id="directive"></div>

## Directive的用法

### Directive是干嘛的

Directive作为对DOM元素（包括属性、元素、css类、注释等）的一种标签，告诉Angularjs的HTML
compiler如何将特定的行为附加到对应的DOM元素或者其子元素上。说白了就是对HTML元素的一种扩展。

Angularjs有许多内建的的directive，例如ngBind、ngModel、ngClass等，你也可以像创建controller、
service那样创建自己的directive。当Angular启动应用时，HTML编译器会遍历所有匹配DOM的directive，
并将其与相应的DOM元素对应。

### 匹配Directive

在我们编写directive之前，需要知道HTML编译器如何决定何时使用某个给定的directive。

与元素选择器类似，当directive作为元素声明的一部分时，我们称这个元素与directive相匹配。

下面的两个例子中，我们称\<input\>元素于ngModel指令相匹配：
```html
<input ng-model="foo">
<input data-ng-model="foo">
```
下面的例子中，元素于person指令相匹配：
```html
<person>{{name}}</person>
```

### 归一化处理

Angular会将与指令相匹配的元素的标签名和属性名进行归一化处理，我们之前提到的几个指令，
如ngModel，就是经过归一化处理后的结果，即为大小写敏感的驼峰命名法。然而，HTML是大小写不敏感的，
因此，我们在DOM里边匹配指令的时候一般是写成小写的形式，并且用“-”作为连接符，如ng-model。

归一化处理的一般步骤：

- 从元素或属性名中，删除`x-`或者`data-`的前缀
- 将`:`、`-`、`_`转换成驼峰命名法的形式  

以下集中写法是等价的，均匹配了ngBind指令：
```html
<div ng-controller="Controller">
  Hello <input ng-model='name'> <hr/>
  <span ng-bind="name"></span> <br/>
  <span ng:bind="name"></span> <br/>
  <span ng_bind="name"></span> <br/>
  <span data-ng-bind="name"></span> <br/>
  <span x-ng-bind="name"></span> <br/>
</div>
```

> 最佳实践：以上集中形式，我们推荐使用`-`连接的形式，即ng-bind来匹配ngBind，
如果你使用了HTML语法检查工具，则需要加上`data-`前缀，即data-ng-bind来匹配ngBind。
其他集中形式虽然在语法上是符合的，但是我们并不推荐使用。

### Directive的类型

`$compile`可以根据元素名、属性、类名以及注释来匹配相应的的指令。所有Angular提供的内部指令，都可以通过属性名、标签名、
类名和注释来匹配，示例如下：

```html
<my-dir></my-dir>
<span my-dir="exp"></span>
<!-- directive: my-dir exp -->
<span class="my-dir: exp;"></span>
```

> **最佳实践**：使用标签名和属性名来匹配directive，而不是注释和类名，这样匹配显得更加清晰。

> **最佳实践**：用注释匹配用在DOM创建directive受限时，如跨越多个元素(比如\<table\>元素)。 
AngularJS 1.2 引入了ng-repeat-start和ng-repeat-end指令，作为更好的解决方案。 
建议开发者使用这种方式，而不要用“自定义注释”形式的指令。

### 创建指令

首先来看看指令注册的API，同controller一样，directive也注册在module之上，即必须使用module.directive
API，该接口接收一个归一化的指令名和一个工厂方法。这个工厂方法要返回一个带有不同配置的object，
用于告诉`$compile`指令的具体行为是怎样的。

工厂方法只在`$compile`配置指令时调用一次，你可以在这里做一些初始化的工作。该函数使用`$injector.invoke`调用，所以它可以像控制器一样进行依赖注入。

> **最佳实践**：尽量返回一个对象，而不是仅仅返回一个函数。

> **最佳实践**：为了避免指令名与内建指令或者HTML的标签名重复，最好给指令名加上前缀，
两到三字符就可以，但是同样记得不要使用ng作为前缀，这样可能会和Angular未来的扩展冲突。
最简单的，可以使用my最为前缀。

### 用模板来扩展指令

假设你有大量表示用来呈现用户信息的模板，而且会重复出现在多个地方，一旦你修改了某一处的代码，
你就需要将所有的用到它的地方全部修改一遍，这时，就是指令使用的一个很好的机会。

你可以将HTML模板直接写在指令的代码中，也可以通过templateUrl来加载.html文件，
建议使用后者，除非你的模板代码很简单。

```javascript
angular.module('docsTemplateUrlDirective', [])
.controller('Controller', ['$scope', function($scope) {
  $scope.customer = {
    name: 'Naomi',
    address: '1600 Amphitheatre'
  };
}])
.directive('myCustomer', function() {
  return {
    restrict: 'E',
    scope: {
      customerInfo: '=info'
    },
    templateUrl: 'my-customer.html'
  };
});
```
restrict选项用来设置指令匹配的对象，即：

- 'A' - 仅匹配属性
- 'E' - 仅匹配标签
- 'C' - 仅匹配类名
- 'M' - 仅匹配注释
- 'AEC' - 匹配属性、标签或类名

> 何时使用元素标签及属性名？使用元素标签创建指令时，表示你有自己定义的HTML模板，
而如果你的目的仅仅是扩展一个标签的行为则最好使用属性名来匹配。

### 作用域隔离

在未配置的情况下，指令使用的变量的作用域为其所在的controller的作用域，如果想在同一个
controller中使用多个相同的指令，则必须要将指令的作用域与外界的controller的作用隔离。

使用`scope: {customerInfo: '=info'}`或其简写`scope：{customerInfo: '='}`将指令的作用域与外部隔离。
不使用作用域隔离，指令将继承父类的作用域，使用作用域隔离可以避免除传入其中的数据模型之外的其他任何操作对指令内部的行为的影响。

### 创建操作DOM的指令

指令通过`link`选项来注册DOM监听和更新DOM，`link`函数在模板复制好后执行，指令的代码逻辑应该放到这边。
`link`的定义如下：
`function link(scope, element, attrs, controller, transcludeFn) { ... }`：

- `scope` - Angular的作用域对象
- `element` - 指令匹配的jqLite封装的元素(angular内部实现的类jquery的库)
- `attrs` - 归一化处理过的属性名与值的键值对
- `controller` - 传入的controller对象或者自身所在的controller对象
- `transcludeFn` - 转制函数，预先将指令绑定到正确的转制作用域

### 创建包含其他元素的指令


### 给指令添加事件监听



### 指令之间的通信

### 总结

### 更多

- [Angularjs Developer Guide - Directive](https://docs.angularjs.org/guide/directive)
- [Angularjs API - $compile](https://docs.angularjs.org/api/ng/service/$compile)










