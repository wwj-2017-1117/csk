---
keyword: [Knative系统组件, 监控告警, Prometheus]
---

# 设置Knative系统组件监控告警

您可以通过应用实时监控服务中的Prometheus监控设置Knative系统组件的监控指标。本文介绍如何设置Knative系统组件监控告警信息。

Knative系统组件采集指标包括：

-   组件可用实例数
-   组件CPU
-   组件Memory

Knative系统组件包括：

-   knative-serving
    -   activator
    -   autoscaler
    -   autoscaler-hpa
    -   controller
    -   istio-webhook
    -   networking-istio
    -   webhook
-   knative-eventing
    -   eventing-controller
    -   eventing-webhook

## 步骤一：安装Prometheus监控组件

1.  登录[容器服务管理控制台](https://cs.console.aliyun.com)。

2.  在控制台左侧导航栏中，选择**市场** \> **应用目录**。

3.  在应用目录页面的**阿里云应用**页签中，搜索并单击**ack-arms-prometheus**。

4.  在应用目录 - ack-arms-prometheus页面，单击**参数**页签。

5.  在**参数**页签，配置集群ID参数cluster\_id。

    **说明：**

    ACK自动匹配目标集群ID。在控制台左侧导航栏选择**集群**进入**集群列表**，目标集群的名称下面的字符串即为集群ID。

6.  在应用目录 - ack-arms-prometheus右侧的**创建**区域中，选择目标集群，然后单击**创建**。

    **说明：** **命名空间**和**发布名称**默认为**arms-prom**。


## 步骤二：查看Pod监控信息

在进行报警策略设置前，您可以预先查看Pod相关监控信息。

1.  登录[ARMS 控制台](https://arms-intl.console.aliyun.com/)。

2.  在左侧导航栏中单击**Prometheus监控**。

3.  在Prometheus监控页面上，单击**已安装插件**列中的**k8s state**链接，即可在浏览器新窗口中打开对应的监控仪表板，并查看Pod的CPU、Memory，以及Pod个数。


## 步骤三：设置组件报警策略



1.  报警创建提供两个入口，您可根据需要自行选择其中一个入口进入创建报警环节。

    -   在[Prometheus Grafana大盘](http://grafana.console.aliyun.com/)的New DashBoard页面，单击左侧的图标，跳转至Prometheus 创建报警对话框。
    -   在控制台左侧导航栏中选择**报警管理** \> **报警策略管理**，进入报警策略管理页面，在右上角单击**创建报警** \> **Prometheus**。
2.  在弹出的创建报警对话框中，设置相应参数。

    以Knative Serving系统组件activator为例，如果可用Pod数小于期望实例数（即不可用Pod数大于等于1），则进行告警通知。设置告警策略如下图。

    ![报警策略](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/6895659951/p128009.png)

    其中，activator组件PromQL如下。

    ```
    ((sum(kube_deployment_status_replicas{deployment=~"activator"}) or vector(0))) - ((sum(kube_deployment_status_replicas_available{deployment=~"activator"}) or vector(0)))
    ```

    详细的参数说明，请参见[创建报警](/intl.zh-CN/大盘和报警/创建报警.md)。

3.  单击**保存**。


