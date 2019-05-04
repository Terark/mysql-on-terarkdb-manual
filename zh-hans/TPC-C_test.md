## 简介

TPC-C是一个联机事物处理基准，tpcc-mysql是percona基于TPC-C衍生出来的产品，专用于mysql基准测试。TPC-C测试规范中模拟了一个比较复杂并具有代表意义的OLTP应用环境：假设有一个大型商品批发商，它拥有若干个分布在不同区域的商品库；每个仓库负责为10个销售点供货；每个销售点为3000个客户提供服务；每个客户平均一个订单有10项产品；所有订单中约1%的产品在其直接所属的仓库中没有存货，需要由其他区域的仓库来供货。

该系统需要处理的交易为以下几种： 

* New-Order：客户输入一笔新的订货交易； 
* Payment：更新客户账户余额以反映其支付状况; 
* Delivery：发货(模拟批处理交易); 
* Order-Status：查询客户最近交易的状态； 
* Stock-Level：查询仓库库存状况，以便能够及时补货。 

对于前四种类型的交易，要求响应时间在5秒以内；对于库存状况查询交易，要求响应时间在20秒以内。

测试程序源码使用在bazaar上管理的版本。需要先安装bazaar客户端。然后可使用如下命令下载源码：

```shell
bzr branch lp:~percona-dev/perconatools/tpcc-mysql
```

测试的数据库有：[MySQL on TerarkDB](http://terark.com/docs/mysql-on-terarkdb-manual/zh-hans/installation.html) （下简称 TerarkDB），官方原版 MySQL（下简称 InnoDB）。MySQL 开启压缩。

## 测试平台

- CPU: Intel(R) Xeon(R) CPU E5-2630 v3 @ 2.40GHz x2 （共 16 核 32 线程）
- 内存: DDR4 16G @ 1866 MHz x 12 （共 192 G）
- SSD: INTEL SSDSC2BP48 0420 IOPS 89000
- 操作系统: CentOS 7

测试中使用的官方原版 MySQL 版本为 Ver 5.6.35 for linux-glibc2.5 on x86_64。

下文 G, GB 指 230，而非 109。

##数据导入

## 测试结果

![tpcc8G](tpcc8G.PNG)
