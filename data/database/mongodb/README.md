# MongoDB 的安装

当前脚本是为了在测试环境中，快速安装实例使用，请不要应用于生产环境；在测试环境中，如果需要保留数据，请谨慎使用脚本中的 `uninstall` 操作；[点此](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-red-hat/)查看官方安装手册。

### 操作过程

#### 准备工作

##### 克隆脚本

如果没有 `git`、`tree`、`zip` 等工具，请先安装。

```bash
[fsyyft@kvm-centos7-mongodb tmp]# /usr/bin/yum -y install git
[fsyyft@kvm-centos7-mongodb tmp]# /usr/bin/yum -y install tree
[fsyyft@kvm-centos7-mongodb tmp]# /usr/bin/yum -y install zip
```

```bash
[fsyyft@kvm-centos7-mongodb tmp]# cd /tmp/
[fsyyft@kvm-centos7-mongodb tmp]# /usr/bin/rm -rf ./deploy
[fsyyft@kvm-centos7-mongodb tmp]# /usr/bin/git clone https://github.com/fsyyft/deploy
```

##### 软件安装

```bash
[fsyyft@kvm-centos7-mongodb tmp]# cd ./deploy/data/shell/
[fsyyft@kvm-centos7-mongodb shell]# /usr/bin/chmod a+x ./deploy 
[fsyyft@kvm-centos7-mongodb shell]# ./deploy yum_install_mongodb
```

#### 实例安装

使用 `/data/database/mongodb/deploy` 脚本进行安装，直接运行脚本可以看到相关帮助信息。

```bash
[fsyyft@kvm-centos7-mongodb tmp]# cd /data/database/mongodb/
[fsyyft@kvm-centos7-mongodb mongodb]# ./deploy 
安装单实例：
    /data/database/mongodb/deploy install {端口号} single
    /data/database/mongodb/deploy install 2741 single
安装配置集群（不包含配置）：
    /data/database/mongodb/deploy install {端口号} config {集群名称}
    /data/database/mongodb/deploy install 2701 config configs
安装数据集群（不包含配置）：
    /data/database/mongodb/deploy install {端口号} shard {集群名称}
    /data/database/mongodb/deploy install 2711 shard shard1
安装路由（不包含配置）：
    /data/database/mongodb/deploy install {端口号} mongos {配置集群信息}
    /data/database/mongodb/deploy install 2700 mongos configs/127.0.0.1:2701,127.0.0.1:2702,127.0.0.1:2703
卸载实例（包含删除数据）：
    /data/database/mongodb/deploy uninstall {端口号}
    /data/database/mongodb/deploy uninstall 2742
```

##### 安装一个单实例

使用脚本进行安装。

```bash
[fsyyft@kvm-centos7-mongodb mongodb]# ./deploy install 2741 single
```

进行简单的测试。

```bash
[fsyyft@kvm-centos7-mongodb mongodb]# /usr/bin/mongo --port 2741
> use test;
switched to db test
> db.test.insert({"test": "hello"});
WriteResult({ "nInserted" : 1 })
> db.test.find();
{ "_id" : ObjectId("5bff6302b63b8c3737230de9"), "test" : "hello" }
> exit
bye
```

实例的目录结构如下：

```bash
[fsyyft@kvm-centos7-mongodb mongodb]# /usr/bin/tree -ug -L 3 ./2741
./2741
├── [mongod   mongod  ]  conf
│   └── [mongod   mongod  ]  mongodb.2741.conf
├── [mongod   mongod  ]  data
│   ├── [mongod   mongod  ]  admin
│   │   ├── [mongod   mongod  ]  collection
│   │   └── [mongod   mongod  ]  index
│   ├── [mongod   mongod  ]  config
│   │   ├── [mongod   mongod  ]  collection
│   │   └── [mongod   mongod  ]  index
│   ├── [mongod   mongod  ]  diagnostic.data
│   │   └── [mongod   mongod  ]  metrics.2018-11-29T03-52-25Z-00000
│   ├── [mongod   mongod  ]  journal
│   │   ├── [mongod   mongod  ]  WiredTigerLog.0000000001
│   │   ├── [mongod   mongod  ]  WiredTigerPreplog.0000000001
│   │   └── [mongod   mongod  ]  WiredTigerPreplog.0000000002
│   ├── [mongod   mongod  ]  local
│   │   ├── [mongod   mongod  ]  collection
│   │   └── [mongod   mongod  ]  index
│   ├── [mongod   mongod  ]  _mdb_catalog.wt
│   ├── [mongod   mongod  ]  mongod.lock
│   ├── [mongod   mongod  ]  sizeStorer.wt
│   ├── [mongod   mongod  ]  storage.bson
│   ├── [mongod   mongod  ]  test
│   │   ├── [mongod   mongod  ]  collection
│   │   └── [mongod   mongod  ]  index
│   ├── [mongod   mongod  ]  WiredTiger
│   ├── [mongod   mongod  ]  WiredTigerLAS.wt
│   ├── [mongod   mongod  ]  WiredTiger.lock
│   ├── [mongod   mongod  ]  WiredTiger.turtle
│   └── [mongod   mongod  ]  WiredTiger.wt
├── [mongod   mongod  ]  logs
│   └── [mongod   mongod  ]  mongodb.2741.log
└── [mongod   mongod  ]  tmp
    ├── [mongod   mongod  ]  mongodb.2741.pid
    └── [mongod   mongod  ]  mongodb-2741.sock
```

进行实例删除。

```bash
[fsyyft@kvm-centos7-mongodb mongodb]# ./deploy uninstall 2741
```

##### 配置一个集群

###### 安装分片

使用脚本进行安装。

```bash
[fsyyft@kvm-centos7-mongodb mongodb]# /data/database/mongodb/deploy install 2711 shard shard1
[fsyyft@kvm-centos7-mongodb mongodb]# /data/database/mongodb/deploy install 2712 shard shard1
[fsyyft@kvm-centos7-mongodb mongodb]# /data/database/mongodb/deploy install 2713 shard shard1
[fsyyft@kvm-centos7-mongodb mongodb]# /data/database/mongodb/deploy install 2721 shard shard2
[fsyyft@kvm-centos7-mongodb mongodb]# /data/database/mongodb/deploy install 2722 shard shard2
[fsyyft@kvm-centos7-mongodb mongodb]# /data/database/mongodb/deploy install 2723 shard shard2
[fsyyft@kvm-centos7-mongodb mongodb]# /data/database/mongodb/deploy install 2731 shard shard3
[fsyyft@kvm-centos7-mongodb mongodb]# /data/database/mongodb/deploy install 2732 shard shard3
[fsyyft@kvm-centos7-mongodb mongodb]# /data/database/mongodb/deploy install 2733 shard shard3
```

配置分片集群。

```bash
[fsyyft@kvm-centos7-mongodb mongodb]# /usr/bin/mongo --port 2711
> config = { _id: "shard1", members: [ { _id: 11, host: "127.0.0.1:2711" }, { _id: 12, host: "127.0.0.1:2712" }, { _id: 13, host: "127.0.0.1:2713" } ] };
> rs.initiate(config);
{ "ok" : 1 }
shard1:PRIMARY> db.createUser({ user: "mongo", pwd: "mongo", roles: [ "root" ] });
Successfully added user: { "user" : "mongo", "roles" : [ "root" ] }
shard1:PRIMARY> db.auth("mongo", "mongo");
1
```

```bash
[fsyyft@kvm-centos7-mongodb mongodb]# /usr/bin/mongo --port 2721
> config = { _id: "shard2", members: [ { _id: 21, host: "127.0.0.1:2721" }, { _id: 22, host: "127.0.0.1:2722" }, { _id: 23, host: "127.0.0.1:2723" } ] };
> rs.initiate(config);
{ "ok" : 1 }
shard2:PRIMARY> db.createUser({ user: "mongo", pwd: "mongo", roles: [ "root" ] });
Successfully added user: { "user" : "mongo", "roles" : [ "root" ] }
shard2:PRIMARY> db.auth("mongo", "mongo");
1
```

```bash
[fsyyft@kvm-centos7-mongodb mongodb]# /usr/bin/mongo --port 2731
> config = { _id: "shard3", members: [ { _id: 31, host: "127.0.0.1:2731" }, { _id: 32, host: "127.0.0.1:2732" }, { _id: 33, host: "127.0.0.1:2733" } ] };
> rs.initiate(config);
{ "ok" : 1 }
shard3:PRIMARY> db.createUser({ user: "mongo", pwd: "mongo", roles: [ "root" ] });
Successfully added user: { "user" : "mongo", "roles" : [ "root" ] }
shard3:PRIMARY> db.auth("mongo", "mongo");
1
```

配置完成后可通过 `rs.status();` 指令查看状态。

###### 安装配置

使用脚本进行安装。

```bash
[fsyyft@kvm-centos7-mongodb tmp]# cd /data/database/mongodb/
[fsyyft@kvm-centos7-mongodb mongodb]# ./deploy install 2701 config configs
[fsyyft@kvm-centos7-mongodb mongodb]# ./deploy install 2702 config configs
[fsyyft@kvm-centos7-mongodb mongodb]# ./deploy install 2703 config configs
```

配置集群。

```bash
[fsyyft@kvm-centos7-mongodb mongodb]# /usr/bin/mongo --port 2701
> config = { _id: "configs", members: [ { _id: 1, host: "127.0.0.1:2701" }, { _id: 2, host: "127.0.0.1:2702" }, { _id: 3, host: "127.0.0.1:2703" } ] };
> rs.initiate(config);
```

配置完成后可通过 `rs.status();` 指令查看状态。

###### 安装路由

使用脚本进行安装。

```bash
[fsyyft@kvm-centos7-mongodb mongodb]# /data/database/mongodb/deploy install 2700 mongos configs/127.0.0.1:2701,127.0.0.1:2702,127.0.0.1:2703
```

添加用户。

```bash
[fsyyft@kvm-centos7-mongodb mongodb]# /usr/bin/mongo --port 2700
mongos> use admin;
switched to db admin
mongos> db.createUser({ user: "mongo", pwd: "mongo", roles: [ "root" ] });
Successfully added user: { "user" : "mongo", "roles" : [ "root" ] }
mongos> db.auth("mongo", "mongo");
1
mongos> sh.addShard("shard1/127.0.0.1:2711,127.0.0.1:2712,127.0.0.1:2713");
mongos> sh.addShard("shard2/127.0.0.1:2721,127.0.0.1:2722,127.0.0.1:2723");
mongos> sh.addShard("shard3/127.0.0.1:2731,127.0.0.1:2732,127.0.0.1:2733");
```

配置完成后可通过 `sh.status();` 指令查看状态。

###### 测试

创建一个 `hello` 库，并开启分片；设置 `hello.world` 这个集合使用 `id` 分片。

为了容易看效果，把 `chunksize` 设置为 `1`。

```bash
[fsyyft@kvm-centos7-mongodb mongodb]# /usr/bin/mongo --port 2700
mongos> use admin;
switched to db admin
mongos> db.auth("mongo", "mongo");
1
mongos> use config;
switched to db config
mongos> db.settings.save({ _id:"chunksize", value: 1 });
WriteResult({ "nMatched" : 0, "nUpserted" : 1, "nModified" : 0, "_id" : "chunksize" })
mongos> use admin;
switched to db admin
mongos> db.runCommand({ enablesharding :"hello"});
mongos> db.runCommand({ shardcollection : "hello.world",key : {id: 1} });
```

写入测试数据。

```bash
[fsyyft@kvm-centos7-mongodb mongodb]# /usr/bin/mongo --port 2700
mongos> use admin;
switched to db admin
mongos> db.auth("mongo", "mongo");
1
mongos> use hello;
switched to db hello
mongos> for (var idx = 1; idx <= 65535; idx++) db.world.save({"id": idx, "hello": "world"});
WriteResult({ "nInserted" : 1 })
```

写入数据完成后，可以在 `hello` 库中通过 `db.world.stats();` 查看分片情况。

###### 清除测试环境

```bash
[fsyyft@kvm-centos7-mongodb tmp]# cd /data/database/mongodb/
[fsyyft@kvm-centos7-mongodb mongodb]# ./deploy uninstall 2700
[fsyyft@kvm-centos7-mongodb mongodb]# ./deploy uninstall 2701
[fsyyft@kvm-centos7-mongodb mongodb]# ./deploy uninstall 2702
[fsyyft@kvm-centos7-mongodb mongodb]# ./deploy uninstall 2703
[fsyyft@kvm-centos7-mongodb mongodb]# ./deploy uninstall 2711
[fsyyft@kvm-centos7-mongodb mongodb]# ./deploy uninstall 2722
[fsyyft@kvm-centos7-mongodb mongodb]# ./deploy uninstall 2733
[fsyyft@kvm-centos7-mongodb mongodb]# ./deploy uninstall 2721
[fsyyft@kvm-centos7-mongodb mongodb]# ./deploy uninstall 2722
[fsyyft@kvm-centos7-mongodb mongodb]# ./deploy uninstall 2723
[fsyyft@kvm-centos7-mongodb mongodb]# ./deploy uninstall 2731
[fsyyft@kvm-centos7-mongodb mongodb]# ./deploy uninstall 2732
[fsyyft@kvm-centos7-mongodb mongodb]# ./deploy uninstall 2733
```