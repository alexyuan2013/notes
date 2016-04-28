- 2014-04-28——获取oracle数据表最新更新时间

解决方法来自stackoverflow的[问题](http://stackoverflow.com/questions/265122/how-to-find-out-when-an-oracle-table-was-updated-the-last-time)，简单来说，
就是通过通过如下语句实现：

```sql
SELECT SCN_TO_TIMESTAMP(MAX(ora_rowscn)) from myTable;
```

基本实现了在不用数据库触发器的情况下，通过定时执行以上语句，获取数据表更新的最新时间，从而执行相应的操作。
