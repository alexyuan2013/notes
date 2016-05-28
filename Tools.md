- 2016-05-28 Chrome调试Android应用功能

缘起为Ionic项目调试过程中，由于存在一些Cordova插件，需要调用原生的API，因此只能在真机上测试。但是，一旦出现bug，调试过程相对于浏览器调试，
就显得将异常繁琐，只能用alert函数打印相关信息。

然后，也就是一直这样调试下去，也没人去探索一个更便捷的方式，直到有人发现了Chrome浏览器居然有这样的功能。而且操作很简单，在Chrome浏览器中输入：
`chrome://inspect`即可，同时手机端在开发者模式中，设置调试应用，这样就可以将手机端应用的调试信息投射到Chrome浏览器中，同时还是实现了操作的相互性，
断点设置，调试信息完全和浏览器端一致，很给力的工具。官方文档在[这里](https://developers.google.com/web/tools/chrome-devtools/debug/remote-debugging/remote-debugging?hl=en)。
