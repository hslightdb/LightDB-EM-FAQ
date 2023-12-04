# LightDB-EM-FAQ

## 1、EM操作遇到问题，该提供哪些信息给开发团队才能更高效的排查问题？
```
1. 当前问题界面的截图，并简单描述；
2. 提供引发问题的操作步骤，越详细越好；
3. 立即备份em的服务端日志，日志文件在em安装目录下的 /logs 文件夹；
4. 浏览器按F12打开控制台，查看是否有明显的红字报错，如有报错可截图提供给开发团队；
5. 联系开发团队提供远程支持；
```

## 2、上传文件时，出现上传失败的错误，该如何定位解决？

![errupload.png](https://github.com/Jaywzy/LightDB-EM-FAQ/blob/master/static/errupload.png "DBSCAN Performance Comparison")
```
解决方法 ：
	a :通过浏览器控制台查看上传接口提示:Processing of multipart/form-data request failed. 设备上没有空间
	b: 修改em配置文件jrescloud.properties，配置server.tomcat.basedir的地址为em服务所在路径
```

## 3、手工调整了em内嵌的绿色版本的lightdb数据库监听端口引发的问题解决
```
a、em内嵌的绿色版本的lightdb的版本(13.8-22.4)已支持单机分布式，默认启动后监听端口为5434，
如果手动修改了em内嵌的绿色版本的lightdb监听端口(配置文件${LTDATA}/lightdb.conf中配置项port)为5436，在em启动后(em启动过程中会启动绿色版本lightdb数据库)需要进行如下操作：

1、[lightdb@192_168_76_128 scripts]$ ltsql -h localhost -p5436 -d postgres -c 'update cron.job set nodeport=5436;'
UPDATE 5
2、[lightdb@192_168_76_128 scripts]$ ltsql -h localhost -p5436 -d postgres -c "update servers set connstr='dbname=postgres port=5436';"
UPDATE 1
3、[lightdb@192_168_76_128 scripts]$ ltsql -h localhost -p5436 -d em -c "update pg_dist_node set nodeport=5436;"
UPDATE 1

b、验证是否操作成功：
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