---
title: Mysql常用操作
---
整理一些常用和效率快一点的sql

+ Mysql对同一张表删除重复数据
  ``` sql
  DELETE w FROM subscribe_msg_list w RIGHT JOIN ( SELECT *, count(1) AS cc FROM subscribe_msg_list GROUP BY type, user_id, time HAVING cc > 1 ) t ON w.id = t.id;
  ```
+ MySQL8 开启慢查询日志，并记录到表中。
  ``` sql
  SHOW VARIABLES LIKE '%slow_query%';
  SET GLOBAL log_output = 'TABLE'; --  开启慢日志,纪录到 mysql.slow_log 表
  SET GLOBAL long_query_time = 1; --  设置超过1秒的查询为慢查询
  SET GLOBAL slow_query_log = 'ON'; --  打开慢日志记录
  SELECT CONVERT ( sql_text USING utf8 ) sql_text FROM mysql.slow_log -- 查询慢sql的 日志
  ```
+ MySQL group by 统计每5分钟数据量
  ``` sql
  select
  concat(from_unixtime(create_time - create_time % 300), ' ~ ', from_unixtime(create_time - create_time % 300 + 300)) as period,
  count(1) as record_count
    from 打卡日志表
    WHERE create_time BETWEEN 1603123200 and 1603209599
    group by period
    order by record_count DESC;
  ```
+ Mysql 查询数据是否存在
  ``` sql
  ### 差：
  select count(*) from tab_record where a=1 and b= 1;
  ### 优：
  select id from tab_record where a=1 and b=1 limit 1;
  ```
