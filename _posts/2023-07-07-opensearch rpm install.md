## opensearch介绍

opensearch是一种分布式，由社区驱动并取得 Apache 2.0 许可的 100% 开源搜索和分析套件，可用于一组广泛的使用案例，如实时应用程序监控、日志分析和网站搜索。OpenSearch 提供了一个高度可扩展的系统，通过集成的可视化工具 OpenSearch 控制面板为大量数据提供快速访问和响应，使用户可以轻松地探索他们的数据。OpenSearch 由 Apache Lucene 搜索库提供技术支持，它支持一系列搜索及分析功能，如 k-最近邻（KNN）搜索、SQL、异常检测、Machine Learning Commons、Trace Analytics、全文搜索等。

## opensearch与elasticsearch

由亚马逊创建的OpenSearch项目是基于旧版本的Elasticsearch和Kibana的分叉搜索项目。这些项目主要是为了支持 Amazon OpenSearch Service（以前称为 Amazon Elasticsearch Service）而创建的。

Elastic: 从版本 7.11（2021 年 2 月）开始，Elastic 将产品的许可证更改为 Elastic 许可证 v2 （ELv2） 和 SSPL。这是对亚马逊的非协作行为和滥用我们商标的回应。我们的产品仍然免费开放，但如果不与我们合作，亚马逊将无法再自由使用 Elasticsearch 和 Kibana 产品。亚马逊没有与我们合作并回馈，而是创建了自己的分叉项目。

amazon；开发人员接受开源软件有很多原因，其中最重要的一个原因是他们可以自由地在任何地方和以任何方式使用该软件。2021 年 1 月 21 日，Elastic NV 宣布，他们将改变软件许可策略，不在宽松的 Apache 2.0 版本（ALv2）下发布 Elasticsearch 和 Kibana 的新版本。相反，Elastic 发布 Elasticsearch 和 Kibana 时，在弹性许可证或服务器端公共许可证（SSPL）下提供源代码。这些许可证不是开源的，不会为用户提供同样的自由。由于部分开发人员希望他们的软件是开源的，而且他们想要避免被单个供应商锁定，因此我们决定从 Elasticsearch 和 Kibana 的最近 ALv2 版本创建并维护分叉。该分支被称为 OpenSearch 并在 ALv2 下提供。

## opensearch安装遇到的问题

**需要CA签发证书**

虽然open search有演示证书，但是生产环境并不适用。opensearch官方给了一个自签证书的教程，但是在启动时还是会出现问题，log显示不信任的证书。操，搞了半天的自签证书到最后不能用，那为什么还要放上这么一段无用操作的Guide?

因为没有CA签的证书，选择使用演示证书进行启动，使用tar包的方式太过于浪费时间，所以选择了使用RPM包进行安装，只要下载下来想要的rpm版本安装启动即可，然后进行更改配置重启就可以完成，大大节省了时间。

TIP: *如果是生产环境还是建议大家使用CA签发的证书，不要使用演示证书，为了安全着想*

**SSL和密码验证强行绑定**

在刚开始的时候因为没有证书导致无法启动，从网络上一顿搜索，最后发现可以关闭security插件或者移除插件，但是有个问题，现在没有了security插件连认证功能都没了。对，就是关闭了security连用户密码都没了。

笔者被折磨了两天时间，各种搜索引擎都上了，但是都没找到想要的答案，只好推翻了所有执行了rm -rf *。然后转向RPM安装，使用演示证书先启动了起来，后来有大佬帮忙更改了一个参数，这才达到了我想要的效果(使用http连接并且有验证功能)。

**opensearch dashboards密码问题**

我看了网上很多文章信誓旦旦的说，在yaml配置文件中直接更改用户密码就可以，我也认为dashabords跟kibana一样，密码是独立的。但是万万没想到，dashboards的密码是opensearch的密码，dashboards自己压根没有密码(笔者在关闭了opensearch security插件以后才发现)。希望相互转发的各位，你们不要再转发了，太坑人了。你不验证对错你有什么资格将他转载到你的博客里面。

## opensearch安装

问题已经说清楚了现在该安装了，这里选择使用RPM进行安装，三台服务器组成集群。

```bash
 wget https://artifacts.opensearch.org/releases/bundle/opensearch/2.8.0/opensearch-2.8.0-linux-x64.rpm
 rpm -ivh opensearch-2.8.0-linux-x64.rpm
systemctl daemon-reload
systemctl enable opensearch.service
systemctl start opensearch.service
```

这么看是不是简单多了，现在已经使用演示证书起了一个节点，其他节点也都是相同操作。确定opensearch可以使用演示证书启动后，接一下就要更改配置组成集群了，这部分具体的配置可以参考官方的文档，我做的更改仅适用自己的环境。

```yaml
# 集群名称，用于区分不同的OpenSearch集群
cluster.name: opensearch-test

# 节点名称，每个节点的名称应该是唯一的
node.name: node1

# 数据存储路径，OpenSearch将在这个路径下存储所有的索引和数据
path.data: /data/data

# 日志文件路径，OpenSearch将在这个路径下存储所有的日志文件
path.logs: /data/logs

# 网络主机地址，设置为0.0.0.0表示监听所有网络接口
network.host: 0.0.0.0

# HTTP服务端口，客户端将通过这个端口连接到OpenSearch
http.port: 9200

# 种子主机列表，这些主机将用于发现集群中的其他节点
discovery.seed_hosts: ["ip1", "ip2", "ip3"]

# 初始集群管理节点列表，这些节点将用于集群的初始启动和管理
cluster.initial_cluster_manager_nodes: ["node1", "node2", "node3"]

# 在启动恢复过程之前，集群需要的最小节点数
gateway.recover_after_nodes: 3

# 是否启用HTTP SSL，如果设置为false，那么HTTP连接将不会使用SSL加密
plugins.security.ssl.http.enabled: false
```

所有节点完成重启后，三个节点的open search集群就搭建好了。现在看起来似乎很容易，但是在坑里挣扎出不来的时候是真的很难受。