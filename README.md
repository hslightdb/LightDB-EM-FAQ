# LightDB-EM-FAQ


## 手工调整了em内嵌的绿色版本的lightdb监听的端口，需要进行如下操作
```
em内嵌的绿色版本的lightdb的版本(13.8-22.4)已支持单机分布式，默认启动后监听端口为5434，
如果手动修改了em内嵌的绿色版本的lightdb监听端口(配置文件${LTDATA}/lightdb.conf中配置项port)为5436，在em启动后(em启动过程中会启动绿色版本lightdb数据库)需要进行如下操作：
1、[lightdb@192_168_76_128 scripts]$ ltsql -h localhost -p5436 -d postgres -c 'update cron.job set nodeport=5436;'
UPDATE 5
2、[lightdb@192_168_76_128 scripts]$ ltsql -h localhost -p5436 -d postgres -c "update servers set connstr='dbname=postgres port=5436';"
UPDATE 1
3、[lightdb@192_168_76_128 scripts]$ ltsql -h localhost -p5436 -d em -c "update pg_dist_node set nodeport=5436;"
UPDATE 1

验证是否操作成功：
1、[lightdb@192_168_76_128 scripts]$ ltsql -h localhost -p5436 -d em -c "select * from pg_dist_node;"
 nodeid | groupid | nodename  | nodeport | noderack | hasmetadata | isactive | noderole | nodecluster | metadatasynced | shouldhaveshards
--------+---------+-----------+----------+----------+-------------+----------+----------+-------------+----------------+------------------
      1 |       0 | localhost |     5436 | default  | t           | t        | primary  | default     | t              | t
(1 row)

2、[lightdb@192_168_76_128 scripts]$ ltsql -h localhost -p5436 -d postgres -c "select connstr from servers;"
          connstr
---------------------------
 dbname=postgres port=5436
(1 row)

                 |                  |
(1 row)
3、[lightdb@192_168_76_128 scripts]$ ltsql -h localhost -p5436 -d postgres -c "select * from cron.job;"
 jobid |   schedule    |              command              | nodename  | nodeport | database | username | active |         jobname
-------+---------------+-----------------------------------+-----------+----------+----------+----------+--------+--------------------------
     1 | */5 * * * *   | SELECT lt_catalog.pg_show_plans() | localhost |     5436 | postgres | lightdb  | t      | lt_show_plans
     2 | */1 * * * * * | SELECT collect_activity_history() | localhost |     5436 | postgres | lightdb  | t      | collect_activity_history
     3 | */1 * * * *   | SELECT collect_activity_profile() | localhost |     5436 | postgres | lightdb  | t      | collect_activity_profile
     4 | 0 0 * * *     | SELECT clean_activity_profile()   | localhost |     5436 | postgres | lightdb  | t      | clean_activity_profile
     5 | */10 * * * *  | SELECT lt_catalog.take_sample()   | localhost |     5436 | postgres | lightdb  | t      | take_sample
(5 rows)
```