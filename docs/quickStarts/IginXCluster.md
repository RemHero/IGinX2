# IginX 安装使用教程（集群）

[TOC]

IginX 是清华大学大数据系统软件国家工程实验室，为满足工业互联网场景推出的新一代高可扩展时序数据库分布式中间件，目前支持 IoTDB，InfluxDB 作为数据后端。

## 安装

### Java 安装

由于 ZooKeeper、IginX 以及 IoTDB 都是使用 Java 开发的，因此首先需要安装 Java。如果本地已经安装了 JDK>=1.8 的运行环境，**直接跳过此步骤**。

1. 首先访问 [Java官方网站](https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html)下载适用于当前系统的 JDK 包。
2. 安装

```shell
$ cd ~/Downloads
$ tar -zxf jdk-8u181-linux-x64.gz # 解压文件
$ mkdir /opt/jdk
$ mv jdk-1.8.0_181 /opt/jdk/
```

3. 设置路径

编辑 ~/.bashrc 文件，在文件末端加入如下的两行：

```shell
export JAVA_HOME = /usr/jdk/jdk-1.8.0_181
export PATH=$PATH:$JAVA_HOME/bin
```

加载更改后的配置文件：

```shell
$ source ~/.bashrc
```

4. 使用 java -version 判断 JDK 是否安装成功

```shell
$ java -version
java version "1.8.0_181"
Java(TM) SE Runtime Environment (build 1.8.0_181-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.181-b13, mixed mode)
```

如果显示出如上的字样，则表示安装成功。

### ZooKeeper 安装

ZooKeeper 是 Apache 推出的开源的分布式应用程序协调服务。如果您需要部署大于一个 IginX 实例，则需要安装 ZooKeeper

ZooKeeper 是 Apache 推出的开源的分布式应用程序协调服务。具体安装方式如下：

1. 访问[官网](https://zookeeper.apache.org/releases.html)下载并解压 ZooKeeper

```shell
$ cd ~
$ wget https://mirrors.bfsu.edu.cn/apache/zookeeper/zookeeper-3.7.0/apache-zookeeper-3.7.0-bin.tar.gz
$ tar -zxvf apache-zookeeper-3.7.0-bin.tar.gz
```

2. 修改 ZooKeeper 默认配置文件

```shell
$ cd apache-zookeeper-3.7.0-bin/
$ mkdir data
$ cp conf/zoo_sample.cfg conf/zoo.cfg
```

然后编辑 conf/zoo.cfg 文件，将

```shell
dataDir=/tmp/zookeeper
```

修改为

```shell
dataDir=data
```

### IoTDB 安装

IoTDB 是 Apache 推出的时序数据库，具体安装方式如下：

```shell
$ cd ~
$ wget https://mirrors.bfsu.edu.cn/apache/iotdb/0.12.0/apache-iotdb-0.12.0-server-bin.zip
$ unzip apache-iotdb-0.12.0-server-bin.zip
```

### IginX 安装

直接访问 [IginX 项目](https://github.com/thulab/IginX/)下载 [IginX 项目发布包](https://github.com/thulab/IginX/releases/download/rc%2Fv0.2.0/IginX-release-v0.2.0-bin.tar.gz) 即可。

```shell
$ cd ~
$ wget https://github.com/thulab/IginX/releases/download/release%2Fv0.2.0/IginX-release-v0.2.0-bin.zip
$ unzip IginX-release-v0.2.0-bin.zip
```

## 启动

这里以启动一个两个 IginX 实例，两个 IoTDB 实例为例子，演示如何启动 IginX 集群

### 启动多个 IoTDB 实例

这里以单机启动两个端口分别为 6667 和 7667 的实例为例

修改配置文件 IoTDB_HOME/conf/iotdb-engine.properties

```shell
rpc_port=6667
```

启动第一个实例

```shell
$ cd ~
$ cd apache-iotdb-0.12.0-server-bin/
$ ./sbin/start-server.sh # 启动实例一 127.0.0.1: 6667
```

修改配置文件 conf/iotdb-engine.properties

```shell
rpc_port=7667
```

启动第二个实例

```shell
$ ./sbin/start-server.sh # 启动实例二 127.0.0.1: 7667
```

### 启动 ZooKeeper

```shell
$ cd ~
$ cd apache-zookeeper-3.7.0-bin/
$ ./bin/zkServer.sh start
```

显示出如下字样，表示 ZooKeeper 启动成功

```shell
ZooKeeper JMX enabled by default
Using config: /home/root/apache-zookeeper-3.7.0-bin/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
```

### 启动多个 IginX 实例

修改 IginX_HOME/conf/config. Properties，加入启动的两台IoTDB实例

```shell
storageEngineList=127.0.0.1#6667#iotdb#username=root#password=root#sessionPoolSize=100#dataDir=/path/to/your/data/,127.0.0.1#6688#iotdb#username=root#password=root#sessionPoolSize=100#dataDir=/path/to/your/data/

#存储方式选择 ZooKeeper
metaStorage=zookeeper 

# 提供ZooKeeper端口
zookeeperConnectionString=127.0.0.1:2181

#注释掉file、etcd相关配置
#fileDataDir=meta
#etcdEndpoints=http://localhost:2379
```

启动第一个 IginX 实例

```shell
$ cd ~
$ cd Iginx
$ chmod +x sbin/start_iginx.sh # 为启动脚本添加启动权限
$ ./sbin/start_iginx.sh
```

修改 conf/config. Properties

```shell
# iginx 绑定的端口
port=7888
# rest 绑定的端口
restPort=7666
```

启动第二个 IginX 实例

```shell
$ ./sbin/start_iginx.sh
```

## 访问 IginX

### RESTful 接口

启动完成后，可以便捷地使用 RESTful 接口向 IginX 中写入并查询数据。

创建文件 insert.json，并向其中添加如下的内容：

```json
[
  {
    "name": "archive_file_tracked",
    "datapoints": [
        [1359788400000, 123.3],
        [1359788300000, 13.2 ],
        [1359788410000, 23.1 ]
    ],
    "tags": {
        "host": "server1",
        "data_center": "DC1"
    }
  },
  {
      "name": "archive_file_search",
      "timestamp": 1359786400000,
      "value": 321,
      "tags": {
          "host": "server2"
      }
  }
]
```

使用如下的命令即可从 IginX 实例一向数据库中插入数据：

```shell
$ curl -XPOST -H'Content-Type: application/json' -d @insert.json http://127.0.0.1:6666/api/v1/datapoints
```

在插入数据后，还可以使用 RESTful 接口查询刚刚写入的数据。

创建文件 query.json，并向其中写入如下的数据：

```json
{
	"start_absolute" : 1,
	"end_relative": {
		"value": "5",
		"unit": "days"
	},
	"time_zone": "Asia/Kabul",
	"metrics": [
		{
		"name": "archive_file_tracked"
		},
		{
		"name": "archive_file_search"
		}
	]
}
```

使用如下的命令从 IginX 实例二查询数据：

```shell
$ curl -XPOST -H'Content-Type: application/json' -d @query.json http://127.0.0.1:6666/api/v1/datapoints/query
```

命令会返回刚刚插入的数据点信息：

```json
{
    "queries": [
        {
            "sample_size": 3,
            "results": [
                {
                    "name": "archive_file_tracked",
                    "group_by": [
                        {
                            "name": "type",
                            "type": "number"
                        }
                    ],
                    "tags": {
                        "data_center": [
                            "DC1"
                        ],
                        "host": [
                            "server1"
                        ]
                    },
                    "values": [
                        [
                            1359788300000,
                            13.2
                        ],
                        [
                            1359788400000,
                            123.3
                        ],
                        [
                            1359788410000,
                            23.1
                        ]
                    ]
                }
            ]
        },
        {
            "sample_size": 1,
            "results": [
                {
                    "name": "archive_file_search",
                    "group_by": [
                        {
                            "name": "type",
                            "type": "number"
                        }
                    ],
                    "tags": {
                        "host": [
                            "server2"
                        ]
                    },
                    "values": [
                        [
                            1359786400000,
                            321.0
                        ]
                    ]
                }
            ]
        }
    ]
}
```

更多接口可以参考 [IginX 官方手册](https://github.com/thulab/IginX/blob/main/docs/pdf/userManualC.pdf) 。

### RPC 接口

除了 RESTful 接口外，IginX 还提供了 RPC 的数据访问接口，具体接口参考 [IginX 官方手册](https://github.com/thulab/IginX/blob/main/docs/pdf/userManualC.pdf)，同时 IginX 还提供了部分[官方 example](https://github.com/thulab/IginX/tree/main/example/src/main/java/cn/edu/tsinghua/iginx/session)，展示了 RPC 接口最常见的用法。

下面是一个简短的使用教程。

由于目前 IginX 0.2 版本还未发布到 maven 中央仓库，因此如需使用的话，需要手动安装到本地的 maven 仓库。具体安装方式如下：

```shell
# 下载 iginx 0.2 rc 版本源码包
$ wget https://github.com/thulab/IginX/archive/refs/tags/rc/v0.2.0.tar.gz 
# 解压源码包
$ tar -zxvf v0.2.0.tar.gz
# 进入项目主目录
$ cd IginX-rc-v0.2.0
# 安装到本地 maven 仓库
$ mvn clean install -DskipTests
```

具体在使用时，只需要在相应的项目的 pom 文件中引入如下的依赖：

```xml
<dependency>
  	<groupId>cn.edu.tsinghua</groupId>
  	<artifactId>iginx-core</artifactId>
  	<version>0.1.0-SNAPSHOT</version>
</dependency>
```

在访问 iginx 之前，首先需要创建 session，并尝试连接。Session 构造器有 4 个参数，分别是要连接的 IginX 的 ip，port，以及用于 IginX 认证的用户名和密码。目前的权鉴系统还在编写中，因此访问后端 IginX 的账户名和密码直接填写 root 即可：

```Java
Session session = new Session("127.0.0.1", 6888, "root", "root");
session.openSession();
```

随后可以尝试向 IginX 中插入数据。由于 IginX 支持在数据首次写入时创建时间序列，因此并不需要提前调用相关的序列创建接口。IginX 提供了行式和列式的数据写入接口，以下是列式数据写入接口的使用样例：

```java
private static void insertColumnRecords(Session session) throws SessionException, ExecutionException {
        List<String> paths = new ArrayList<>();
        paths.add("sg.d1.s1");
        paths.add("sg.d2.s2");
        paths.add("sg.d3.s3");
        paths.add("sg.d4.s4");

        int size = 1500;
        long[] timestamps = new long[size];
        for (long i = 0; i < size; i++) {
            timestamps[(int) i] = i;
        }

        Object[] valuesList = new Object[4];
        for (long i = 0; i < 4; i++) {
            Object[] values = new Object[size];
            for (long j = 0; j < size; j++) {
                if (i < 2) {
                  values[(int) j] = i + j;
                } else {
                  values[(int) j] = RandomStringUtils.randomAlphanumeric(10).getBytes();
                }
            }
            valuesList[(int) i] = values;
        }

        List<DataType> dataTypeList = new ArrayList<>();
        for (int i = 0; i < 2; i++) {
            dataTypeList.add(DataType.LONG);
        }
        for (int i = 0; i < 2; i++) {
            dataTypeList.add(DataType.BINARY);
        }

        session.insertColumnRecords(paths, timestamps, valuesList, dataTypeList, null);
}
```

在完成数据写入后，可以使用数据查询接口查询刚刚写入的数据：

```java
private static void queryData(Session session) throws SessionException, ExecutionException {
        List<String> paths = new ArrayList<>();
        paths.add("sg.d1.s1");
        paths.add("sg.d2.s2");
        paths.add("sg.d3.s3");
        paths.add("sg.d4.s4");

        long startTime = 100L;
        long endTime = 200L;

        SessionQueryDataSet dataSet = session.queryData(paths, startTime, endTime);
        dataSet.print();
}
```

还可以使用降采样聚合查询接口来查询数据的区间统计值：

```java
private static void downsampleQuery(Session session) throws SessionException, ExecutionException {
        List<String> paths = new ArrayList<>();
        paths.add("sg.d1.s1");
        paths.add("sg.d2.s2");

        long startTime = 100L;
        long endTime = 1101L;

        // MAX
        SessionQueryDataSet dataSet = session.downsampleQuery(paths, startTime, endTime, AggregateType.MAX, 100);
        dataSet.print();

        // MIN
        dataSet = session.downsampleQuery(paths, startTime, endTime, AggregateType.MIN, ROW_INTERVAL * 100);
        dataSet.print();

        // FIRST
        dataSet = session.downsampleQuery(paths, startTime, endTime, AggregateType.FIRST, ROW_INTERVAL * 100);
        dataSet.print();

        // LAST
        dataSet = session.downsampleQuery(paths, startTime, endTime, AggregateType.LAST, ROW_INTERVAL * 100);
        dataSet.print();

        // COUNT
        dataSet = session.downsampleQuery(paths, startTime, endTime, AggregateType.COUNT, ROW_INTERVAL * 100);
        dataSet.print();

        // SUM
        dataSet = session.downsampleQuery(paths, startTime, endTime, AggregateType.SUM, ROW_INTERVAL * 100);
        dataSet.print();

        // AVG
        dataSet = session.downsampleQuery(paths, startTime, endTime, AggregateType.AVG, ROW_INTERVAL * 100);
        dataSet.print();

}

```

最终使用完 session 后需要手动关闭，释放连接：

```shell
session.closeSession();
```

完整版使用代码可以参考：https://github.com/thulab/IginX/blob/main/example/src/main/java/cn/edu/tsinghua/iginx/session/IoTDBSessionExample.java