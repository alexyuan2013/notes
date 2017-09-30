MongoDB 学习笔记
===
## 安装配置

### Windows

- 到[官方](http://www.mongodb.org/downloads?_ga=2.262303692.1165601754.1506754157-1835465429.1503549167&_gac=1.24965192.1505209308.Cj0KCQjwi97NBRD1ARIsAPXVWWA7aYKp-I4kGon3K-kuY3A38ZACI6iGeTJclqqikhlSuXBHZYknUsUaAoL9EALw_wcB)去下载windows版的mongodb server就可以了，默认安装。

- 一般配置

1. 最好是配置一下环境变量，将`mongodb`加入`path`路径下：`C:\Program Files\MongoDB\Server\3.4\bin\` 
2. 创建数据库目录，默认是安装目录盘（如`C:` ）的根目录下：`\data\db`，如：`C:\data\db`；或者也可以自定义目录，然后在启动mongodb的时候加入`--dbpath`参数即可，如：`mongod --dbpath d:\test\mongodb\data`
3. 启动`mongodb`，在命令行中执行：`mongod`即可
4. 连接`mongodb`，执行`mongo`即可

- 配置为windows服务

1. 以管理员身份启动命令行（cmd,  Ctrl + Shift + Enter）
2. 创建数据库和日志目录：
```
mkdir c:\data\db
mkdir c:\data\log
```
3. 在安装目录创建一个配置文件，如：`C:\Program Files\MongoDB\Server\3.4\mongod.cfg`，文件中至少包含`systemLog.path`，如：
```
systemLog:
    destination: file
    path: c:\data\log\mongod.log
storage:
    dbPath: c:\data\db
```
4. 安装为服务
```
mongod --config "C:\Program Files\MongoDB\Server\3.4\mongod.cfg" --install
```
5. 启动服务
```
net start MongoDB
```
6. 停止或移除服务
```
net stop MongoDB

mongod --remove
```
- 手动配置为windows服务
1. 以管理员身份启动命令行
2. 创建数据库和日志目录
3. 在安装目录创建一个配置文件
4. 创建服务
```
sc.exe create MongoDB binPath= "\"C:\Program Files\MongoDB\Server\3.4\bin\mongod.exe\" --service --config=\"C:\Program Files\MongoDB\Server\3.4\mongod.cfg\"" DisplayName= "MongoDB" start= "auto"
```
5. 启动服务
6. 停止或移除服务
```
net stop MongoDB

sc.exe delete MongoDB
```
### Ubuntu

具体安装参见[官方文档](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/)，这里需要知道三个路径：
```
/var/lib/mongodb -- 数据库实例
/var/log/mongodb -- 日志文件
/etc/mongod.conf -- 配置文件
```

### 使用Binary文件安装 -- 不推荐使用这种方式安装，除非Linux无法使用报管理器安装
具体参见[官方文档](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-linux/)
1. 下载文件
```
curl -O https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-3.4.9.tgz
```
2. 解压
```
tar -zxvf mongodb-linux-x86_64-3.4.9.tgz
```
3. 移动到想要的安装目录
```
mkdir -p mongodb
cp -R -n mongodb-linux-x86_64-3.4.9/ mongodb
```
4. 添加目录到环境变量
```
export PATH=<mongodb-install-directory>/bin:$PATH
```
5. 后面的使用可以参照windows的安装配置


## 安全配置

### 绑定IP白名单
首先可以通过绑定IP白名单的方式来控制客户端的访问权限，这个通过修改配置文件来实现，以Ubuntu为例，找到`/etc/mongod.conf`，配置如下：
```
net:
  port: 27017
  bind: [127.0.0.1, 192.168.221.4]
```
这样就是有指定下的IP才可以访问数据库，如果不需要绑定IP白名单，则将这个配置注销即可

### 开启身份认证
1. 正常启动mongodb，不开启身份认证：`mongod`
2. 连接到mongodb：`mongo`
3. 创建管理员用户，分配`userAdminAnyDatabase`角色，例如：
```shell
use admin
db.createUser(
  {
    user: "myUserAdmin",
    pwd: "abc123",
    roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
  }
)
```
> 注意：管理员用户只能用来管理用户，并没有数据读写的权限

4. 重启mongod：`mongod --auth`，如果配置了服务则需要修改配置文件：
```
security:
  authorization: enabled
```
配置文件的具体内容可以参考[官方文档](https://docs.mongodb.com/manual/reference/configuration-options/)

5. 以管理员身份登录
两种方式：
```
mongo --port 27017 -u "myUserAdmin" -p "abc123" --authenticationDatabase "admin"
```
```
mongo --port 27017

use admin
db.auth("myUserAdmin", "abc123" )
```



