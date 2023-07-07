---
layout: single
title:  "Prometheus新手指南"
---

# Prometheus新手用户指南

## 1. Prometheus简介

Prometheus是一款开源的监控和告警系统，最初由SoundCloud开发，现在是Cloud Native Computing Foundation的一部分。Prometheus的设计目标是实现丰富的数据模型，灵活的查询语言，以及高效的存储和处理时间序列数据。

### 1.1 Prometheus的主要特性

- **多维数据模型**：Prometheus的数据模型是基于时间序列的，时间序列由指标名称和键值对（标签）标识。这种多维数据模型使得Prometheus可以非常灵活地查询和聚合数据。

- **强大的查询语言**：Prometheus自带的查询语言PromQL可以对收集的数据进行复杂的查询和计算，包括聚合、预测、滚动窗口等操作。

- **独立的服务器节点**：Prometheus服务器是独立的节点，它们不依赖网络存储，这使得Prometheus可以在网络分区或者存储故障的情况下继续工作。

- **主动拉取数据**：Prometheus主动从被监控的服务中拉取数据，这使得Prometheus可以更好地控制数据的收集频率和超时，同时也简化了被监控服务的配置。

- **支持推送数据**：虽然Prometheus默认是拉取数据，但也支持通过Pushgateway推送短期的批处理任务的数据。

- **多种数据可视化选项**：Prometheus自带一个简单的Web界面，同时也支持Grafana等更强大的数据可视化工具。

### 1.2 Prometheus的架构和组件

Prometheus的架构主要包括以下几个组件：

![Prometheus Architecture](https://prometheus.io/assets/architecture.png)

- **Prometheus Server**：负责收集和存储时间序列数据。

- **Client Libraries**：提供给被监控的服务，用于暴露指标数据。

- **Pushgateway**：用于接收不能被Prometheus Server直接拉取数据的服务推送的数据。

- **Exporters**：用于暴露常见服务的指标数据，例如Linux系统指标、MySQL指标等。

- **Alertmanager**：处理Prometheus Server发送的告警。

- **其他工具**：包括支持服务发现的工具、支持远程存储的适配器等。

## 2. Prometheus的基本概念

理解Prometheus的基本概念是使用Prometheus的关键。这些概念包括时间序列数据、指标和标签、数据抓取和存储。

### 2.1 时间序列数据

Prometheus的所有数据都是以时间序列的形式存储的。时间序列是由指标名称和一组标签（键值对）标识的数据点的集合。每个数据点包含一个时间戳和一个浮点值。

例如，假设我们有一个监控HTTP请求的指标，它的名称是`http_requests_total`，我们可以为不同的请求方法和处理器添加标签，如下所示：

```plaintext
http_requests_total{method="GET", handler="/api/tracks"} 17232
http_requests_total{method="POST", handler="/api/tracks"} 1023
http_requests_total{method="GET", handler="/api/albums"} 2313
http_requests_total{method="POST", handler="/api/albums"} 123
```

这四个时间序列分别表示GET和POST方法在/api/tracks和/api/albums处理器上的HTTP请求总数。

### 2.2 指标和标签

指标是Prometheus用来度量的对象，例如HTTP请求的数量、CPU使用率、磁盘空间等。指标名称通常是一个描述度量内容的字符串，可以包含字母、数字、下划线和冒号。

标签是键值对，用于区分不同的时间序列。在上面的例子中，我们使用method和handler两个标签来区分不同的请求方法和处理器。

### 2.3 数据抓取和存储

Prometheus通过HTTP从被监控的服务中抓取数据。被监控的服务需要提供一个HTTP接口，返回当前的所有指标值。这个接口通常是/metrics。

Prometheus抓取数据的频率可以配置，通常是每15秒或者每分钟一次。抓取的数据会存储在本地的磁盘上，并且可以通过PromQL查询。

Prometheus的存储是基于时间序列的，数据按照时间顺序存储。Prometheus会定期压缩和清理旧的数据，你可以配置数据保留的时间。

## 3. 配置Prometheus

Prometheus的配置是通过YAML格式的配置文件进行的。配置文件主要包括两部分：`global`和`scrape_configs`。

### 3.1 全局配置参数

全局配置参数包括`scrape_interval`和`evaluation_interval`，分别设置Prometheus抓取指标和评估告警规则的频率。

### 3.2 数据抓取配置

数据抓取配置包括`job_name`、`scrape_interval`、`metrics_path`和`static_configs`。`job_name`设置任务的名称，用于标识抓取的数据来源。`scrape_interval`设置这个任务抓取指标的频率，如果没有设置，会使用全局的`scrape_interval`。`metrics_path`设置抓取指标的HTTP接口的路径。`static_configs`设置静态的目标，每个目标是一个包含地址（`targets`）和标签（`labels`）的配置。

### 3.3 配置示例和解析

下面是一个Prometheus配置文件的示例：

```yaml
global:
  scrape_interval:     15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  - job_name: 'node_exporter'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9100']
```

这个配置文件定义了两个抓取任务：一个是抓取Prometheus自身的指标，一个是抓取Node Exporter的指标。Node Exporter是一个常用的系统指标导出器，它可以导出CPU、内存、磁盘等系统指标。

对于Prometheus的任务，使用了全局的抓取间隔（15秒）。对于Node Exporter的任务，设置了一个特定的抓取间隔（5秒）。

### 3.4 使用file_sd配置动态目标

除了静态配置目标，Prometheus还支持动态配置目标。其中一种常用的方式是使用`file_sd`配置。

`file_sd`配置允许Prometheus从指定的文件中读取目标。这些文件必须是JSON或者YAML格式的，包含了一组目标的列表。每个目标是一个包含地址（`targets`）和标签（`labels`）的配置。

当这些文件发生变化时，Prometheus会自动重新加载配置。这使得你可以动态地添加、修改或者删除目标，而不需要重启Prometheus。

下面是一个`file_sd`配置的示例：

```yaml
scrape_configs:
  - job_name: 'my_service'
    file_sd_configs:
      - files:
        - 'targets.json'
```

这个配置告诉Prometheus从targets.json文件中读取my_service的目标。targets.json文件的内容可能如下：

```json
[
  {
    "targets": ["localhost:8080"],
    "labels": {
      "env": "dev",
      "job": "my_service"
    }
  }
]
```

这个文件定义了一个目标，地址是localhost:8080，标签是env=dev和job=my_service。

注意，file_sd配置的路径可以包含通配符，例如'targets/*.json'。这样，Prometheus会加载所有匹配的文件。

### 3.5 使用kubernetes_sd_config配置Kubernetes目标

如果你在Kubernetes环境中运行Prometheus，你可以使用`kubernetes_sd_config`配置来自动发现Kubernetes的服务、端点、Pod和节点。

`kubernetes_sd_config`允许Prometheus从Kubernetes API服务器获取目标。Prometheus会自动发现新的Pod、服务等，并根据它们的状态和注解更新目标。

下面是一个`kubernetes_sd_config`配置的示例：

```yaml
scrape_configs:
  - job_name: 'kubernetes'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        target_label: __address__
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
```
这个配置告诉Prometheus从Kubernetes API服务器获取Pod的信息，并根据Pod的注解来配置目标。relabel_configs部分定义了如何从原始的标签生成最终的标签。

注意，kubernetes_sd_configs配置需要Prometheus可以访问Kubernetes API服务器。你可能需要配置Kubernetes的服务账号和角色绑定，以给Prometheus提供必要的权限。

### 3.6 使用relabel_configs配置标签重写

在Prometheus抓取数据之前，可以使用`relabel_configs`配置来重写或者过滤标签。这是一个强大的功能，可以用来动态生成标签、改变标签名称、过滤不需要的目标等。

`relabel_configs`配置是一个列表，每个配置包括`source_labels`、`separator`、`target_label`、`regex`、`modulus`、`replacement`和`action`等参数。

- `source_labels`：定义了源标签，这些标签可以从`__meta_*`标签、标签名称或者其他值中选择。
- `separator`：定义了源标签之间的分隔符，默认是`;`。
- `target_label`：定义了目标标签，如果这个标签已经存在，它的值会被替换。
- `regex`：定义了一个正则表达式，用于匹配源标签的值。
- `modulus`：定义了一个数字，用于取模运算。
- `replacement`：定义了一个替换字符串，用于生成目标标签的值。你可以使用`$1`、`$2`等来引用正则表达式的分组。
- `action`：定义了操作类型，包括`replace`（替换）、`keep`（保留）、`drop`（删除）和`hashmod`（哈希取模）。

下面是一个`relabel_configs`配置的示例：

```yaml
relabel_configs:
  - source_labels: [__address__]
    target_label: __param_target
  - source_labels: [__param_target]
    target_label: instance
  - target_label: __address__
    replacement: blackbox_exporter:9115
```

这个配置做了三个操作：

把__address__标签的值复制到__param_target标签。
把__param_target标签的值复制到instance标签。
把__address__标签的值替换为blackbox_exporter:9115。
这个配置通常用于Blackbox Exporter的配置，Blackbox Exporter是一个常用的黑盒监控工具，可以监控HTTP、TCP、ICMP等协议。

## 4. Prometheus查询语言PromQL

PromQL是Prometheus自带的查询语言，可以用来查询和处理时间序列数据。

### 4.1 PromQL的数据类型

PromQL主要有四种数据类型：Counter(计数器), Gauge, Histogram(直方图), Summary。

Counter：计数器是只能增加（或重置）的累计指标。这种类型的指标通常用于计算请求总数、任务完成的数量等。

Gauge：Gauge是一个可以任意上下浮动的值。这种类型的指标通常用于度量可以增加和减少的值，如内存使用量、CPU使用率等。

Histogram：直方图是一种可以对值进行采样观察（通常是请求持续时间或响应大小等），并将其划分到可配置的桶中的指标。这种类型的指标通常用于度量在一定时间范围内观察到的事件的大小分布。

Summary：Summary和Histogram类似，也是一种可以对值进行采样观察的指标，但它计算的是滑动时间窗口的百分位数。这种类型的指标通常用于度量某种事件（如请求持续时间）的百分位数。

### 4.2 瞬时向量和区间向量的使用

瞬时向量和区间向量是PromQL中最常用的数据类型。你可以使用瞬时向量来查询当前的数据，使用区间向量来查询过去一段时间的数据。

例如，下面的查询返回的是一个瞬时向量，包含了所有HTTP请求的当前总数：

```promql
http_requests_total
```

下面的查询返回的是一个区间向量，包含了过去5分钟内所有HTTP请求的数据:

```promql
http_requests_total[5m]
```

你可以使用rate()函数来计算区间向量的速率，这个函数返回的是一个瞬时向量。例如，下面的查询返回的是过去5分钟内每分钟的HTTP请求的平均数：

```promql
rate(http_requests_total[5m])
```

注意，rate()函数只能用于区间向量，不能用于瞬时向量。这是因为速率的计算需要多个数据点，而瞬时向量只有一个数据点。

### 4.3 PromQL的基本语法

PromQL的查询可以包括以下几部分：

- **指标名称**：查询的时间序列的指标名称。
- **标签选择器**：用于过滤时间序列的标签。标签选择器可以是`=`（等于）、`!=`（不等于）、`=~`（匹配正则表达式）或`!~`（不匹配正则表达式）。
- **操作符**：用于对时间序列进行算术运算或者比较的操作符。操作符可以是`+`、`-`、`*`、`/`、`%`、`^`、`==`、`!=`、`>`、`<`、`>=`、`<=`、`and`、`or`、`unless`等。
- **函数**：用于对时间序列进行处理的函数。PromQL提供了丰富的函数，包括聚合函数、预测函数、滚动窗口函数等。
- **范围选择器**：用于选择时间序列的一段时间范围。范围选择器是一个时间长度，例如`5m`（5分钟）、`1h`（1小时）等

### 4.4 使用Prometheus的Web界面查询数据

Prometheus自带一个Web界面，你可以在这个界面上输入PromQL查询并查看结果。Web界面的地址是http://<prometheus-server>:9090/graph。

在Web界面上，你可以选择查询的时间范围、查看查询的图形结果、查看查询的原始数据等。

## 5. 设置告警规则

Prometheus支持设置告警规则，当满足某些条件时，Prometheus会发送告警通知。

### 5.1 告警规则的结构和语法

告警规则是通过YAML格式的配置文件进行的。每个告警规则包括`alert`、`expr`、`for`、`labels`和`annotations`等字段。

- `alert`：定义了告警的名称。
- `expr`：定义了触发告警的表达式，这个表达式是一个PromQL查询。
- `for`：定义了告警持续的时间，只有当表达式持续满足这个时间后，才会触发告警。
- `labels`：定义了告警的标签，这些标签会被添加到告警通知中。
- `annotations`：定义了告警的注解，这些注解会被添加到告警通知中。

### 5.2 记录规则和告警规则

Prometheus支持两种类型的规则：记录规则和告警规则。

- 记录规则：记录规则用于预计算常用的或者复杂的表达式，并保存为新的时间序列。记录规则的配置和告警规则类似，但是使用`record`和`expr`字段。
- 告警规则：告警规则用于定义告警条件和告警通知的内容。

### 5.3 告警表达式和持续时间

告警表达式是一个PromQL查询，当这个查询的结果满足一定条件时，会触发告警。告警表达式可以是任何返回瞬时向量的查询。

持续时间定义了告警表达式需要持续满足多长时间才会触发告警。这可以防止短暂的问题触发告警。

### 5.4 告警规则示例和解析

下面是一个告警规则的示例：

```yaml
groups:
- name: example
  rules:
  - alert: HighMemoryUsage
    expr: (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 80
    for: 10m
    labels:
      severity: critical
    annotations:
      summary: High memory usage
```

这个告警规则定义了一个名为HighMemoryUsage的告警，当内存使用率超过80%，并且持续10分钟，就会触发这个告警。告警的严重级别是critical，告警的摘要是High memory usage。

## 6. Prometheus告警管理器Alertmanager

Alertmanager是Prometheus的告警管理器，用于处理由Prometheus服务器发送的告警通知。

### 6.1 Alertmanager的功能

Alertmanager的主要功能包括：

- **分组**：根据告警的标签将相似的告警分组在一起。
- **去重**：对于同一组的多个相同的告警，只发送一次通知。
- **抑制**：当某个告警已经触发时，抑制其他的相关告警。
- **路由**：根据告警的标签将告警路由到不同的接收器。

### 6.2 Alertmanager的配置

Alertmanager的配置是通过YAML格式的配置文件进行的。配置文件主要包括`global`、`route`和`receivers`等部分。

- `global`：定义了全局的配置参数，例如SMTP服务器的地址和端口。
- `route`：定义了告警的路由规则，包括匹配的标签、接收器的名称、子路由等。
- `receivers`：定义了告警的接收器，每个接收器包括一个名称和一组通知配置。

### 6.3 Alertmanager的接收器

Alertmanager支持多种类型的接收器，包括电子邮件、PagerDuty、Slack、Webhook等。每种类型的接收器都有自己的配置参数。

例如，电子邮件接收器需要配置SMTP服务器的地址和端口、发件人的地址、收件人的地址等。Slack接收器需要配置Slack的Webhook URL和通道名称等。

### 6.4 Alertmanager的配置示例

下面是一个Alertmanager配置文件的示例：

```yaml
global:
  smtp_smarthost: 'localhost:25'
  smtp_from: 'alertmanager@example.org'
route:
  group_by: ['alertname', 'cluster', 'service']
  receiver: 'team-X-mails'
receivers:
- name: 'team-X-mails'
  email_configs:
  - to: 'team-X@example.org'
```

这个配置文件定义了一个电子邮件接收器team-X-mails，并将所有的告警路由到这个接收器。告警会被按照alertname、cluster和service标签分组，同一组的告警会被去重和抑制。

## 7. 使用Grafana进行数据可视化

Grafana是一个开源的数据可视化和监控工具，支持Prometheus等多种数据源。

### 7.1 Grafana的主要特性

Grafana的主要特性包括：

- **多数据源**：支持Prometheus、Graphite、InfluxDB等多种数据源。
- **丰富的图表类型**：支持时间序列图、表格、热图、地图等多种图表类型。
- **灵活的仪表板**：支持自定义仪表板和面板，可以配置查询、显示方式、颜色、大小等。
- **告警和通知**：支持设置告警规则和通知方式，可以发送邮件、Slack等通知。

### 7.2 添加Prometheus数据源

在Grafana中，你可以通过以下步骤添加Prometheus数据源：

1. 登录Grafana，点击左侧菜单的"Configuration" > "Data Sources"。
2. 点击"Add data source"，选择"Prometheus"。
3. 输入Prometheus服务器的URL，点击"Save & Test"。

### 7.3 创建仪表板和面板

在Grafana中，你可以通过以下步骤创建仪表板和面板：

1. 点击左侧菜单的"Create" > "Dashboard"。
2. 点击"Add new panel"，配置PromQL查询和显示方式。

### 7.4 配置PromQL查询和显示方式

在Grafana的面板中，你可以配置PromQL查询和显示方式：

- 在"Metrics"区域，输入PromQL查询，选择数据源。
- 在"Visualization"区域，选择图表类型，配置颜色、大小等。

### 7.5 仪表板示例和解析

下面是一个仪表板的示例：

![Grafana Dashboard](https://grafana.com/static/img/docs/v50/dashboard.png)

这个仪表板包括了多个面板，每个面板显示了一个PromQL查询的结果。你可以看到CPU的使用率、内存的使用率、磁盘的使用率等信息。你可以点击面板的标题，修改PromQL查询和显示方式。