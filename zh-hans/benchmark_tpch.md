
## MyRocks + Terark

### 插入阶段

插入表结构及配置参见附注。

共 120G 数据，插入耗时 3.5 小时。初次插完空间占用为 57G，经过 compaction 后大小为 35G，插入过程中需要总空间 ~ 64G。


### 查询阶段

使用测试程序 [MyRocks](https://github.com/Terark/MyRocksTest), 32 线程，


| 内存限制*   |  QPS   | CPU 占用  | TPS(iostat) | kB_read(iostat) |
|----------:|-------:|---------:|------------:|----------------:|
| 不限内存   | 97,000 | 2400%    |  517        |  2,068          |
| 限制为 32G | 95,000 | 2400%    |  349        |  1,396          |
| 为 24G    | 78,000 | 2000%    |  17,000     |  68,104         |
| 为 16G    | 47,000 | 1400%    |  44,405     |  177,620        |
| 为 8G     | 28,000 | 900%     |  48,000     |  195,016        |

注：TerarkDB 使用 mmap，所以这里使用了 cgroup 的方式进行内存限制。

## MyRocks + InnoDB

### 插入阶段

插入表结构及配置参见附注。

共 120G 数据，插入 21 小时共插入 90% 左右的数据，大小 210G 左右。期间 kill -9 后重启，耗时1小时10分钟；（注：没有设置 ```buffer_pool_size``` 也即使用的是默认配置）


### 查询阶段

使用测试程序 [MyRocks](https://github.com/Terark/MyRocksTest), 使用 32个 线程，buffer_cache 设置为 96G，

| 内存限制*  |  QPS   | CPU 占用  | TPS(iostat) | kB_read(iostat) |
|----------:|-------:|---------:|------------:|----------------:|
| 不限内存**  | 47,000 |  900%   |             |                 |
| 限制为 96G | 28,000 |  470%    |  14,995     |  473,572        |
| 为 32G    | 19,000 |  400%    |  18,326     |  450,656        |
| 为 16G    | 14,500 |  350%    |  19,076     |  438,828        |
| 为 8G     | 14,000 |  400%    |  22,051     |  422,212        |
 
注* : 系统总计内存 188G，不足以装下所有数据。这里 “不限内存” 与 “限制为 96G” 对应的```buffer_pool_size``` 设置为 96G。其余测试的 ```buffer_pool_size```均为限制后系统内存的一半。

注** : innodb 使用 pread/aio 读取数据，而 cgroup 无法限制操作系统 filesystem cache，所以通过操作系统启动参数进行内存限制。

## 附注

建表如下，从 lineitem1 到 lineitem100 共 100 张表，

```
CREATE TABLE lineitem  (
             L_ORDERKEY    	BIGINT NOT NULL,
             L_PARTKEY     	BIGINT NOT NULL,
             L_SUPPKEY     	INTEGER NOT NULL,
             L_LINENUMBER  	INTEGER NOT NULL,
             L_QUANTITY    	DECIMAL(15,2) NOT NULL,
             L_EXTENDEDPRICE    DECIMAL(15,2) NOT NULL,
             L_DISCOUNT    	DECIMAL(15,2) NOT NULL,
             L_TAX         	DECIMAL(15,2) NOT NULL,
             L_RETURNFLAG  	CHAR(1) NOT NULL,
             L_LINESTATUS  	CHAR(1) NOT NULL,
             L_SHIPDATE    	DATE NOT NULL,
             L_COMMITDATE  	DATE NOT NULL,
             L_RECEIPTDATE 	DATE NOT NULL,
             L_SHIPINSTRUCT 	 CHAR(25) NOT NULL,
             L_SHIPMODE     	 CHAR(10) NOT NULL,
             L_COMMENT      	 VARCHAR(512) NOT NULL,
             PRIMARY KEY      (L_ORDERKEY, L_PARTKEY),
             KEY 		(L_SHIPDATE, L_ORDERKEY),
             KEY 		(L_ORDERKEY, L_SUPPKEY),
             KEY 		(L_COMMITDATE, L_PARTKEY),
             KEY 		(L_PARTKEY, L_ORDERKEY),
             KEY 		(L_PARTKEY, L_SUPPKEY),
             KEY 		(L_SUPPKEY, L_RECEIPTDATE),
             KEY 		(L_SUPPKEY, L_ORDERKEY),
             KEY 		(L_SUPPKEY, L_PARTKEY)
);
```

MyRocks + Terark 的 my.cnf 配置

```
rocksdb
default-storage-engine=rocksdb
skip-innodb
default-tmp-storage-engine=MyISAM
character-set-server=utf8
collation-server=utf8_bin

user = wangfo
bind-address = 0.0.0.0
port = 3307

back_log = 600
max_connections = 6000

#binlog-format=ROW
secure_file_priv=""
rocksdb_commit_in_the_middle=ON
rocksdb_bulk_load_size=1000

table_open_cache = 21397
rocksdb_default_cf_options=memtable=rbtree

```

Terark 的环境变量设置

```
TerarkZipTable_localTempDir=$PWD/terark-temp \
TerarkZipTable_keyPrefixLen=4 \
TerarkZipTable_offsetArrayBlockUnits=128 \
TerarkZipTable_extendedConfigFile=$PWD/license \
TerarkUseDivSufSort=1 \
TerarkZipTable_max_background_compactions=5 \
TerarkZipTable_max_subcompactions=1 \
TerarkZipTable_min_merge_width=3 \
TerarkZipTable_max_merge_width=7 \
TerarkZipTable_level0_file_num_compaction_trigger=4 \
TerarkZipTable_softZipWorkingMemLimit=2G \
TerarkZipTable_hardZipWorkingMemLimit=4G \
TerarkZipTable_write_buffer_size=1G \
TerarkZipTable_target_file_size_base=2G \
TerarkZipTable_target_file_size_multiplier=1 \
TerarkZipTable_indexCacheRatio=0 \
TerarkZipTable_warmUpIndexOnOpen=false \
TerarkZipTable_sampleRatio=0.015 \
TerarkZipTable_disableFewZero=true \
TerarkZipTable_enable_partial_remove=true \
Terark_enableChecksumVerify=0 \
```

MyRocks + InnoDB 的 my.cnf 配置

```
```

