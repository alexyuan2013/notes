- 2016-05-28 Chrome调试Android应用功能

缘起为Ionic项目调试过程中，由于存在一些Cordova插件，需要调用原生的API，因此只能在真机上测试。但是，一旦出现bug，调试过程相对于浏览器调试，
就显得将异常繁琐，只能用alert函数打印相关信息。

然后，也就是一直这样调试下去，也没人去探索一个更便捷的方式，直到有人发现了Chrome浏览器居然有这样的功能。而且操作很简单，在Chrome浏览器中输入：
`chrome://inspect`即可，同时手机端在开发者模式中，设置调试应用，这样就可以将手机端应用的调试信息投射到Chrome浏览器中，同时还是实现了操作的相互性，
断点设置，调试信息完全和浏览器端一致，很给力的工具。官方文档在[这里](https://developers.google.com/web/tools/chrome-devtools/debug/remote-debugging/remote-debugging?hl=en)。


- 2016-08-18 移动端https自签名证书的导入

项目需要将http接口全部转为https的接口，首选要用自签名的证书去测试，在pc端是没有问题的，只要在浏览器中导入自签名的证书即可。
而到移动端，android和ios系统实际上是从系统层面上加入了https的签名证书，这些证书都是预置在系统中的可信任机构办法的。
对于自签名的证书，要想进行测试，就需要将其导入到系统的证书中去。

### 向android系统添加自签名证书

  - Open Firefox (I suppose it's also possible with Chrome, but it's easier for me with FF)
  - Visit your development site with a self-signed SSL certificate.
  - Click on the certificate (next to the site name)
  - Click on "More information"
  - Click on "View certificate"
  - Click on "Details"
  - Click on "Export..."
  - Choose "X.509 Certificate whith chain (PEM)", select the folder and name to save it and click "Save"
  - Copy the .crt file to the root of the /sdcard folder inside your Android device
  - Inside your Android device, Settings > Security > Install from storage. It should detect the certificate and let you add it to the device
  - Browse to your development site. The first time it should ask you to confirm the security exception. That's all. The certificate should work with any browser installed on your Android (Browser, Chrome, Opera, Dolphin...)

就是将自签名的证书拷贝到android系统的更目录，然后到设置/安全/从SD卡安装证书（国产定制系统中可能在更多设置中寻找）即可。

参考：[Add self signed SSL certificate to Android](https://coderwall.com/p/wv6fpq/add-self-signed-ssl-certificate-to-android-for-browsing)

### 向ios系统添加自签名证书

You can add an SSL certificate to the trusted list in iOS by simply emailing the file to yourself as an attachment:
![](http://blog.httpwatch.com/wp-content/uploads/2013/12/email_cert.png)

Then select Install to add the certificate. Once you’ve done this you use the certificate without warnings in Safari or other iOS apps that use the device’s keychain..

Also unlike Safari SSL exceptions, you can access the certificate at any time in Settings->General->Profiles and remove it if required:

![](http://blog.httpwatch.com/wp-content/uploads/2013/12/trusted_cert.png)

Apple provides an iPhone configuration utility for Mac and PC that can also install certificates. This would be a better option where email is not available or you have a larger number of iOS devices to manage.

参考：[Five Tips for Using Self Signed SSL Certificates with iOS](https://blog.httpwatch.com/2013/12/12/five-tips-for-using-self-signed-ssl-certificates-with-ios/)