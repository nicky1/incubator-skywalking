## 所需的第三方软件
- 被监控程序要求JDK6+
- SkyWalking collector和WebUI要求JDK8+
- Elasticsearch 5.x
- Zookeeper 3.4.10

## 下载发布版本
- 前向[发布页面](http://skywalking.apache.org/downloads/)

## 部署Elasticsearch
- 修改`elasticsearch.yml`文件
  - 设置 `cluster.name: CollectorDBCluster`。此名称需要和collector配置文件一致。
  - 设置 `node.name: anyname`, 可以设置为任意名字，如Elasticsearch为集群模式，则每个节点名称需要不同。
  - 增加如下配置

```
# ES监听的ip地址
network.host: 0.0.0.0
thread_pool.bulk.queue_size: 1000
```

- 启动Elasticsearch

### 部署collector
1. 解压安装包`tar -xvf skywalking-collector.tar.gz`，windows用户可以选择zip包
2. 设置Collector集群模式

集群模式主要依赖Zookeeper的注册和应用发现能力。所以，你只需要调整 `config/application.yml`中的host和port配置，使用实际IP和端口，代替默认配置。
其次，将storage的注释取消，并修改为Elasticsearch集群的节点地址信息。


- `config/application.yml`
```
cluster:
# 配置zookeeper集群信息
  zookeeper:
    hostPort: localhost:2181
    sessionTimeout: 100000
naming:
# 配置探针使用的host和port
jetty:
    host: localhost
    port: 10800
    contextPath: /
remote:
  gRPC:
    host: localhost
    port: 11800
agent_gRPC:
  gRPC:
    host: localhost
    port: 11800
agent_jetty:
  jetty:
    host: localhost
    port: 12800
    contextPath: /
analysis_register:
  default:
analysis_jvm:
  default:
analysis_segment_parser:
  default:
    bufferFilePath: ../buffer/
    bufferOffsetMaxFileSize: 10M
    bufferSegmentMaxFileSize: 500M
ui:
  jetty:
    host: localhost
    port: 12800
    contextPath: /
# 配置 Elasticsearch 集群连接信息
storage:
  elasticsearch:
    clusterName: CollectorDBCluster
    clusterTransportSniffer: true
    clusterNodes: localhost:9300
    indexShardsNumber: 2
    indexReplicasNumber: 0
    highPerformanceMode: true
    # 设置统计指标数据的失效时间，当指标数据失效时系统将数据自动删除.
    traceDataTTL: 90 # 单位为分
    minuteMetricDataTTL: 90 # 单位为分
    hourMetricDataTTL: 36 # 单位为小时
    dayMetricDataTTL: 45 # 单位为天
    monthMetricDataTTL: 18 # 单位为月
configuration:
  default:
#     namespace: xxxxx
# 告警阀值
    applicationApdexThreshold: 2000
    serviceErrorRateThreshold: 10.00
    serviceAverageResponseTimeThreshold: 2000
    instanceErrorRateThreshold: 10.00
    instanceAverageResponseTimeThreshold: 2000
    applicationErrorRateThreshold: 10.00
    applicationAverageResponseTimeThreshold: 2000
# 热力图配置，修改配置后需要删除热力指标统计表，由系统重建
    thermodynamicResponseTimeStep: 50
    thermodynamicCountOfResponseTimeSteps: 40
```


3. 运行`bin/startup.sh`启动。windows用户为.bat文件。


### 部署UI

1. 解压安装包 `tar -xvf skywalking-dist.tar.gz`，windows用户可以选择zip包
2. 配置UI集群模式.

UI的配置信息保存在 `webapp/webapp.yml` 中.

| 配置项                            | 描述                                                                             |
|----------------------------------|----------------------------------------------------------------------------------|
| `server.port`                    | 监听端口                                                                          |
| `collector.ribbon.listOfServers` | collector命名服务地址.(与 `config/application.yml` 中的`naming.jetty`配置保持相同 )，多个Collector地址以`,`分割 |
| `collector.path`                 | Collector查询uri. 默认: /graphql                                                                          |
| `collector.ribbon.ReadTimeout`   | 查询超时时间. 默认: 10 秒                                                                                   |
| `security.user.*`                | 登录用户名/密码. 默认: admin/admin                                                                          |

3. 运行 `bin/webappService.sh`
