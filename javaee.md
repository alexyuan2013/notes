- 2016-04-28——获取oracle数据表最新更新时间

Google搜索关键字`oracle get table update time`

解决方法来自stackoverflow的[问题](http://stackoverflow.com/questions/265122/how-to-find-out-when-an-oracle-table-was-updated-the-last-time)，简单来说，就是通过通过如下语句实现：

```sql
SELECT SCN_TO_TIMESTAMP(MAX(ora_rowscn)) from myTable;
```

基本实现了在不用数据库触发器的情况下，通过定时执行以上语句，获取数据表更新的最新时间，从而执行相应的操作。

2016-05-19 补充：

上面的语句对长时间不更新的表会报错，这个在上面的问题回答中已经提到，只是当时没注意，具体的解决方式暂时没看明白，把原回答贴到下边：

>Works as long as last update to your table hasn't been too long ago. Else you get an error, so it's safest to first do a: left join sys.smon_scn_time tiemposmax on myTable.ora_rowscn <= tiemposmax.scn and then apply SCN_TO_TIMESTAMP to your table's ora_rowscn if and only if there's a match. Otherwise you can display the SCN_TO_TIMESTAMP(MIN(SCN)) from sys.smon_scn_time as the earliest date the table was modified before.

不过对于项目所要实现的功能没有太大的影响，因为要取的是更新的最新时间，长时间不更新，
上面语句就会报异常，而在抛出异常时，不去更新最新时间就可以了，异常直接忽略掉。

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

中文乱码：

> 需要设置plsql字符集，plsql默认加载的是windows系统变量的nls_lang的字符集，所以去我的电脑中，右键选择“属性”，再选择“系统高级设置”，再
> 选择“环境变量”，再选择“系统变量”，新建或者修改NLS_LANG</br>
> 变量名：NLS_LANG</br>
> 变量值：SIMPLIFIED CHINESE_CHINA.ZHS16GBK</br>
