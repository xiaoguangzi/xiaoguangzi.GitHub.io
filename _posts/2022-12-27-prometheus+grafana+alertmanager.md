## prometheus

Prometheus受启发于Google的Brogmon监控系统（相似的Kubernetes是从Google的Brog系统演变而来），从2012年开始由前Google工程师在Soundcloud以开源软件的形式进行研发，并且于2015年早期对外发布早期版本。2016年5月继Kubernetes之后成为第二个正式加入CNCF基金会的项目，同年6月正式发布1.0版本。2017年底发布了基于全新存储层的2.0版本，能更好地与容器平台、云平台配合。

![prometheus](https://2584451478-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LBdoxo9EmQ0bJP2BuUi%2F-LPMFlGDFIX7wuLhSHx9%2F-LPMFo9ZTdKYHyFzu4DJ%2Fprometheus-release-roadmaps.png?generation=1540136064641479&alt=media)

**prometheus 下载安装**

<https://prometheus.io/download/>

Prometheus一般需要搭配多种组件使用，像alert manager，node_exporter，grafana等等。官方提供了很多种exporter，可以根据自己的需要去下载，有些时候需要我们自定义exporter的时候，可以根据开发语言选择jmx_exporter和blackbox_exporter。

```shell
tar xzf prometheus-2.41.0.linux-amd64.tar.gz
mv prometheus-2.41.0.linux-amd64 /usr/local
cd /usr/local/prometheus-2.41.0.linux-amd64
./prometheus -h
```

*--web.listen-address="0.0.0.0:9090"* //prometheus监听地址，默认是9090，如果有需要的话可以自己更改

*--config.file="prometheus.yml"*      //prometheus配置文件，默认使用prometheus.yml

*--web.enable-lifecycle*              //启用该参数可以通过HTTP POST重启Prometheus server // curl -X POST http://127.0.0.1:9090/-/reload

```shell
cat prometheus.yml

# 单独设置全局设置会被覆盖
global:
  scrape_interval:     15s # 全局默认的数据拉取间隔
  evaluation_interval: 15s # 全局默认的规则(主要是报警规则)拉取间隔
  scrape_timeout: 10s # 全局默认的单次数据拉取超时

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets: ['192.168.64.10:9093']

rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

scrape_configs:
  - job_name: 'linux_node'
    file_sd_configs:
    - files:
      - '/mnt/opt/prometheus/prometheus-2.17.1.linux-amd64/file_sd/node.json'
  - job_name: 'hadoop'
    static_configs:
    - targets:
      - '192.168.64.8:6688'

# file_sd_configs 基于文件的服务发现提供了一种更通用的方式来配置静态目标，并用作插入自定义服务发现机制的接口,josn格式如下

[
  {
    "targets": [ "<host>", ... ],
    "labels": {
      "<labelname>": "<labelvalue>", ...
    }
  },
  ...
]

# 也可以是yaml格式

- targets:
  [ - '<host>' ]
  labels:
    [ <labelname>: <labelvalue> ... ]

cat rules/node_rules.yml

groups:
  - name: admin
    rules:
    - alert: InstanceDown # 告警名称
      expr: up == 0       # 告警的判定条件，参考Prometheus高级查询来设定
      for: 15s            # 满足告警条件持续时间多久后，才会发送告警
      labels:
        team: admin_team
      annotations:        # 解析项，详细解释告警信息
        summary: "{{$labels.instance}}: has been down"
        description: "{{$labels.instance}}:has been down"
        value: "{{$value}}"


nohup ./prometheus --config.file=prometheus.yml --web.enable-lifecycle &

```

参考文档： <https://www.jianshu.com/p/872bafe8d85e>

## grafana安装配置

<https://grafana.com/grafana/download>

```shell
systemctl start grafana-server # centos系列使用systemd作为启动管理

service grafana-server start   # Ubuntu以及centos6系列启动方式

访问http://ip:3000登录grafana页面，默认用户密码为admin/admin。

添加数据源，选择Prometheus作为数据源。settings 配置如下：

URL: http://192.168.64.8:9090

Access： Server(default)

dashboards选择grafana metrics导入

```

## alertmanager安装配置

<https://prometheus.io/download/>

```shell
tar xzf alertmanager-0.25.0.linux-amd64.tar.gz

vi alertmanager.yml

> 全局配置（global）：用于定义一些全局的公共参数，如全局的SMTP配置，Slack配置等内容；
> 模板（templates）：用于定义告警通知时的模板，如HTML模板，邮件模板等；
> 告警路由（route）：根据标签匹配，确定当前告警应该如何处理；
> 接收人（receivers）：接收人是一个抽象的概念，它可以是一个邮箱也可以是微信，Slack或者Webhook等，接收人一般配合告警路由使用；
> 抑制规则（inhibit_rules）：合理设置抑制规则可以减少垃圾告警的产生

global:
  resolve_timeout: 5m # 
  # smtp配置
  smtp_from: "1234567890@qq.com" # 发送邮件主题
  smtp_smarthost: 'smtp.qq.com:465' # 邮箱服务器的SMTP主机配置
  smtp_auth_username: "1234567890@qq.com" # 登录用户名
  smtp_auth_password: "auth_pass" # 此处的auth password是邮箱的第三方登录授权密码，而非用户密码，尽量用QQ来测试。
  smtp_require_tls: false # 有些邮箱需要开启此配置，这里使用的是163邮箱，仅做测试，不需要开启此功能。
route:
  receiver: ops
  group_wait: 30s # 在组内等待所配置的时间，如果同组内，30秒内出现相同报警，在一个组内出现。
  group_interval: 5m # 如果组内内容不变化，合并为一条警报信息，5m后发送。
  repeat_interval: 24h # 发送报警间隔，如果指定时间内没有修复，则重新发送报警。
  group_by: [alertname]  # 报警分组
  routes:
      - match:
          team: operations
        group_by: [env,dc]
        receiver: 'ops'
      - receiver: ops # 路由和标签，根据match来指定发送目标，如果 rule的lable 包含 alertname， 使用 ops 来发送
        group_wait: 10s
        match:
          team: operations
# 接收器指定发送人以及发送渠道
receivers:
# ops分组的定义
- name: ops
  email_configs:
  - to: '9935226@qq.com,xxxxx@qq.com' # 如果想发送多个人就以 ','做分割，写多个邮件人即可。
    send_resolved: true
    headers:
      from: "警报中心"
      subject: "[operations] 报警邮件"
      to: "小煜狼皇"

  wechat_configs:
  - corp_id: 'wwxxxxx' # 企业ID是唯一标识
    api_url: 'https://qyapi.weixin.qq.com/cgi-bin/' # 企业微信api接口，统一定义
    send_resolved: true # 设置发送警报恢复信息
    to_party: '2' # 部门id，比如我的叫警报组，因此显示的是2，如果你DB组，就可能会是3，WEB组就是4，依次类推，另外需要接收警报的相关人员必须在这个部门里。
    agent_id: '1000004' # 新建应用的agent_id
    api_secret: 'F-fzpgsabmfiFt7_4QRQwWEl8eyx7evO12sRYe_Q5vA' # 新建应用的Secret

  webhook_configs:
  - url: 'https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx'
```

```shell

cat tmaplates/wchat.tmpl

<https://zhuanlan.zhihu.com/p/179294441>

```

参考资料：

<https://yunlzheng.gitbook.io/prometheus-book/>

<https://zhuanlan.zhihu.com/p/179294441>

<https://blog.51cto.com/starsliao/5763175>

<https://cloud.tencent.com/developer/article/1808130>
