### 一、字段类型及选型

##### 1.数字类型

- 数字类型对照表
    
|<div style="width: 80pt">类型名</div> | <div style="width: 80pt">所占字节长度</div> | <div style="width: 110pt">无符号范围(unsigned)</div> | <div style="width: 110pt">有符号范围</div> | 说明|
|---|---|---|---|---|
**tinyint** | 1 (8 bit) | 0 ~ 255 | -128 ~ 127 | 布尔值、类型、状态等字段推荐
**smallint** | 2 | 0 ~ 65535 | -32758 ~ 32767 | &nbsp;
**mediumint** | 3 | 0 ~ 2^24-1 | -2^23 ~ 2^23-1 | &nbsp;
**int** | 4 | 0 ~ 2^32-1 | -2^31 ~ 2^31-1 | 配置表推荐主键
**bigint** | 8 | 0 ~ 2^64-1 | -2^63 ~ 2^63-1 | 推荐主键
~~**float**~~ | 4 | &nbsp; | &nbsp; | 禁止使用
~~**double**~~ | 8 | &nbsp; | &nbsp; | 禁止使用
**decimal(M, D)** | M+2 | &nbsp; | &nbsp; | 金钱相关推荐类型，M表示数字总个数，D表示小数位个数
    
- 注意事项
    - int(M)
        
    ```
    1. M表示显示长度，无实际意义，定义时不建议使用，默认即可
    
    create table t1(
        a int ...    推荐
        b int(8) ... 不推荐
    );
    ```
    
    - 主键与自增属性（auto_increment）
    
    ```
    1. 自增字段的插入可以使用以下4种方式
        
        mysql> show create table t1\G
        *************************** 1. row ***************************
               Table: t1
        Create Table: CREATE TABLE `t1` (
          `id` int(11) NOT NULL AUTO_INCREMENT,
          `a` int(11) DEFAULT NULL,
          PRIMARY KEY (`id`),
          UNIQUE KEY `a` (`a`)
        ) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8mb4

        (1) 不指定自增字段
            inert into t1(a) values(3)
        (2) 对自增字段插入NULL值或0值
            insert into t1(id, a) values(NULL, 4)
            insert into t1(id, a) values(0, 5)
        (3) 指定一个值插入（不推荐）
            insert into t1(id, a) values(999, 6)
            
    2. 自增属性对于插入失败依旧累加
    
    mysql> create table t1(id int primary key auto_increment, a int unique);
    Query OK, 0 rows affected (0.03 sec)
    
    mysql> insert into t1 select 0, 0;
    Query OK, 1 row affected (0.00 sec)
    Records: 1  Duplicates: 0  Warnings: 0
    
    mysql> show create table t1\G
    *************************** 1. row ***************************
           Table: t1
    Create Table: CREATE TABLE `t1` (
      `id` int(11) NOT NULL AUTO_INCREMENT,
      `a` int(11) DEFAULT NULL,
      PRIMARY KEY (`id`),
      UNIQUE KEY `a` (`a`)
    ) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4
    1 row in set (0.00 sec)
    
    mysql> insert into t1 select 0, 0;
    ERROR 1062 (23000): Duplicate entry '0' for key 'a'
    mysql> show create table t1\G
    *************************** 1. row ***************************
           Table: t1
    Create Table: CREATE TABLE `t1` (
      `id` int(11) NOT NULL AUTO_INCREMENT,
      `a` int(11) DEFAULT NULL,
      PRIMARY KEY (`id`),
      UNIQUE KEY `a` (`a`)
    ) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8mb4
    1 row in set (0.00 sec)

    mysql> insert into t1 select 0, 2;
    Query OK, 1 row affected (0.00 sec)
    Records: 1  Duplicates: 0  Warnings: 0
    
    mysql> select * from t1;
    +----+------+
    | id | a    |
    +----+------+
    |  1 |    0 |
    |  3 |    2 |
    +----+------+
    2 rows in set (0.00 sec)

    
    2. MySQL 5.7及以前的版本，自增值并不持久化，在MySQL重启后会重新计算（auto_increment = max(PK)+1）
    3. 自增字段必须有索引，否则报错
    4. 建议使用自增字段作为主键，MySQL是索引组织表，二级索引会存储主键，使用uuid为会造成一定程度的空间浪费，除非必要，否则不建议使用uuid作为主键
    ```
        
    - zerofill属性（0填充）
        
    ```
    # 通常禁止使用
    1. 使用该属性时MySQL会自动加上 unsigned 属性
    2. int(M) zerofill，数字长度不足M时，select出来会在数字前用“0”填充
    ```
    
    - 无符号（unsigned）
    
    ```
    # 通常禁止使用
    1. 参考“数字类型对照表”，存储范围可提高一倍，但是不会有数量级的改变，通常不会使用
    2. unsigned属性的字段做减法操作时，如果出现负值会直接报错
    3. 如果已经使用unsigned属性，在不允许改变表结构的前提，仍然需要做减法且有负值，可以修改sql_mode
        set session sql_mode = 'no_unsigned_substraction'; 
    ```
    
    - decimal(M, D)
    
    ```
    1. decimal中 [M≥D], M表示数字总个数，D表示小数位个数
        decimal(3, 2) 最大可以存储 9.99
    2. 超范围插入：（MySQL5.7严格模式下依然存在）
        小数超出，四舍五入；整数超出，取限定的最大值
        decimal(3, 1): 
            19.05 -> 19.1
            19.03 -> 19.0
            191.2 -> 99.9
    ```
        
    - 布尔值
    
    ```
    1. 布尔值会自动转化为 tinyint
        insert into t_name values (true),(false); 等价于 insert into t_name values (1),(0);
    ```
    
    - IP推荐使用int存储
    
    ```
    1. 使用mysql内置函数转换
        IP转数字: inet_ntoa()
        数字转IP：inet_aton()
    2. 程序端转换
    ```

##### 2.字符类型

- 字符类型对照表
    
|<div style="width: 80pt">类型名</div> | <div style="width: 60pt">注释</div> | <div style="width: 60pt">单位</div> | <div style="width: 80pt">是否使用字符集</div> | 最大取值 | 说明|
|---|---|---|---|---|---|
|**char(N)** | 定长字符 | character | 是 | N≤255 | &nbsp;|
|**varchar(N)** | 变长字符 | character | 是 | 见 varchar存储限制对照表 | &nbsp;|
|~~**binary(N)**~~ | 定长二进制字节 | byte | 否 | N≤255 | 不推荐|
|~~**varbinary(N)**~~ | 变长二进制字节 | byte | 否 | 略 | 不推荐|
|~~**tinytext**~~ | 大对象 | byte | 是 | 256B | 一般不用|
|**text** | 大对象 | byte | 是 | 16K | &nbsp;|
|**mediumtext** | 大对象 | byte | 是 | 16M | &nbsp;|
|**longtext** | 大对象 | byte | 是 | 4G | &nbsp;|
|~~**tinyblob**~~ | 二进制大对象 | byte | 否 | 256B | 一般不用|
|**blob** | 二进制大对象 | byte | 否 | 16K | &nbsp;|
|**mediumblob** | 二进制大对象 | byte | 否 | 16M | &nbsp;|
|**longblob** | 二进制大对象 | byte | 否 | 4G | &nbsp;|
|**json** | JSON串 | byte | &nbsp; | 与longblob或longtext列的存储要求大致相同 | 5.7版本不推荐使用，推荐使用varchar|
    
- 注意事项
    - char(N)
        - utf8字符集下存储对照表
        
        |<div style="width: 80pt">值</div>|char(4)|<div style="width: 50pt">存储需求</div>|varchar(4)|存储需求|
        |---|---|---|---|---|
        |'' | '&nbsp;&nbsp;&nbsp;&nbsp;' | 12 bytes | '' | 0+1 bytes（1用于记录字符长度=0）|
        |'慧眼' | '慧眼&nbsp;&nbsp;' | 12 bytes | '慧眼' | 6+1 bytes|
        |'雾里看花' | '雾里看花' | 12 bytes | '雾里看花' | 12+1 bytes|
        |'你知哪句是真哪句是假' | '你知哪句'（非严格模式）| 12 bytes | '你知哪句'（非严格模式）| 12+1 bytes|
        
        ```
        1. 超过规定的最大字符长度， 非严格模式下，会自动截断；严格模式下，直接报错
        ```
        
    - varchar(N)
        - 存储限制对照表
        
        |<div style="width: 80pt">字符集</div>|单个字符所占最大字节数|<div style="width: 100pt">N的最大取值（表中仅此一列）</div>|计算方法|
        ---|---|---|---
        |latin1 | 1 | 65532 | 65535-2-1|
        |gbk | 2 | 32766 | (65535-2-1)/2|
        |utf8 | 3 | 21844 | (65535-2-1)/3|
        |utf8mb4 | 4 | 16383 | (65535-2-1)/4|
        
        ```
        1. “65535-2-1”中 -1 的含义：字段如果有 "DEFAULT NULL" 会占用一个字节
        2. “65535-2-1”中 -2 的含义：1-2个字节存储字符数（255个字符以内1个字节存储，以上2个字节存储）
        ```
        
        - 在存储长度<=20的字符串时，varchar(20)和varchar(200)的实际存储大小相同，但是在内存使用上有区别，varchar(200)会浪费内存，尤其是涉及排序和临时表
    - BLOB & TEXT
        - 在BLOB和TEXT类型的列上创建索引时，必须指定索引前缀的长度（前缀索引）
        - 在BLOB和TEXT类型的列上创建索引时，必须指定索引前缀的长度
            - MySQL最小的存储单元是“页”：默认16K，为了高效存储这样的列，一般将这种列的数据值存放在溢出页中
    
    - JSON
        - 官方文档：https://dev.mysql.com/doc/refman/5.7/en/storage-requirements.html#data-types-storage-reqs-json
        - MySQL 5.7开始原生支持JSON类型，与MongoDB BSON 相比，存储空间严重浪费，不推荐使用
        - 如果必要，推荐使用varchar存储JSON串，相关json方法仍然可以使用
        

##### 3.日期时间类型

- 日期时间类型对照表
    
| <div style="width: 80pt">类型名</div> | <div style="width: 60pt">注释</div> | <div style="width: 60pt">字节数</div> | 取值范围 | 零值 | 说明 |
|---|---|---|---|---|---|
|~~**year**~~ | 年 | 1 | 1901-2155 | 0000 | 不推荐使用|
|**date** | 日期 | 3 | 1000-01-01~9999-12-31 | 0000:00:00 | &nbsp;|
|~~**time**~~ | 时间 | 3 or 3+ (MySQL5.6.4) | -838:59:59~838:59:59 | 00:00:00 | 不推荐使用|
|**datetime** | 日期时间 | 8 or 5+ (MySQL5.6.4)| 1000-01-01 00:00:00~9999-12-31 23:59:59 | 0000-00-00 00:00:00 | 推荐|
|~~**timestamp**~~ | 日期时间 | 4 or 4+ (MySQL5.6.4) | 19700101000001~20380119031407 | 00000000000000 | 不推荐使用|
    
- 注意事项
    - datetime
        - MySQL5.5 及以前的版本日期类型不能精确到微秒级别，任何微秒数值都会被截断（5.7测试是四舍五入）
        - MySQL5.6 以后提供 datetime(6)，可存储到微秒
        ```
        mysql> show create table t1\G
        *************************** 1. row ***************************
               Table: t1
        Create Table: CREATE TABLE `t1` (
          `a` datetime DEFAULT NULL
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
        1 row in set (0.00 sec)

        mysql> insert into t1(a) values('2020-10-17 23:59:59.500000');
        Query OK, 1 row affected (0.00 sec)
        
        mysql> insert into t1(a) values('2020-10-17 23:59:59.400000');
        Query OK, 1 row affected (0.01 sec)
        
        mysql> select * from t1;
        +---------------------+
        | a                   |
        +---------------------+
        | 2020-10-18 00:00:00 |
        | 2020-10-17 23:59:59 |
        +---------------------+
        2 rows in set (0.00 sec)
        ```
    - timestamp
        - 中国时区上限是 "2038-01-19 11:14:07"  MySQL5.6.4以后版本，存储空间与datetime差距不大，不推荐使用
    

### 二、字段属性

##### 1.not null
    
- 对于小varchar， char通常建议加上 not null default ''

- 不允许为空，设置 default null 时需要占用额外的1个字节的存储空间
    
##### 2. auto_increment
    
- 推荐自增id作为主键（bigint） 

- innodb引擎下必须创建在有索引的字段上

- MySQL5.7及以前，auto_increment值不做持久化存储，因此每次重启MySQL，都要重新计算下一个自增值

##### 3. unsigned
    
- 禁止使用

- 无符号整型，不允许存储负值，正数存储范围增加但无数量级的改变

- 设置 unsigned 在减法运算中如果出现负数会报错
    
##### 4.zerofill
    
- 禁止使用

- 自动会加上 unsigned 属性
    
##### 5.DEFAULT CURRENT_TIMESTAMP
    
- 可以使用

- 等价于 default now()，可加在 datetime 类型后，当新创建行数据时如果未指定会自动插入当前系统时间
    
##### 6.ON UPDATE CURRENT_TIMESTAMP

- 可以使用

- 等价于 on update now()，可加在 datetime 类型后，当创建或更新行数据时如果未指定会自动插入当前系统时间

##### 7. binary

- 当字符型字段需要区分大小写时推荐使用 binary

- 禁止直接更改字段的字符集和校对规则

```
mysql> create table t3(a varchar(50) binary);
Query OK, 0 rows affected (0.11 sec)

mysql> show create table t3\G\
*************************** 1. row ***************************
       Table: t3
Create Table: CREATE TABLE `t3` (
  `a` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_bin DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
1 row in set (0.00 sec)

```


### 三、在线表结构变更（Online DDL）


- 官方文档：https://dev.mysql.com/doc/refman/5.7/en/innodb-online-ddl-operations.html

- **旧表新加字段，需要允许为NULL（避免全表数据更新 ，长期持锁导致阻塞）**
    
- 以下二级索引变更，可直接在原表操作（直接alter）
    
| 操作 | 是否原地更新 | 是否重建表 |变更过程中是否允许DML | 是否仅修改元数据 |
|---|---|---|---|---|
|创建或添加二级索引 | 是 | 否 | 是 | 否 |
|删除索引 | 是 | 否 | 是 | 是 |
|重命名索引 | 是 | 否 | 是 | 是 |

- 主键变更均会重建表，如有必要推荐使用 pt-osc 工具

- 字段变更

|操作 | 是否原地更新 | 是否重建表 |变更过程中是否允许DML | 是否仅修改元数据 | 备注|
|---|---|---|---|---|---|
|添加字段|是|**是**|是|否|pt-osc|
|删除字段|是|**是**|是|否|pt-osc|
|重命名字段|是|**否**|是|是|alter|
|调整字段顺序|是|**是**|是|否|pt-osc|
|添加/删除字段默认值|是|**否**|是|是|alter|
|更改字段类型|**否**|**是**|否|否|pt-osc|
|调高varchar上限|是|**否**|是|是|alter|
|更改自增值|是|**否**|是|否|alter|
|字段设置NULL/NOT NULL属性|是|**是**|是|否|pt-osc|

- 生成列操作（Generated Column）

|操作 | 是否原地更新 | 是否重建表 |变更过程中是否允许DML | 是否仅修改元数据 | 备注|
|---|---|---|---|---|---|
|添加/修改字段顺序 STORAED 列|**否**|是|**否**|否|绝对禁止直接alter|
|删除 STORAED 列|是|是|**是**|否|pt-osc|
|添加/删除 VIRTUAL 列|是|**否**|是|是|alter|
|修改字段顺序 VIRTUAL 列|**否**|**是**|否|否|绝对禁止直接alter|


### 四、MySQL特点及使用约束及建议

##### 1.MySQL特点

- MySQL是单进程多线程架构
- 每个SQL同时只能用到一个逻辑CPU线程
- 无执行计划缓存，每条SQL都是硬解析（轻量级，通常不会造成瓶颈）
- query cache的更新需要持有 全局mutex，数据有任何更新都需要等待该mutex，效率低，且整个表的query cache也会失效，因此，强烈建议关闭query cache
- 没有thread pool时，如果有瞬间大量连接请求，性能会急剧下降

##### 2.使用约束及建议

- 引擎选择：InnoDB
- 尽量避免在DB做运算
- 禁止使用非decimal的浮点类型
- 禁用自定义函数
- 尽量避免使用外键、分区表
- 慎用 Generated Column
- 尽量避免多个大表关联查询，单表做适当的冗余

### 五、索引规范

- 索引名一律使用小写字母加连字符（如：idx_uid）
- 主键索引使用默认，推荐自增id做主键（int/bigint）
- 唯一索引：uk_字段名_字段名(字段, ...)
- 普通二级索引：idx_字段名_字段名(字段, ...)
- 对于大varchar（50以上），按需求考虑使用前缀索引或其它方法
- ORDER BY，GROUP BY，DISTINCT的字段需要添加在索引的后面
- 合理创建联合索引，避免冗余
    ```
    如：存在索引 idx_a_b_c(a, b, c)
    则：不应该再创建 idx_a, idx_a_b
    ```
- 创建联合索引，应该尽可能将选择性高的字段放在前导列上
- 查询时尽可能使用覆盖索引扫描，避免回表

### 六、SQL编写规范

##### 约定的概念

- 常数：不从表里获取直接指定的值
```
# 下面的 1 为常数
select 1 as a from t where user_id = ''
```
- 常数表：空行或只返回一行数据的表
```
select col1 from t where pk = xxx  # 主键等值查询
select col1 from t where uk = xxx  # 唯一键等值查询
```
- 字典表/配置表：数据量很小且很少变动的表，如：等级信息表
- 小表：本身数据量不多或通过索引筛选出相对数据量较少的表
- 大表：本身数据量很大且通过索引（或无法使用索引）筛选出的数据量依然很大的表
- 记录表/日志表：只INSERT的表，通常数据量会非常大

##### 1.相同字段的条件过滤，禁止使用 or，推荐使用 in

```
(1) 禁止
where a = 100 or a = 200
(2) 推荐
where a in (100, 200)
```

##### 2.不同字段的条件过滤应尽可能避免使用 or，可能导致无法使用索引，考虑改写成union或其它方法

- 5.7开启 index_merge 可以不用改写

```
(1) 原始SQL
mysql> desc select ugc_id, user_id, order_id from t1 where user_id = 'xxx' or order_id = 'xxx';
+----+-------------+-------+------------+------+---------------------+------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys       | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+------+---------------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | t1   | NULL       | ALL  | idx_orderid,idx_uid | NULL | NULL    | NULL | 5701 |    19.00 | Using where |
+----+-------------+-------+------------+------+---------------------+------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.02 sec)

(2) 改写union
mysql> desc select ugc_id, user_id, order_id from t1 where user_id = 'xxx'
    -> union
    -> select ugc_id, user_id, order_id from t1 where order_id = 'xxx';
+----+--------------+------------+------------+------+---------------+-------------+---------+-------+------+----------+-----------------+
| id | select_type  | table      | partitions | type | possible_keys | key         | key_len | ref   | rows | filtered | Extra           |
+----+--------------+------------+------------+------+---------------+-------------+---------+-------+------+----------+-----------------+
|  1 | PRIMARY      | t1         | NULL       | ref  | idx_uid       | idx_uid     | 203     | const |   17 |   100.00 | NULL            |
|  2 | UNION        | t1         | NULL       | ref  | idx_orderid   | idx_orderid | 203     | const |    1 |   100.00 | NULL            |
| NULL | UNION RESULT | <union1,2> | NULL       | ALL  | NULL          | NULL        | NULL    | NULL  | NULL |     NULL | Using temporary |
+----+--------------+------------+------------+------+---------------+-------------+---------+-------+------+----------+-----------------+
3 rows in set, 1 warning (0.01 sec)

(3) 5.7开启index_merge
mysql> set session optimizer_switch='index_merge=on';
Query OK, 0 rows affected (0.01 sec)

mysql> desc select ugc_id, user_id, order_id from t1 where user_id = 'xxx' or order_id = 'xxx';
+----+-------------+-------+------------+-------------+---------------------+---------------------+---------+------+------+----------+----------------------------------------------------+
| id | select_type | table | partitions | type        | possible_keys       | key                 | key_len | ref  | rows | filtered | Extra                                              |
+----+-------------+-------+------------+-------------+---------------------+---------------------+---------+------+------+----------+----------------------------------------------------+
|  1 | SIMPLE      | t1    | NULL       | index_merge | idx_orderid,idx_uid | idx_uid,idx_orderid | 203,203 | NULL |   18 |   100.00 | Using sort_union(idx_uid,idx_orderid); Using where |
+----+-------------+-------+------------+-------------+---------------------+---------------------+---------+------+------+----------+----------------------------------------------------+
1 row in set, 1 warning (0.01 sec)
```

##### 3.尽量避免使用不等值查询（"!=" 或 "<>"）

##### 4.常数表优先，字典表或小表其次，大表最后（小表驱动）

```
(1) SQL的执行计划是基于cost的 除非强制使用hint否则无法控制
(2) 表的连接条件最好是查询结果集最少的为驱动表，后续表要有良好的索引
```

##### 5.尽量避免在执行计划的extra中出现 "using temporary" 和 "using filesort" 关键字

##### 6.尽量避免使用 "is null" 或 "is not null" 的判断

##### 7.模糊匹配尽量避免使用前置百分号

- 虽然MySQL支持全文索引，但是使用较少，可能出现问题，不推荐使用
- 全文搜索推荐使用 ElasticSearch 

##### 8.where判断中的字段禁止做计算，禁止函数处理

```
(1.1) 函数处理，无法使用索引，最好的情况也只能使用全索引扫描
mysql> desc select ugc_id, content, create_time from t1 where date(create_time) = '2020-10-04';
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | t1    | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 5701 |   100.00 | Using where |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.01 sec)

(1.2) 去掉函数处理，成功使用索引
mysql> desc select ugc_id, content, create_time from t1 where create_time >= '2020-10-04 00:00:00' and create_time < '2020-10-05 00:00:00';
+----+-------------+-------+------------+-------+---------------+-----------+---------+------+------+----------+-----------------------+
| id | select_type | table | partitions | type  | possible_keys | key       | key_len | ref  | rows | filtered | Extra                 |
+----+-------------+-------+------------+-------+---------------+-----------+---------+------+------+----------+-----------------------+
|  1 | SIMPLE      | t1    | NULL       | range | idx_ctime     | idx_ctime | 6       | NULL |    1 |   100.00 | Using index condition |
+----+-------------+-------+------------+-------+---------------+-----------+---------+------+------+----------+-----------------------+
1 row in set, 1 warning (0.01 sec)

(2.1) 字段计算，无法使用索引
mysql> desc select product_id, price from t2 where unique_id * 2 > 100000 limit 10;
+----+-------------+---------+------------+------+---------------+------+---------+------+-------+----------+-------------+
| id | select_type | table   | partitions | type | possible_keys | key  | key_len | ref  | rows  | filtered | Extra       |
+----+-------------+---------+------------+------+---------------+------+---------+------+-------+----------+-------------+
|  1 | SIMPLE      | t2      | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 13244 |   100.00 | Using where |
+----+-------------+---------+------------+------+---------------+------+---------+------+-------+----------+-------------+
1 row in set, 1 warning (0.01 sec)

(2.2) 去掉计算
mysql> desc select product_id, price from t2 where unique_id > 50000 limit 10;
+----+-------------+---------+------------+-------+---------------+--------------+---------+------+------+----------+-----------------------+
| id | select_type | table   | partitions | type  | possible_keys | key          | key_len | ref  | rows | filtered | Extra                 |
+----+-------------+---------+------------+-------+---------------+--------------+---------+------+------+----------+-----------------------+
|  1 | SIMPLE      | t2      | NULL       | range | idx_uniqueid  | idx_uniqueid | 4       | NULL |    1 |   100.00 | Using index condition |
+----+-------------+---------+------------+-------+---------------+--------------+---------+------+------+----------+-----------------------+
1 row in set, 1 warning (0.02 sec)
```

##### 9.where判断中必须与字段的类型统一

```
(1) 类型不统一，register_prop_id为varchar类型
mysql> desc select register_prop_id, prop_id from t1 where register_prop_id = 1;
+----+-------------+---------------+------------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table         | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+---------------+------------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | t1            | NULL       | ALL  | PRIMARY       | NULL | NULL    | NULL |    5 |    20.00 | Using where |
+----+-------------+---------------+------------+------+---------------+------+---------+------+------+----------+-------------+
1 row in set, 3 warnings (0.01 sec)

(2) 类型统一
mysql> desc select register_prop_id, prop_id from t1 where register_prop_id = '1';
+----+-------------+---------------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
| id | select_type | table         | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra |
+----+-------------+---------------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
|  1 | SIMPLE      | t1            | NULL       | const | PRIMARY       | PRIMARY | 202     | const |    1 |   100.00 | NULL  |
+----+-------------+---------------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
1 row in set, 1 warning (0.01 sec)
```

##### 10.A left join B，后面where条件中对B的判断必须是 is null 或 is not null，如果有对B的过滤条件则应放到 on 中，否则 left join 失效变成 inner join

##### 11.对同一表的order by和group by操作分别小于3个字段（2个以内），否则考虑修改业务逻辑或跟产品聊聊

##### 12.能用union all时不要用union

```
(1) union all 不去重，少了排序操作，速度快于 union
(2) MySQL5.6及以前 union all 跟 union 一样 在执行计划中会产生 <UNION RESULT> 
(3) MySQL5.7及以后 union all 不会产生 <UNION RESULT>

# 5.7 union
mysql> desc select ugc_id from t1 union select order_id from t2 limit 1;
+----+--------------+------------+------------+-------+---------------+-------------------+---------+------+------+----------+-----------------+
| id | select_type  | table      | partitions | type  | possible_keys | key               | key_len | ref  | rows | filtered | Extra           |
+----+--------------+------------+------------+-------+---------------+-------------------+---------+------+------+----------+-----------------+
|  1 | PRIMARY      | t1         | NULL       | index | NULL          | idx_ctime         | 6       | NULL | 5703 |   100.00 | Using index     |
|  2 | UNION        | t2         | NULL       | index | NULL          | idx_state_paytime | 7       | NULL |  826 |   100.00 | Using index     |
| NULL | UNION RESULT | <union1,2> | NULL       | ALL   | NULL          | NULL              | NULL    | NULL | NULL |     NULL | Using temporary |
+----+--------------+------------+------------+-------+---------------+-------------------+---------+------+------+----------+-----------------+
3 rows in set, 1 warning (0.01 sec)

# 5.7 union all
mysql> desc select ugc_id from t1 union all select order_id from t2 limit 1;
+----+-------------+-------+------------+-------+---------------+-------------------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type  | possible_keys | key               | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+-------+---------------+-------------------+---------+------+------+----------+-------------+
|  1 | PRIMARY     | t1    | NULL       | index | NULL          | idx_ctime         | 6       | NULL | 5703 |   100.00 | Using index |
|  2 | UNION       | t2    | NULL       | index | NULL          | idx_state_paytime | 7       | NULL |  826 |   100.00 | Using index |
+----+-------------+-------+------------+-------+---------------+-------------------+---------+------+------+----------+-------------+
2 rows in set, 1 warning (0.01 sec)

```

##### 13.尽量使用主键update和delete

##### 14.select时必须指定字段名，不允许出现 "select *"，尤其不允许出现不必要的大字段（text, blob, json）

- text、blob等大字段数据存在溢出行中，可能会产生行链接问题，造成额外的I/O，尤其是排序过程中

##### 15.对于分页操作，推荐先仅查询pk做排序分页，后用pk查出余下数据

```
(1) 不推荐，排序过程select无关的字段会产生额外的开销
select ugc_id, content, create_time from t1 order by create_time desc limit 10;
(2) 推荐
select ugc_id from t1 order by create_time desc limit 10;
select ugc_id, content, create_time from t1 where ugc_id in (...);
```

##### 16.避免不必要的排序，group by 语句如果不需要排序，可以增加 order by null（8.0以前）

- 8.0 之前 group by 是有排序的 但是8.0开始 group by 没有排序

```
# 注意：这里仅仅为了展示发生了排序操作，才选择非索引字段分组，应用中建议使用索引字段分组
(1) 5.7 group by 有排序
mysql> desc select update_time, count(*) as cnt from t1 group by update_time;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+---------------------------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra                           |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+---------------------------------+
|  1 | SIMPLE      | t1    | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 5703 |   100.00 | Using temporary; Using filesort |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+---------------------------------+
1 row in set, 1 warning (0.01 sec)

(2) 加上 order by null 无排序
mysql> desc select update_time, count(*) as cnt from t1 group by update_time order by null;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-----------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra           |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-----------------+
|  1 | SIMPLE      | t1    | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 5703 |   100.00 | Using temporary |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-----------------+
1 row in set, 1 warning (0.01 sec)
```

##### 17.不建议使用不确定的函数（rand、sysdate 等），可以使用确定的函数（now 等）

- 不确定指在多条行数据下，每次查出的值不同

```
(1) 不确定
mysql> select rand() as r, sysdate(6) as s from t1 limit 10;
+---------------------+----------------------------+
| r                   | s                          |
+---------------------+----------------------------+
|  0.3222939766219761 | 2020-11-05 13:07:33.749634 |
|  0.2126485530404826 | 2020-11-05 13:07:33.749651 |
| 0.09636086607012978 | 2020-11-05 13:07:33.749655 |
|  0.8438587019628461 | 2020-11-05 13:07:33.749658 |
|  0.9302109423374859 | 2020-11-05 13:07:33.749662 |
| 0.11947863653253638 | 2020-11-05 13:07:33.749665 |
|  0.8067603863838692 | 2020-11-05 13:07:33.749668 |
|  0.6753660530553814 | 2020-11-05 13:07:33.749670 |
|  0.9565491368589449 | 2020-11-05 13:07:33.749672 |
|  0.7566475558622252 | 2020-11-05 13:07:33.749675 |
+---------------------+----------------------------+
10 rows in set (0.01 sec)

(2) 如果必须使用应先“确定化”处理后使用
mysql> select t.r, t.s from (select rand() as r, sysdate(6) as s from dual) t cross join t1 limit 10;
+--------------------+----------------------------+
| r                  | s                          |
+--------------------+----------------------------+
| 0.9135903994679362 | 2020-11-05 13:10:18.552830 |
| 0.9135903994679362 | 2020-11-05 13:10:18.552830 |
| 0.9135903994679362 | 2020-11-05 13:10:18.552830 |
| 0.9135903994679362 | 2020-11-05 13:10:18.552830 |
| 0.9135903994679362 | 2020-11-05 13:10:18.552830 |
| 0.9135903994679362 | 2020-11-05 13:10:18.552830 |
| 0.9135903994679362 | 2020-11-05 13:10:18.552830 |
| 0.9135903994679362 | 2020-11-05 13:10:18.552830 |
| 0.9135903994679362 | 2020-11-05 13:10:18.552830 |
| 0.9135903994679362 | 2020-11-05 13:10:18.552830 |
+--------------------+----------------------------+
10 rows in set (0.01 sec)
```

##### 18.连接语句建议写清楚（inner join、left join、cross join）

连接方式 | 说明
---|---
inner join | 内连接
left join | 左外连接
cross join | 笛卡尔积

##### 19.单表的批量插入建议使用 batch-insert

```
# simple insert
insert into t1(a, b) values (1, 1);
insert into t1(a, b) values (2, 2);
insert into t1(a, b) values (3, 3);
...

# batch insert
insert into t1(a, b) values (1, 1), (2, 2), (3, 3) ...
```

##### 20.多表关联查询时每个表必须指定表别名，select的字段必须带上表别名

```
# 不推荐
select user_type
    ,name 
from user_details 
inner join user 
    on user.user_id = user_details.user_id 
limit 10;

# 推荐
select u.user_type
    ,ud.name 
from user_details ud 
inner join user u 
    on u.user_id = ud.user_id 
limit 10;
```

##### 21.SQL中in包含的值应该尽量少于200个，如果超过过大考虑改写成子查询或其它方式

- in中的值过多可能导致优化器放弃索引采用全表扫描
- MySQL5.7 优化器对in子查询有多种优化方式，如果是semi join会有性能提升

##### 22.绝对禁止update语句时将 “,” 写成 ~~“and”~~

```
# 正确的update
update t1 set a = a+2, b = 200, c = 300 where id = 10;
```

##### 23.update时如果有其他表的判断条件，推荐使用 update-join，尽量避免使用 in/exists

```
# 不推荐
update t2 set t2.a = 100 where t2.id in (select t1.id from t1);

# 推荐
update t2 inner join t1 on t1.id = t2.id set t2.a = 100;
```

##### 24.对于唯一键的插入，以下操作慎用

```
# 降低性能，高并发下可能出现死锁
(1) INSERT ... ON DUPLICATE KEY UPDATE

# 非DBA禁用
(2) REPLATE INTO ...

# 同(1)
(3) INSERT IGNORE INTO ...

# 推荐程序段去判断，超高并发建议使用redis
if not exists(<ROWS>):
    try:
        <INSERT>
    except <DUPLICATE_KEY_ERROR>:  # 唯一键冲突
        pass
```

##### 25.当SQL结果仅用于条件判断时推荐 select 1 as a from t1 where ... limit 1

- 禁止查询不需要的字段

```
# 禁止
select * from t1 where user_id = 'xxx';

# 推荐
select 1 as a from t1 where user_id = 'xxx' limit 1;
```

##### 26.prepared statement 可以提高性能并避免SQL注入【待研究】

##### 27.对于非后端的值（如前端传来的）禁止直接拼接到SQL中，容易引起SQL注入，必须防注入处理后才能加入SQL中

##### 28.非特殊情况，禁止使用视图、存储过程

##### 29.禁止在MySQL中定义UDF（自定义函数）、Trigger（触发器）、Event（定时任务）

- UDF在查询中为非确定的值
- Trigger会增加MySQL负担，仅允许DBA临时使用，使用后立即删除
- Event推荐程序端去实现
- UDF、Trigger、Event等会使得MySQL不易维护

##### 30.尽量避免在MySQL中使用聚合统计，如果有需要推荐在相应表做冗余字段，程序端做处理

- 报表等业务，推荐定时（如：每月一次）在备库执行然后插入到专门的统计表中

```
(1) 不推荐
select count(*) from follow where user_id = 'xxx' and status = 1;

(2) 推荐
CREATE TABLE `user` (
  ...
  number_of_fans int not null default 0 comment '粉丝数',  # 冗余字段
  ...
)
select user_id, number_of_fans from user where user_id = 'xxx';
```

##### 31.尽量避免在循环中查询SQL

- MySQL查询开销、网络开销

```
# 不推荐
for user_id in user_ids:
    sql = 'select nickname, avatar from user where user_id = %s'
    cur.execute(sql, (user_id,))
    ...

# 推荐
sql = 'select user_id, nickname, avatar from user where user_id in %s'
cur.execute(sql, (user_ids,))
...
```

##### 32.大批量更新或删除操作建议加limit分段提交，注意复制环境必须 binlog_format=row

```
# 不推荐
delete from t1;

# 推荐
limit_rows = 1000
sql = 'delete from t1 limit %s' % limit_rows
while True:
    conn.begin()
    rows = cur.execute(sql)
    conn.commit()
    if not rows:
        break
```

##### 33.生成列（Generated column）为5.7新增概念，通常用于索引必须函数处理的字段

```
# 查询今天过生日的用户

# 添加虚拟列
alter table user add birthday_md char(4) GENERATED ALWAYS AS (date_format(birthday, '%m%d')) VIRTUAL comment '生日的月份和日期';
# 对虚拟列创建索引
alter table user add index idx_bmd(birthday_md);

# 查询方式一
select user_id 
from user 
where date_format(birthday, '%m%d') = date_format(now(), '%m%d');

# 查询方式二
select user_id 
from user 
where birthday_md = date_format(now(), '%m%d');
```

##### 34.字符集统一使用 utf8mb4；如果老项目使用utf8则该项目统一utf8，新项目统一utf8mb4

##### 35.建表时不允许指定 ROW_FORMAT


### 参考文档

[https://github.com/zhishutech/mysql-sql-standard](https://github.com/zhishutech/mysql-sql-standard)
