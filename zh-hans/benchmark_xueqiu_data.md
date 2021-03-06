# 雪球模拟数据的顺序插入测试

## 1. 数据样例

```
152894879800009_sh78338, 2, 2, 130, 2
152894879800011_sh13307, 2, 1, 1, 1
152894879800013_sh33269, 2, 2, 2, 1
152894879800013_sh46940, 120, 0, 2, 1
152894879800015_sh34942, 130, 120, 1, 2
152894879800017_sh79776, 1, 2, 120, 1
152894879800018_sh26834, 120, 0, 2, 0
152894879800018_sh94212, 0, 0, 2, 0
152894879800019_sh85115, 1, 120, 2, 0
152894879800022_sh75017, 1, 1, 1, 0
152894879800023_sh72599, 2, 1, 0, 2
152894879800024_sh45429, 2, 1, 1, 0
152894879800024_sh48580, 0, 2, 1, 2
152894879800029_sh32673, 2, 0, 1, 0
152894879800038_sh83764, 1, 1, 2, 2
152894879800039_sh67403, 2, 2, 1, 1
152894879800039_sh98918, 2, 1, 2, 1
152894879800040_sh39662, 0, 0, 2, 2
152894879800045_sh19990, 0, 2, 2, 0
152894879800051_sh18314, 0, 1, 0, 1
```

共 **69,931,112** 条, 总大小为 **2.37G**, 平均每条 **36.4 bytes**。数据已经排好序。

## 2. 工具

使用 ```load data infile``` 命令和 mysqlomport 工具。

以下时间均使用 time 统计得来。

## 3. 结果

### 3.1 load data infile

#### 3.1.1 关闭 unique_checks

测试命令:
```
time mysql -uroot -h127.0.0.1 -P660x -e "use xueqiu; set unique_checks=OFF; \
    show variables like 'unique_checks'; \
    LOAD DATA INFILE '/disk2/data/xueqiu_lmitated_data_sort.txt' \
    INTO TABLE xueqiu_lmitated_data FIELDS TERMINATED BY ',';"
```

| 项目 | 第一次 | 第二次 | 第三次 | 第四次 |
|:---|:-----|:-----|:-----|:-----|
| mysql                      | 6m6.942s  |           |           |           |
| terarksql(skiplist)        | 4m35.630s | 4m44.055s |           |           |
| terarksql(patricia-rbtree) | 4m52.856s | 4m50.501s |           |           |
| terarksql(patricia-vector) | 4m55.377s | 4m56.572s | 4m40.203s | 4m27.000s |
| terarksql(patricia-batch)  | 4m21.541s | 4m27.223s |           |           |
| terarksql(patricia-final)  | 3m53.294s | 4m19.709s | 3m43.295s |           |

#### 3.1.2 打开 unique_checks

测试命令:
```
time mysql -uroot -h127.0.0.1 -P660x -e "use xueqiu; show variables like 'unique_checks'; \
    LOAD DATA INFILE '/disk2/data/xueqiu_lmitated_data_sort.txt' \
    INTO TABLE xueqiu_lmitated_data FIELDS TERMINATED BY ',';"
```

| 项目 | 第一次 |
|:----|:------|
| mysql                     |
| terarksql(skiplist)       |
| terarksql(patricia-final) |

### 3.2 mysqlimport

#### 3.1.1 关闭 unique_checks

测试命令:
```
time mysqlimport --columns=id,num1,num2,num3,num4 --fields-terminated-by=, \
    -h127.0.0.1 -uroot -P660x xueqiu \
    --local /disk2/data/xueqiu_lmitated_data_sort.txt
```

测试前使用 ```set global unique_checks=OFF``` 关闭 unique_checks

| 项目 | 第一次 | 第二次 |
|:----|:------|:------|
| mysql                     | 6m13.570s | 6m26.699s |
| terarksql(skiplist)       |           |           |
| terarksql(patricia-final) | 3m57.990s | 3m42.092s |

#### 3.1.2 打开 unique_checks

测试命令:
```
time mysqlimport --columns=id,num1,num2,num3,num4 --fields-terminated-by=, \
    -h127.0.0.1 -uroot -P660x xueqiu --replace \
    --local /disk2/data/xueqiu_lmitated_data_sort.txt
```
| 项目 | 第一次 | 第二次 |
|:----|:------|:------|
| mysql                     | 6m40.679s |
| terarksql(skiplist)       | 12m55.138s | 12m54.280s |
| terarksql(patricia-final) |  9m13.138s |  8m51.860s |
