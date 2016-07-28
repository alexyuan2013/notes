# 目录
1. [$broadcast $emit $on的用法](#broadcast-emit-on)
1. [$watch $apply $digest的用法](#watch-apply-digest)

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
document.getElementById("updateTimeButton).addEventListener('click', function(){
  $scope.$apply(function(){//在$apply()中更新时间，标签更新
    console.log("update time clicked");
    $scope.data.time = new Date();
  });
});
```
更多：
- [AngularJS $watch() , $digest() and $apply()](http://tutorials.jenkov.com/angularjs/watch-digest-apply.html)
- [Angularjs Developer Guide-Integration with the browser event loop](https://docs.angularjs.org/guide/scope)
## 