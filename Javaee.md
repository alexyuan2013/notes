- 2016-05-12——获取文件的MD5值

[StackOverflow相关问题](http://stackoverflow.com/questions/304268/getting-a-files-md5-checksum-in-java)

由于自己是在spring framework框架下实现的，代码中用到了`DigestUtils.md5DigestAsHex(byte [])`方法，
具体代码如下
```java
public static String getMD5OfFile(File file){
  String value = "";
  FileInputStream fis = null;
  try {
    fis = new FileInputStream(file);
    MessageDigest md5 = MessageDigest.getInstance("MD5");
    byte[] buffer = new byte[1024];
    int numRead;
    do {
      numRead = fis.read(buffer);
      if(numRead > 0){
        md5.update(buffer, 0, numRead);
      }
    } while (numRead != -1);
    value = DigestUtils.md5DigestAsHex(md5.digest());
  } catch (Exception e){
    e.printStackTrace();
  } finally {
    if (null != fis){
      try {
        fis.close();
      } catch (Exception e){
        e.printStackTrace();
      }
    }
  }
  return value;
}
```
- 2016-06-01——websocket "x-webkit-deflate-frame is not supported"错误解决

在较早的版本的浏览器，对websocket的支持还是停留在较早的规范，而"x-webkit-deflate-frame"
在最新的标准中已经被弃用了，所以服务端一般会选择忽略这个头，但是spring支持的websocket
并不会忽略，而是抛出异常，即出现"The extension x-webkit-deflate-frame is not supported"
的异常。

解决的方式方法，就是在服务端握手时，对x-webkit-deflate-frame的头强制转化，转为permessage-deflate
即可。

具体解决方法及websocket的创建，可以参见之前创建的[websocket示例](https://github.com/alexyuan2013/javaee-demos/blob/master/spring-mvc-websockets-master/README.md)
 
参考资料：

1. [https://spring.io/blog/2014/09/16/preview-spring-security-websocket-support-sessions](https://spring.io/blog/2014/09/16/preview-spring-security-websocket-support-sessions)
2. [http://46aae4d1e2371e4aa769798941cef698.devproxy.yunshipei.com/linlzk/article/details/50153745](http://46aae4d1e2371e4aa769798941cef698.devproxy.yunshipei.com/linlzk/article/details/50153745)

