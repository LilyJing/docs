## 第二节 集成后端服务   

在集成后端服务之前，我们先查看上一节 WordPress 的部署结果  

```
$ oc get pods
  NAME                 READY     STATUS             RESTARTS   AGE
  wordpress-1-build    0/1       Completed          0          3m
  wordpress-1-deploy   1/1       Running            0          1m
  wordpress-1-hfzhs    0/1       CrashLoopBackOff   3          1m
``` 

我们现在来创建 MySQL Backing Service

### 查看 DataFoundry 后端服务列表  

在通过 DataFoundry 生成一个 MySQL 后端服务之前，我们可以先查看一下目前 DataFoundry 已集成的后端服务
  
```   
$ oc get bs -n openshift  
  NAME         LABELS                           BINDABLE   STATUS
  Cassandra    asiainfo.io/servicebroker=etcd   true       Active
  ETCD         asiainfo.io/servicebroker=etcd   true       Active
  Greenplum    asiainfo.io/servicebroker=rdb    true       Active
  Kafka        asiainfo.io/servicebroker=etcd   true       Active
  MongoDB      asiainfo.io/servicebroker=rdb    true       Active
  Mysql        asiainfo.io/servicebroker=rdb    true       Active
  PostgreSQL   asiainfo.io/servicebroker=rdb    true       Active
  RabbitMQ     asiainfo.io/servicebroker=etcd   true       Active
  Redis        asiainfo.io/servicebroker=etcd   true       Active
  Spark        asiainfo.io/servicebroker=etcd   true       Active
  Storm        asiainfo.io/servicebroker=etcd   true       Active
  ZooKeeper    asiainfo.io/servicebroker=etcd   true       Active
```

注意：   
> 后端服务是 DataFoundry 特有功能，所以必须使用 DataFoundry 客户端连接 DataFoundry 服务端后查看
在查看 DataFoundry 平台已集成的后端服务时，要添加后端服务默认的集成命名空间 openshift

### 创建后端服务实例

在创建实例之前，我们要先通过 `oc describe bs <backingservcie-name> ` 获取应用所需的后端服务计划（plan），例如我们获取 MySQL 后端服务的服务计划为：

```   
$ oc describe bs Mysql -n openshift
  Name:					Mysql
  Created:				21 hours ago
  Labels:				asiainfo.io/servicebroker=rdb
  Annotations:			<none>
  Description:			A MYSQL DataBase
  Status:				Active
  Bindable:				true
  Updateable:			false
  documentationUrl:		http://docs.mysql.com
  longDescription:		OpenSoure RDMBS Mysql
  providerDisplayName:	asiainfoLDP
  supportUrl:			http://www.mysql.com
  displayName:			Mysql
────────────────────
  Plan:					Experimental
  PlanID:				56660431-6032-43D0-A114-FFA3BF521B66
  PlanDesc:				share a mysql database in one instance
  PlanFree:				true
  Bullets:
    20 GB of Disk
    20 connections
  PlanCosts:
    CostUnit:			MONTHLY
    Amount:
      eur: 49
      usd: 99
    CostUnit:			1GB of messages over 20GB
    Amount:
      eur: 0.49
      usd: 0.99
```

我们选取 Experimental 计划创建 MySQL 后端实例

```   
oc new-backingserviceinstance mysql-inst1 \
--backingservice_name=Mysql \
--planid=56660431-6032-43D0-A114-FFA3BF521B66
Backing Service Instance has been created.
```

查看后端服务列表

```   
oc get bsi
NAME          SERVICE   PLAN           BOUND     STATUS
mysql-inst1   Mysql     Experimental   0         Unbound 
```

查看后端服务详细信息
```
oc describe bsi mysql-inst1
Name:			mysql-inst1
Created:		5 minutes ago
Labels:			<none>
Annotations:		<none>
Status:			Unbound
DashboardUrl:		http://e412377c9b5f3db:1394e5f077d1519@phpmyadmin-service-broker-db.app.dataos.io?db=8c60f229ef4dac0
BackingServiceName:	Mysql
BackingServicePlanName:	Experimental
BackingServicePlanGuid:	56660431-6032-43D0-A114-FFA3BF521B66
Parameters:
instance_id:	340082e4-1734-11e6-a653-fa163edcfb45
Bound:		0
Events:
  FirstSeen	LastSeen	Count	From	SubobjectPath	Type		Reason		Message
  ---------	--------	-----	----	-------------	--------	------		-------
  5m		5m		1	{bsi }			Normal		Provisioning	bsi provisioning done, instanceid: 340082e4-1734-11e6-a653-fa163edcfb45
```

### 绑定后端服务实例

以上服务实例创建完成，我们继续把 mysql backing service 绑定到 workdpress 应用中
```
oc bind mysql-inst1 wordpress
oc env dc/wordpress MYSQLBSI=MYSQLINST1
```  
查看部署结果，Pod 已正常启动，给 wordpress 生成一个 route 地址后就可以访问了

```
oc get pods
NAME                READY     STATUS      RESTARTS   AGE
wordpress-1-build   0/1       Completed   0          7m 
wordpress-2-55q5a   1/1       Running     0          41s
```  
  
## 通过界面方式创建后端服务的过程为  
1. 点击“后端服务” 可以看到目前 DataFoundry 所能提供的各类后端服务
 ![](../img/Screenshot from 2016-05-16 18-27-54.png)

1. 点击“申请实例”，填入“服务名称”，在选择“服务套餐”后点击“创建”
![](../img/Screenshot from 2016-05-12 18-41-15.png)

1. 后端服务创建完成后，可以显示创建后端服务详细信息
![](../img/Screenshot from 2016-05-12 18-41-36.png)

1. 回到“后端服务”，点击“我的后端服务实例” Tab 页，在这里可以看到所有已创建的后端服务实例
![](../img/Screenshot from 2016-05-16 18-37-04.png)

1. 点击“服务绑定”按钮，在窗口中点击“新增绑定”
![](../img/Screenshot from 2016-05-16 18-38-15.png)

1. 在弹出窗口中选择需要绑定后端服务的应用，后端服务与应用绑定完成








