# 通过YAML使用OSS静态存储卷

本文为您介绍如何通过YAML的方式使用OSS静态存储卷。

[创建存储空间](/cn.zh-CN/快速入门/创建存储空间.md)

**说明：** 相同region的OSS Bucket可以使用内网地址进行挂载。

## 使用Secret创建静态卷PV/PVC

1.  创建Secret。

    以下为通过Secret来配置用户的AccessKey信息的YAML示例文件。

    ```
    apiVersion: v1
    kind: Secret
    metadata:
      name: oss-secret
      namespace: default
    stringData:
      akId: ***
      akSecret: ***
    ```

    **说明：** 创建Secret选择的Namespace需要和应用所在Namespaces一致。

    本示例中，`akId`和`akSecret`需要替换成用户的AccessKeyID和AccessKeySecret。

2.  创建静态卷PV/PVC。

    以下为创建静态卷PV/PVC的YAML示例文件。

    ```
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: oss-pvc
    spec:
      accessModes:
      - ReadWriteMany
      resources:
        requests:
          storage: 5Gi
    ---
    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: oss-csi-pv
    spec:
      capacity:
        storage: 5Gi
      accessModes:
        - ReadWriteMany
      persistentVolumeReclaimPolicy: Retain
      csi:
        driver: ossplugin.csi.alibabacloud.com
        volumeHandle: oss-csi-pv // 需要和PV名字一致。
        nodePublishSecretRef:
          name: oss-secret
          namespace: default
        volumeAttributes:
          bucket: "oss"
          url: "oss-cn-hangzhou.aliyuncs.com"
          otherOpts: "-o max_stat_cache_size=0 -o allow_other"
    ```

    |参数|描述|
    |--|--|
    |`nodePublishSecretRef`|定义挂载PV时通过Secret对象来获取AccessKey信息。|
    |`volumeHandle`|配置PV的名称。|
    |`bucket`|需要挂载的OSS Bucket。|
    |`url`|挂载OSS的接入域名，挂载节点和Bucket相同region时，可使用内网地址。|
    |`otherOpts`|挂载OSS时支持输入定制化参数，格式为：`-o *** -o ***`。|


## 配置PV/PVC的AccessKey

[\#d7e18](#d7e18)是通过Secret为plugin提供AccessKey信息，您也可以直接在PV中配置AccessKey信息。示例YAML文件如下。

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: oss-csi-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  csi:
    driver: ossplugin.csi.alibabacloud.com
    volumeHandle: oss-csi-pv // 需要和pv名字一致；
    volumeAttributes:
      bucket: "oss"
      url: "oss-cn-hangzhou.aliyuncs.com"
      otherOpts: "-o max_stat_cache_size=0 -o allow_other"
      akId: "***"
      akSecret: "***"
```

## 配置PV/PVC的STS权限

除了[\#d7e18](#d7e18)和[\#d7e92](#d7e92)中使用Secret和使用AK两种方式进行权限认证外，您还可以在PV中配置STS进行权限认证，示例YAML文件如下。

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: oss-csi-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  csi:
    driver: ossplugin.csi.alibabacloud.com
    volumeHandle: oss-csi-pv // 需要和pv名字一致；
    volumeAttributes:
      bucket: "oss"
      url: "oss-cn-hangzhou.aliyuncs.com"
      otherOpts: "-o max_stat_cache_size=0 -o allow_other"
      authType: "sts"
```

## 创建应用

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
        volumeMounts:
          - name: oss-pvc
            mountPath: "/data"
      volumes:
        - name: oss-pvc
          persistentVolumeClaim:
            claimName: oss-pvc
```

您也可以通过控制台的方式使用OSS静态存储卷，请参见[通过控制台使用OSS静态存储卷]()。

