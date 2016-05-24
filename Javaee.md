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

 
