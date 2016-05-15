- 2016-04-28——获取oracle数据表最新更新时间

Google搜索关键字`oracle get table update time`

解决方法来自stackoverflow的[问题](http://stackoverflow.com/questions/265122/how-to-find-out-when-an-oracle-table-was-updated-the-last-time)，简单来说，就是通过通过如下语句实现：

```sql
SELECT SCN_TO_TIMESTAMP(MAX(ora_rowscn)) from myTable;
```

基本实现了在不用数据库触发器的情况下，通过定时执行以上语句，获取数据表更新的最新时间，从而执行相应的操作。

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
- 2016-05-12——PL/SQL配置备忘

首先随便下个PL/SQL的绿色版，然后记得下载一个Oracle的客户端，即[Instant Client](http://www.oracle.com/technetwork/topics/winsoft-085727.html)，解压即可用。

首次打开PL/SQL，先不点击登陆，直接点取消，进入软件，在`Tools/Preferences`中配置Client的解压路径到如下项目：

> Oracle Home(empty is autodetect)->D:\instantclient_11_2 

> OCI library(empty is autodetect)->D:\instantclient_11_2\oci.dll 

最后再次启动PL/SQL，填入用户名，密码，数据库地址及实例即可。