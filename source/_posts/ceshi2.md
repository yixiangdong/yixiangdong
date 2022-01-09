---
title: 阿里云Dataworks+Maxcompute实时计算数据仓库方案之[精华]业务数仓搭建(二)
date: 2020-03-20 03:57:06
tags: 数据仓库
comments: true
---

# 业务数仓搭建 

## 业务数仓架构图 

### 业务数仓系统流程设计 

![系统业务流程设计](1584648329928.png)
<!-- more -->
### 业务表结构 

![业务表结构](1584884841420.png)

### 开发界面

![开发界面](1584893428690.png)

### 业务数仓分层 

业务数仓分层 

### ODS层

![业务数仓分层 ](1584884954626.png)

1) 在临时查询中统一执行建表 

```mysql
CREATE TABLE `ods_order_info_di` (
`id` string COMMENT '订单编号',
`total_amount` double COMMENT '订单金额',
`order_status` string COMMENT '订单状态',
`user_id` string COMMENT '用户 id',
`payment_way` string COMMENT '支付方式',
`out_trade_no` string COMMENT '支付流水号',
`create_time` string COMMENT '创建时间',
`operate_time` string COMMENT '操作时间',
`province_id` string COMMENT '省份'
)
COMMENT '订单表'
PARTITIONED BY (ds string);

CREATE TABLE `ods_order_detail_di` (
`id` string COMMENT '明细 id',
`order_id` string COMMENT '订单 id',
`sku_id` string COMMENT '商品 id',
`sku_name` string COMMENT '商品名称',
`order_price` double COMMENT '购买价格',
`sku_num` bigint COMMENT '购物数量',
`create_time` string COMMENT '创建时间'
)
COMMENT '订单明细'
PARTITIONED BY (ds string);

CREATE TABLE `ods_sku_info_df` (
`id` string COMMENT 'skuid',
`spu_id` string COMMENT 'spuid',
`price` double COMMENT '价格',
`sku_name` string COMMENT '商品名称',
`sku_desc` string COMMENT '商品描述',
`weight` double COMMENT '重量(千克)',
`tm_id` string COMMENT '品牌 id',
`category3_id` string COMMENT '品类 id',
`create_time` string COMMENT '创建时间'
)
COMMENT '商品信息'
PARTITIONED BY (ds string);

CREATE TABLE `ods_user_info_df` (
`id` string COMMENT '用户 id',
`name` string COMMENT '姓名',
`birthday` string COMMENT '生日',
`gender` string COMMENT '性别',
`email` string COMMENT '邮箱',
`user_level` string COMMENT '用户等级',
`create_time` string COMMENT '创建时间'
)
COMMENT '用户信息'
PARTITIONED BY (ds string);

CREATE TABLE `ods_base_category3_df` (
`id` string COMMENT '三级品类 id',
`name` string COMMENT '名称',
`category2_id` string COMMENT '二级品类 id'
)
COMMENT '三级品类信息'
PARTITIONED BY (ds string);

CREATE TABLE `ods_base_trademark_df` (
`tm_id` string COMMENT '品牌 id',
`tm_name` string COMMENT '名称'
)
COMMENT '品牌信息'
PARTITIONED BY (ds string);

CREATE TABLE `ods_base_category1_df` (
`id` string COMMENT '一级品类 id',
`name` string COMMENT '名称'
)
COMMENT '一级品类信息'
PARTITIONED BY (ds string);

CREATE TABLE `ods_base_category2_df` (
`id` string COMMENT '二级品类 id',
`name` string COMMENT '名称',
`category1_id` string COMMENT '一级品类 id'
)
COMMENT '二级品类信息'
PARTITIONED BY (ds string);

CREATE TABLE `ods_payment_info_di` (
`id` bigint COMMENT '编号',
`out_trade_no` string COMMENT '对外业务编号',
`order_id` string COMMENT '订单编号',
`user_id` string COMMENT '用户编号',
`alipay_trade_no` string COMMENT '支付宝交易流水编号',
`total_amount` double COMMENT '支付金额',
`subject` string COMMENT '交易内容',
`payment_type` string COMMENT '支付类型',
`payment_time` string COMMENT '支付时间'
)
COMMENT '支付流水表'
PARTITIONED BY (ds string);

CREATE TABLE `ods_base_region_df` (
`id` bigint COMMENT '地区 id',
`region_name` string COMMENT '地区名称'
)
COMMENT '地区'
PARTITIONED BY (ds string);

CREATE TABLE `ods_base_province_df` (
`id` bigint COMMENT '品牌 id',
`name` string COMMENT '名称',
`region_id` bigint COMMENT '地区 id'
)
COMMENT '省份'
PARTITIONED BY (ds string);
```

2) 在表管理里面查看创建的表

![查看建的表](1584885516491.png)

3）数据开发->ods->导入表 

![导入表 ](1584885569395.png)

4）选择刚创建的所有表 

![选择刚创建的所有表](1584885612740.png)

### 数据同步 

目前 MySQL 里面的数据已经有了， ODS 层表也已经建好，现在需要创建一个脚本，将MySQL 中数据同步到 ODS 层对应的表。

#### 建立数据同步节点 

1）数据集成->新建数据集成节点->数据同步 

![数据同步](1584885796233.png)

2）填写节点名称（ 表名+后缀， 见名知意就好，例如： ods_user_info_sync） 

![表名+后缀](1584885851653.png)

3）鼠标悬浮在选择数据源右侧的问号上->数据源进行新建操作 

![新建数据源](1584885896615.png)

4） 点击新增数据源->点击 MySQL 

![MySQL数据源](1584885964562.png)

5） 配置新增 MySQL 数据源 

![配置新增 MySQL数据源](1584886004636.png)

（ 1）数据源名称：可以任意取，这里取的 gmall_rds
（ 2） RDS 实例 ID： 根据问号的提示获取， 每个人的不一样。 rm-bp1t1li837a1v9gb8 

（3） RDS 实例主账号 ID： 根据问号提示获取，每个人的不一样。 1902761552218725 

（ 4）数据库名： 要连接的数据库名称， 我这里是 gmall
（ 5）用户名、密码：要连接的数据库的用户名和密码
（ 6）白名单配置：因为要用 MaxCompute 访问 RDS 所以要给 RDS 增加对应的白名单IP 地址。 

a）点击点我查看如何添加白名单按钮 

![添加白名单按钮](1584886175005.png)

b）根据购买的 RDS 服务器地址，选择对应 IP 地址 

![选择对应 IP 地址](1584886209804.png)

c）在 RDS 中添加白名单 

![添加白名单](1584886436237.png)

![添加白名单](1584886474174.png)

6） 配置完新增 MySQL 数据源， 就会在数据集成窗口发现增加了一个 gmall_rds 数据源 

![检查添加的数据源](1584886528871.png)

7）再次回到 DataStudio 工作空间，发现就可以选择数据源了 

![在DataStudio工作空间选择数据源](1584886574879.png)

7）配置数据去向数据源->鼠标悬浮问号上面->点击数据源进行新建操作->新增数据源
MaxCompute（ ODPS） 

![新增数据源MaxCompute](1584886662733.png)

![新增数据源MaxCompute](1584886701603.png)

8）配置 MaxCompute 数据源详情 

![配置 MaxCompute 数据源详情](1584886919588.png)

![查看配置的MaxCompute 数据源](1584886947876.png)

### 每日全量表同步 

用户表同步策略：每日全量
每 日 全 量 导 入 的 表 包 括 ： 

ods_user_info 、 

ods_base_category1

 ods_base_category2
ods_base_category3

ods_base_province

 ods_base_region

ods_base_trademark

 ods_sku_info
1）配置完数据源后->点击运行（点击运行前，检查字段映射关系） 

![检查字段映射关系](1584887064689.png)

![检查字段映射关系](1584887104680.png)

![检查字段映射关系](1584887150706.png)

![自定义参数](1584887172517.png)

2）临时查询，验证结果 

```
select * from ods_user_info_df where ds='20191008'; 
```

![临时查询结果](1584887227050.png)

3）重复执行步骤 1-2，依次导入： 

ods_base_category1

ods_base_category2

ods_base_category3
ods_base_province

ods_base_region

ods_base_trademark

ods_sku_info

### 每日增量表同步 

1）同步策略：每日增量 

每日新增的表包括： ods_order_detail

![每日新增的表](1584887595660.png)

每日增量的区别就是要按照日期进行过滤，只筛选出今天新产生的数据条件： 

DATE_FORMAT(create_time,'%Y%m%d')='${bizdate}'   

注意：此处必须是 MySql 的函数语法，不是 Hive 的。 ${bizdate}是系统自带参数用于取前一天的日期。 

$[bizdate]是系统自带参数用于取当天的日期。

2）临时查询，验证结果 

select * from ods_order_detail_di WHERE ds='20191008'; 

### 每日新增及变化表同步

1）同步策略：每日新增及变化 

每日新增及变化的表包括： ods_order_info

![每日新增及变化的表](1584887862971.png)

条件

DATE_FORMAT(create_time,'%Y%m%d')='${bizdate}' or
DATE_FORMAT(operate_time,'%Y%m%d')='${bizdate}' 

2）临时查询，验证结果 

select * from ods_order_info_di WHERE ds='20191008'; 

![查询结果](1584888013602.png)

ODS 层调度

把节点从左侧拖放至右侧，按照上下游的关系用线连接好。 

![连线](1584888088744.png)

### DWD 层 

DWD 层，一般是对 ODS 层数据进行一定的清洗加工，如果是面对关系导入过来的数据表，还要把原本的关系型表结构，进行一定程度的维度退化。作为更易处理的明细数据。
比如： 

ODS 地区 + ODS 省份=> DWD 省份地区
ODS 商品信息 + ODS 品牌 + ODS 商品一级分类 + ODS 商品二级分类 + ODS 商品 三级分类=>DWD 商品信息 

DWD 层表结构如下图所示 

![DWD 层表结构](1584888264329.png)

#### 建表语句

1）临时查询中，执行建表语句

```sql
CREATE TABLE `dwd_order_info_di` (
`id` string COMMENT '订单 id',
`total_amount` double COMMENT '订单总额',
`order_status` string COMMENT ' 1 未支付 2 已支付 3 已发货 4 已
收货 5 完成',
`user_id` string COMMENT '用户 id',
`payment_way` string COMMENT '付款方式',
`out_trade_no` string COMMENT '订单流失号',
`province_id` string COMMENT '省市 id',
`create_time` string COMMENT '创建时间',
`operate_time` string COMMENT '修改时间'
)
COMMENT '订单表'
PARTITIONED BY (ds string);
CREATE TABLE `dwd_order_detail_di` (
`id` string COMMENT '明细 id',
`order_id` string COMMENT '订单 id',
`user_id` string COMMENT '用户 id',
`sku_id` string COMMENT '商品 id',
`sku_name` string COMMENT '商品名称',
`order_price` string COMMENT '购买价格',
`sku_num` string COMMENT '购物数量',
`province_id` string COMMENT '省市 id',
`create_time` string COMMENT '创建时间'
)
COMMENT '订单明细'
PARTITIONED BY (ds string);
CREATE TABLE `dim_sku_info_df` (
`id` string COMMENT '商品 id',
`spu_id` string COMMENT 'spuid',
`price` double COMMENT '商品价格',
`sku_name` string COMMENT '商品名称',
`sku_desc` string COMMENT '商品描述',
`weight` double COMMENT '重量',
`tm_id` string COMMENT '品牌 id',
`tm_name` string COMMENT '品牌名称',
`category3_id` string COMMENT '三级分类 id',
`category2_id` string COMMENT '二级分类 id',
`category1_id` string COMMENT '一级分类 id',
`category3_name` string COMMENT '三级分类名称',
`category2_name` string COMMENT '二级分类名称',
`category1_name` string COMMENT '一级分类名称',
`create_time` string COMMENT '创建时间'
)
COMMENT '商品表信息'
PARTITIONED BY (ds string);
CREATE TABLE `dim_user_info_df` (
`id` string COMMENT 'id',
`name` string COMMENT '用户名称',
`birthday` string COMMENT '生日',
`gender` string COMMENT '性别',
`email` string COMMENT '邮箱',
`user_level` string COMMENT '等级',
`create_time` string COMMENT '注册时间'
)
COMMENT '用户信息表'
PARTITIONED BY (ds string);
CREATE TABLE `dim_base_province_df` (
`id` string COMMENT 'id',
`province_name` string COMMENT '省市名称',
`region_id` string COMMENT '地区 id',
`region_name` string COMMENT '地区名称'
)
COMMENT '地区省市表'
PARTITIONED BY (ds string);
```

2）在表管理里面查看创建的表 

![查看创建的表](1584890633172.png)

3）数据开发->dwd->导入表 

![导入表](1584890690877.png)

4）选择刚创建的所有表 

![选择刚创建的所有表 ](1584890716157.png)

#### 手动将数据导入 DWD 层 

1）在临时查询中执行 

```
Insert overwrite table dwd_order_info_di partition(ds)
select id,
total_amount,
order_status,
user_id,
payment_way,
out_trade_no,
province_id,
create_time,
operate_time,
ds
from ods_order_info_di
where ds='${bizdate}' and id is not null;
insert overwrite table dwd_order_detail_di partition(ds)
select od.id,
order_id,
oi.user_id,
sku_id,
sku_name,
order_price,
sku_num,
oi.province_id,
od.create_time,
od.ds
from ods_order_detail_di od join ods_order_info_di oi
on od.order_id = oi.id and oi.ds = '${bizdate}'
and od.ds = '${bizdate}' and od.id is not null;
insert overwrite table dim_sku_info_df partition(ds)
select
sku.id,
sku.spu_id,
sku.price,
sku.sku_name,
sku.sku_desc,
sku.weight,
sku.tm_id,
tm.tm_name,
sku.category3_id,
c2.id category2_id ,
c1.id category1_id,
c3.name category3_name,
c2.name category2_name,
c1.name category1_name,
sku.create_time,
sku.ds
from
(
select *
from ods_sku_info_df
where ds='${bizdate}' and id is not null
) sku
join ods_base_category3_df c3 on sku.category3_id = c3.id and
c3.ds = '${bizdate}'
join ods_base_category2_df c2 on c3.category2_id = c2.id and
c2.ds = '${bizdate}'
join ods_base_category1_df c1 on c2.category1_id = c1.id and
c1.ds = '${bizdate}'
join ods_base_trademark_df tm on tm.tm_id = sku.tm_id and tm.ds
= '${bizdate}';
insert overwrite table dim_user_info_df partition(ds)
select id,
name ,
birthday,
gender,
email,
user_level,
create_time,
ds from ods_user_info_df
where ds='${bizdate}' and id is not null;
insert overwrite table dim_base_province_df PARTITION (ds)
select
p.id,
p.name,
p.region_id,
r.region_name,
p.ds
from ods_base_province_df p join ods_base_region_df r
on p.region_id = r.id and p.ds='${bizdate}' and r.ds =
'${bizdate}';
```

2）在临时查询中查询结果 

```sql
select * from dwd_order_info_di where ds='20191008';
select * from dwd_order_detail_di where ds='20191008';
select * from dim_sku_info_df where ds='20191008';
select * from dim_user_info_df where ds='20191008';
select * from dim_base_province_df where ds='20191008';
```

### 数据导入脚本 

1）编写 dwd_order_info_di 表脚本

（1）在流程中加入一个数据开发脚本 

![加入一个数据开发脚本](1584891252868.png)

```sql
Insert overwrite table dwd_order_info_di partition(ds)
Select id,
total_amount,
order_status,
user_id,
payment_way,
out_trade_no,
province_id,
create_time,
operate_time,
ds
from ods_order_info_di
where ds='${bizdate}' and id is not null;
```

（2）配置参数
点击调度配置-> bizdate=${yyyymmdd} 

2）编写 dwd_order_detail_di 表脚本
（1）新建节点 dwd_order_detail_di_sql 

![新建节点](1584891363222.png)

![新建节点](1584891389484.png)

```sql
insert overwrite table dwd_order_detail_di partition(ds)
select od.id,
order_id,
oi.user_id,
sku_id,
sku_name,
order_price,
sku_num,
oi.province_id,
od.create_time,
od.ds
from ods_order_detail_di od join ods_order_info_di oi
on od.order_id = oi.id and oi.ds = '${bizdate}'
and od.ds = '${bizdate}' and od.id is not null;
```

(2）配置参数
点击调度配置-> bizdate=${yyyymmdd} 

3）编写 dim_sku_info_df 表脚本
（ 1）新建节点 dim_sku_info_df_sql 

![新建节点](1584891481964.png)

```sql
insert overwrite table dim_sku_info_df partition(ds)
select
sku.id,
sku.spu_id,
sku.price,
sku.sku_name,
sku.sku_desc,
sku.weight,
sku.tm_id,
tm.tm_name,
sku.category3_id,
c2.id category2_id ,
c1.id category1_id,
c3.name category3_name,
c2.name category2_name,
c1.name category1_name,
sku.create_time,
sku.ds
from
(
select *
from ods_sku_info_df
where ds='${bizdate}' and id is not null
) sku
join ods_base_category3_df c3 on sku.category3_id = c3.id and
c3.ds = '${bizdate}'
join ods_base_category2_df c2 on c3.category2_id = c2.id and
c2.ds = '${bizdate}'
join ods_base_category1_df c1 on c2.category1_id = c1.id and
c1.ds = '${bizdate}'
join ods_base_trademark_df tm on tm.tm_id = sku.tm_id and tm.ds
= '${bizdate}';
```

(2）配置参数
点击调度配置-> bizdate=${yyyymmdd} 

4）编写 dim_user_info_df 表脚本 

（ 1）新建节点 dim_user_info_df_sql 

![新建节点](1584891569954.png)

```sql
insert overwrite table dim_user_info_df partition(ds)
select id,
name ,
birthday,
gender,
email,
user_level,
create_time,
ds from ods_user_info_df
where ds='${bizdate}' and id is not null;
```

（ 2）配置参数
点击调度配置-> bizdate=${yyyymmdd}
5）编写 dim_base_province_df 表脚本
（ 1）新建节点 dim_base_province_df_sql 

![新建节点](1584891616692.png)

```sql
insert overwrite table dim_base_province_df PARTITION (ds)
select
p.id,
p.name,
p.region_id,
r.region_name,
p.ds
from ods_base_province_df p join ods_base_region_df r
on p.region_id = r.id and p.ds='${bizdate}' and r.ds =
'${bizdate}';
```

（ 2）配置参数
点击调度配置-> bizdate=${yyyymmdd} 

### DWS 层 

DWS 层主要指针对明细粒度的数据进行短周期的汇总。 DWS 公共汇总层是面向分析对
象的主题聚集建模。

**最终的分析目标为：最近一天某个类目、某个地区、某类人群购买商品的销售总额、购买力分布。**

因此，我们可以以最终交易成功的商品、买家、地区等角度对最近一天的数据进行组合，组合成为涵盖多个维度的事实宽表。 

#### 建表语句 

1）临时查询中，执行建表语句 

```sql
CREATE TABLE `dws_trade_detail_di` (
`user_id` string COMMENT '用户 id',
`sku_id` string COMMENT '商品 Id',
`user_gender` string COMMENT '用户性别',
`user_age` string COMMENT '用户年龄',
`user_level` string COMMENT '用户等级',
`sku_price` double COMMENT '商品当日价格',
`sku_name` string COMMENT '商品名称',
`sku_category3_id` string COMMENT '商品三级品类 id',
`sku_category2_id` string COMMENT '商品二级品类 id',
`sku_category1_id` string COMMENT '商品一级品类 id',
`sku_category3_name` string COMMENT '商品三级品类名称',
`sku_category2_name` string COMMENT '商品二级品类名称',
`sku_category1_name` string COMMENT '商品一级品类名称',
`spu_id` string COMMENT '商品 spu',
`tm_id` string COMMENT '品牌 id',
`tm_name` string COMMENT '品牌名称',
`province_id` string COMMENT '省市 id',
`province_name` string COMMENT '省市名称',
`region_id` string COMMENT '地区 id',
`region_name` string COMMENT '地区名称',
`sku_num` bigint COMMENT '购买个数',
`order_count` bigint COMMENT '当日下单单数',
`order_amount` double COMMENT '当日下单金额'
)
COMMENT '用户单日交易行为宽表'
PARTITIONED BY (ds string);
```

2）在表管理里面查看创建的表 

![查看创建的表](1584891839724.png)

3）数据开发->dws->导入表 

![导入表](1584891877673.png)

4）选择刚创建的所有表 

![选择刚创建的所有表](1584891915067.png)

### 手动将数据导入 DWS 层 

1）在临时查询中执行 

```sql
with tmp_trade AS  --方便sql一次执行多次使用
(
select
od.user_id,od.sku_id,od.province_id,
sum(sku_num) sku_num,
count(*) order_count,
sum(od.order_price*sku_num) order_amount
from dwd_order_detail_di od
where od.ds='${bizdate}'
group by od.user_id, od.sku_id, od.province_id
)

--多个指标Join多个表
insert OVERWRITE TABLE dws_trade_detail_di PARTITION
(ds='${bizdate}')
select
tmp_trade.user_id,
tmp_trade.sku_id,
u.gender,
months_between(to_char(to_date('${bizdate}','yyyymmdd'),'yy
yy-mm-dd'), u.birthday)/12 age,
u.user_level,
price,
sku_name,
category3_id,
category2_id,
category1_id,
category3_name,
category2_name,
category1_name,
spu_id,
tm_id,
tm_name,
p.id,
p.province_name,
p.region_id,
p.region_name,
tmp_trade.sku_num,
tmp_trade.order_count,
tmp_trade.order_amount
from tmp_trade
left join dim_user_info_df u on u.id = tmp_trade.user_id and
u.ds = '${bizdate}'
left join dim_sku_info_df s on tmp_trade.sku_id = s.id and s.ds
= '${bizdate}'
left join dim_base_province_df p on tmp_trade.province_id = p.id
and p.ds='${bizdate}';
```

2）查看结果

```sql
SELECT * from dws_trade_detail_di WHERE ds='20191008'
```

#### 数据导入脚本 

1）编写 dws_trade_detail_di 表脚本
（ 1）在流程中加入一个数据开发脚本 

![加入一个数据开发脚本](1584892575877.png)

（ 2）新建节点 dws_trade_detail_di_sql 

![新建节点](1584892611381.png)

```sql
with tmp_trade AS
(
select
od.user_id,od.sku_id,od.province_id,
sum(sku_num) sku_num,
count(*) order_count,
sum(od.order_price*sku_num) order_amount
from dwd_order_detail_di od
where od.ds='${bizdate}'
group by od.user_id, od.sku_id, od.province_id
)
insert OVERWRITE TABLE dws_trade_detail_di PARTITION
(ds='${bizdate}')
select
tmp_trade.user_id,
tmp_trade.sku_id,
u.gender,
months_between('${bizdate}', u.birthday)/12 age,
u.user_level,
price,
sku_name,
category3_id,
category2_id,
category1_id,
category3_name,
category2_name,
category1_name,
spu_id,
tm_id,
tm_name,
p.id,
p.province_name,
p.region_id,
p.region_name,
tmp_trade.sku_num,
tmp_trade.order_count,
tmp_trade.order_amount
from tmp_trade
left join dim_user_info_df u on u.id = tmp_trade.user_id and
u.ds = '${bizdate}'
left join dim_sku_info_df s on tmp_trade.sku_id = s.id and s.ds
= '${bizdate}'
left join dim_base_province_df p on tmp_trade.province_id = p.id
and p.ds='${bizdate}';
```

（ 2）配置参数
点击调度配置-> bizdate=${yyyymmdd} 

### ADS 层 

ADS 层主要指针对某一个特定的维度进行的汇总。
主要分析三个需求： 

用户各个年龄段统计、地区销售统计、热门商品排行
所以主要是针对年龄、地区、商品进行汇总统计，统计四个指标下单数、购买商品个数、
销售额、平均客单价。 



### 建表语句 

1）临时查询中，执行建表语句 

```sql
CREATE TABLE `ads_trade_age_d` (
`age` bigint COMMENT '年龄',
`sku_num` bigint COMMENT '购买商品个数',
`order_count` bigint COMMENT '订单个数',
`order_amount` double COMMENT '销售额',
`avg_amount` double COMMENT '平均客单价'
)
COMMENT '年龄销售统计'
PARTITIONED BY (ds string);
CREATE TABLE `ads_trade_province_d` (
`province` string COMMENT '省份 id',
`province_name` string COMMENT '省市名称',
`region_id` string COMMENT '地区 ID',
`region_name` string COMMENT '地区名称',
`sku_num` bigint COMMENT '购买商品个数',
`order_count` bigint COMMENT '订单个数',
`order_amount` double COMMENT '销售额',
`avg_amount` double COMMENT '平均客单价'
)
COMMENT '地区销售统计'
PARTITIONED BY (ds string);
CREATE TABLE `ads_trade_sku_d` (
`sku_id` string COMMENT '商品 id',
`sku_name` string COMMENT '商品名称',
`sku_num` bigint COMMENT '购买商品个数',
`category3_id` string COMMENT '三级分类 id',
`category2_id` string COMMENT '二级分类 id',
`category1_id` string COMMENT '一级分类 id',
`category3_name` string COMMENT '三级分类名称',
`category2_name` string COMMENT '二级分类名称',
`category1_name` string COMMENT '一级分类名称',
`order_count` bigint COMMENT '订单个数',
`order_amount` double COMMENT '销售额',
`avg_amount` double COMMENT '平均客单价'
)
COMMENT '商品销售统计'
PARTITIONED BY (ds string);
```

2）在表管理里面查看创建的表 

![查看创建的表](1584892810119.png)

3）数据开发->ads->导入表 

![导入表](1584892852910.png)

4）选择刚创建的所有表 

![选择刚创建的所有表](584892876453.png)

#### 手动将数据导入 ADS 层 

1）在临时查询中执行 

```sql
insert OVERWRITE table ads_trade_age_d PARTITION (ds =
'${bizdate}')
select
round(td.user_age) age,
sum(sku_num) sku_num,
sum(order_count) order_count,
sum(order_amount) order_amount,
round(avg(order_amount),2) avg_amount
from dws_trade_detail_di td
where ds = '${bizdate}'
group by round(td.user_age);
insert OVERWRITE table ads_trade_province_d PARTITION (ds =
'${bizdate}')
select
td.province_id,td.province_name,td.region_id,td.region_name,
sum(sku_num) sku_num,
sum(order_count) order_count,
sum(order_amount) order_amount,
round(avg(order_amount),2) avg_amount
from dws_trade_detail_di td
where ds='${bizdate}'
group by
td.province_id,td.province_name,td.region_id,td.region_name;
insert OVERWRITE table ads_trade_sku_d PARTITION (ds =
'${bizdate}')
select
td.sku_id,td.sku_name,
td.sku_category3_id,
td.sku_category2_id,
td.sku_category1_id,
td.sku_category3_name,
td.sku_category2_name,
td.sku_category1_name,
sum(sku_num) sku_num,
sum(order_count) order_count,
sum(order_amount) order_amount,
round(avg(order_amount),2) avg_amount
from dws_trade_detail_di td
where ds = '${bizdate}'
group by
td.sku_id, td.sku_name, td.sku_category3_id,
td.sku_category2_id, td.sku_category1_id,
td.sku_category3_name, td.sku_category2_name,
td.sku_category1_name;
```

2）在临时查询中查询结果 

```sql
select * from ads_trade_age_d WHERE ds='20191008';
select * from ads_trade_province_d WHERE ds='20191008';
select * from ads_trade_sku_d WHERE ds='20191008';
```

#### 数据导入脚本 

1）编写 ads_trade_age_d 表脚本
（ 1）在流程中加入一个数据开发脚本 

![加入一个数据开发脚本](1584893015966.png)

（ 2）新建节点 ads_trade_age_d_sql 

![新建节点](1584893058247.png)

```sql
insert OVERWRITE table ads_trade_age_d PARTITION (ds =
'${bizdate}')
select
round(td.user_age) age,
sum(sku_num) sku_num,
sum(order_count) order_count,
sum(order_amount) order_amount,
round(avg(order_amount),2) avg_amount
from dws_trade_detail_di td
where ds = '${bizdate}'
group by round(td.user_age)
```

（ 2）配置参数
点击调度配置-> bizdate=${yyyymmdd} 

2）编写 ads_trade_province_d 表脚本
（ 1）新建节点 ads_trade_province_d_sql 

![新建节点](1584893126473.png)

```sql
insert OVERWRITE table ads_trade_province_d PARTITION (ds =
'${bizdate}')
select
td.province_id,td.province_name,td.region_id,td.region_name,
sum(sku_num) sku_num,
sum(order_count) order_count,
sum(order_amount) order_amount,
round(avg(order_amount),2) avg_amount
from dws_trade_detail_di td
where ds='${bizdate}'
group by
td.province_id,td.province_name,td.region_id,td.region_name;
```

（ 2）配置参数
点击调度配置-> bizdate=${yyyymmdd} 

3）编写 ads_trade_sku_d 表脚本
（ 1）新建节点 ads_trade_sku_d_sql 

![新建节点](1584893193490.png)

```sql
insert OVERWRITE table ads_trade_sku_d PARTITION (ds =
'${bizdate}')
select
td.sku_id,td.sku_name,
td.sku_category3_id,
td.sku_category2_id,
td.sku_category1_id,
td.sku_category3_name,
td.sku_category2_name,
td.sku_category1_name,
sum(sku_num) sku_num,
sum(order_count) order_count,
sum(order_amount) order_amount,
round(avg(order_amount),2) avg_amount
from dws_trade_detail_di td
where ds = '${bizdate}'
group by
td.sku_id, td.sku_name, td.sku_category3_id,
td.sku_category2_id, td.sku_category1_id,
td.sku_category3_name, td.sku_category2_name,
td.sku_category1_name;
```

（ 2）配置参数
点击调度配置-> bizdate=${yyyymmdd} 

### 作业调度 

1）整个业务部分的数仓任务应该包括如下图： 

![整个业务部分的数仓任务](1584893262155.png)

2）在原理 ODS 层的基础上继续增加 DWD 层、 DWS 层业务、 ADS 层业务。 

![各层流程图](1584893324196.png)

![完整的开发环境界面如下图](1584893428690.png)

3）在公共表中预览执行结果 

![公共表中预览执行结果](1584893490902.png)

### 数据导出与作业调度 

将 MaxCompute 中的计算完的结果， 需要导入到 RDS 数据库中， 用于后续的可视化。 

#### 创建结果数据库 

1）在 RDS 服务器中， 新建一个 gmall_adb 数据库，用来保存之后从 MaxCompute 中的结果
数据 

![创建数据库表](1584893610335.png)

2）建 4 张和 ADS 层结果一样的表 

```sql
CREATE TABLE `uv_source_d` (
`source` varchar(20) NOT NULL COMMENT '渠道',
`ct` bigint(20) DEFAULT NULL COMMENT '个数',
`ds` varchar(8) NOT NULL COMMENT '日期',
PRIMARY KEY (`source`,`ds`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='渠道日活';
CREATE TABLE `trade_age_d` (
`age` BIGINT(20) NOT NULL COMMENT '年龄',
`sku_num` BIGINT(20) DEFAULT NULL COMMENT '购买商品个数',
`order_count` BIGINT(20) DEFAULT NULL COMMENT '订单个数',
`order_amount` DECIMAL(16,2) DEFAULT NULL COMMENT '销售额',
`avg_amount` DECIMAL(10,2) DEFAULT NULL COMMENT '平均客单价',
`ds` VARCHAR(20) NOT NULL COMMENT '日期',
    PRIMARY KEY (`age`,`ds`)
) ENGINE=INNODB DEFAULT CHARSET=utf8 COMMENT='年龄销售统计';
CREATE TABLE trade_province_d
(
province_id VARCHAR(20) NOT NULL COMMENT '省市 id',
province_name VARCHAR(20) COMMENT '省市名称',
region_id VARCHAR(20) COMMENT '地区 ID',
region_name VARCHAR(20) COMMENT '地区名称',
sku_num BIGINT COMMENT '购买商品个数',
order_count BIGINT COMMENT '订单个数',
order_amount DECIMAL(16,2) COMMENT '销售额',
avg_amount DECIMAL(10,2) COMMENT '平均客单价',
`ds` VARCHAR(20) NOT NULL COMMENT '日期' ,
PRIMARY KEY (`province_id`,`ds`)
)ENGINE=INNODB DEFAULT CHARSET=utf8
COMMENT '地区销售统计';
CREATE TABLE `trade_sku` (
`sku_id` VARCHAR(20) NOT NULL COMMENT '商品 id',
`sku_name` VARCHAR(200) DEFAULT NULL COMMENT '商品名称',
`sku_num` BIGINT(20) DEFAULT NULL COMMENT '购买商品个数',
`category3_id` VARCHAR(20) COMMENT '三级分类 id',
`category2_id` VARCHAR(20) COMMENT '二级分类 id',
`category1_id` VARCHAR(20) COMMENT '一级分类 id',
`category3_name` VARCHAR(20) COMMENT '三级分类名称',
`category2_name` VARCHAR(20) COMMENT '二级分类名称',
`category1_name` VARCHAR(20) COMMENT '一级分类名称',
`order_count` BIGINT(20) DEFAULT NULL COMMENT '订单个数',
`order_amount` DECIMAL(16,2) DEFAULT NULL COMMENT '销售额',
`avg_amount` DECIMAL(10,2) DEFAULT NULL COMMENT '平均客单价',
`ds` VARCHAR(20) NOT NULL COMMENT '日期',
PRIMARY KEY (`sku_id`,`ds`)
) ENGINE=INNODB DEFAULT CHARSET=utf8 COMMENT='商品销售统计';
```

3）查看创建好的 4 张表 

![查看创建好的4张表](1584893708266.png)

#### 创建商品销售数据同步节点 

将 ADS 层数据导出到 MaxCompute。
1）数据开发->数据集成->新建数据集成节点->数据同步 

![数据同步](1584893778309.png)

2）新建节点 ads_trade_sku_exp

![新建节点](1584893815043.png)

3）建立连接 RDS 的数据源（由于和之前导入时的 rds 不是一个 databaseName 所以要新建一
个） 

![建立连接 RDS 的数据源](1584893851471.png)

4）设定要导出的数据库 

![设定要导出的数据库](1584893888320.png)

5）加入数据源后刷新菜单可以看到新的数据源 

![查看新的数据源 ](1584893919037.png)

6）映射部分
注意 此处需注意如果想把分区字段导入到目标表中，需要手动添加该分区字段 

![映射字段](1584893959779.png)

7）配置参数
点击调度配置-> bizdate=${yyyymmdd} 

1）数据开发->数据集成->新建数据集成节点->数据同步
2） 创建节点 ads_trade_age_exp 

![创建节点](1584894000037.png)

3） 配置年龄统计表导出详情 

![配置年龄统计表导出详情](1584894030944.png)

4）配置参数
点击调度配置-> bizdate=${yyyymmdd} 

#### 创建省市统计表数据同步节点 

1）数据开发->数据集成->新建数据集成节点->数据同步 

2） 创建节点 ads_trade_province_exp 

![创建节点](1584894096874.png)

3）配置省市统计表导出详情 

![配置省市统计表导出详情 ](1584894130606.png)

4）配置参数
点击调度配置-> bizdate=${yyyymmdd} 

#### 创建渠道统计表数据同步节点 

1）数据开发->数据集成->新建数据集成节点->数据同步 

2） 创建节点 ads_uv_source_exp 

![创建节点](1584894184531.png)

3）渠道日活统计 

![渠道日活统计](1584894213762.png)

4）配置参数
点击调度配置-> bizdate=${yyyymmdd} 

### 作业调度 

1）把 ADS 层的数据流指向对应的数据导出节点。 

![指向对应的数据导出节点](1584894302665.png)

2）执行调度策略 

![点击左上角执行调度策略](1584894343806.png)

3）流程跑完之后可以在 MySql RDS 中查看最后统计结果表中的数据 ，如下图 

![查看最后统计结果表中的数据](1584894384988.png)