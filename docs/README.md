# &#x1F4A1;  Synjq(数据的闪送)用户操作指南


# Synjq介绍                                          
## 前言
此文档包含了Synjq数据闪送用户操作相关的说明,包括产品的基本功能介绍，支持的数据上下游节点的类型,数据节点,链路,管理等相关内容的配置。</br>
### 产品注意事项

<table style="background-color: #f2f2f2;" >
<tr style="background-color: #ddd;">
<th>源端</th>
<th>备端</th>
<th>中间机</th>
</tr>
<tr>
<td>提供具有抓取同步库表权限的用户名/密码</td>
<td>提供具有写入同步库表权限的用户名/密码</td>
<td>标准配置：8核 / 32g 内存 / 500g 磁盘空间</td>
</tr>
<tr>
<td>提供源端节点的 IP 并开放数据库的端口，如：1521/3306/1433 等常用数据库端口</td>
<td>创建与源端同步库表相对应的库表结构(只针对于目前不支持ddl的database)</td>
<td>安装 Docker 引擎 (Docker 引擎  v >= 20)</td>
</tr>
<tr>
<td>开启源端数据抓取 相关配置权限，见文档「源端配置」(归档必须开启)</td>
<td>提供备端节点的 IP 并开放数据库的端口，如：1521/3306/1433 等常用数据库端口</td>
<td>开放端口: TCP:3000 以便正常访问 synjq web 端</td>
</tr>
</table>

<hr style="border: 2px solid grey;">

### 产品拓扑结构

![](https://image-1302181629.cos.ap-beijing.myqcloud.com/synjq-%E6%8B%93%E6%89%91%E7%BB%93%E6%9E%84.png)</br>

>Synjq[数据的闪送]是一款支持异构数据源的数据同步工具，它分为以下4个模块：</font>

1. Synjq Engine 集群</br>
2. Synjq Web 端</br>
3. Synjq Query 中间件 </br>
4. Synjq.jar 主程序 </br>

### 产品交付
>在产品交付的 synjq.zip 压缩包中，包含了 4 个模块的软件包 + 1 个启动服务的配置文件：
1. Synjq-engine.tar
2. Synjq-query.tar
3. Synjq-web.tar
4. Synjq-daemon.tar
5. Docker-compose.yaml

# Synjq开始部署
## 上传导入镜像
synjq的部署流程基于docker-compose构建，能够实现严格意义上的[一键部署]

1. 上传synjq软件包
```
synjq.tgz
```
2. 解压软件包
```
tar -xvf synjq.tgz
```
3. 执行脚本安装docker
```
tar -xvf synjq-docker.tgz
sh docker.sh //执行脚本
```
![](https://image-1302181629.cos.ap-beijing.myqcloud.com/docker-version.png)</br>

4. 开始load Synjq镜像
```
docker load -i synjq-query.tar
docker load -i synjq-web.tar
docker load -i synjq-engine.tar
docker load -i synjq-daemon.tar
```
![](https://image-1302181629.cos.ap-beijing.myqcloud.com/docker-images.png)</br>

5. 一键启动Synjq
```
docker-compose up -d
```
![](https://image-1302181629.cos.ap-beijing.myqcloud.com/docker-compose.png)</br>

## Synjq-jar包上传
>http://ip:3000/home || 打开web页面</br> 
>上传jar包 

![](https://image-1302181629.cos.ap-beijing.myqcloud.com/Synjq-web.png)</br>

>至此Synjq部署完毕
<hr style="border: 2px solid grey;">


# Synjq源端数据库配置
## oracle源端配置
>分为三种同步方式：

1. xstream
2. logminer
3. oragent

### xstream配置
<pre>
  <code style="white-space: pre-wrap;">
1. 开启归档：
          alter database archivelog; 

2. 打开xstream:
          alter system set enable_goldengate_replication=true;  (oracle数据库11.2.0.4.0以下不支持)

3. 开启oracle库级别或者表级别的全列补偿日志：
          alter database add supplemental log data (all) columns;  （库级别）
          ALTER TABLE inventory.customers ADD SUPPLEMENTAL LOG DATA (ALL) COLUMNS; (表级别)

4. 创建出站服务器管理用户 synjqadmin：
          CREATE TABLESPACE synjq_adm_tbs DATAFILE '/根据客户实际路径修改/synjq_adm_tbs.dbf' SIZE 25M REUSE AUTOEXTEND ON MAXSIZE UNLIMITED;

          CREATE USER synjqadmin IDENTIFIED BY synjqadmin DEFAULT TABLESPACE synjq_adm_tbs QUOTA UNLIMITED ON synjq_adm_tbs;

          GRANT CREATE SESSION TO synjqadmin;

BEGIN
   DBMS_XSTREAM_AUTH.GRANT_ADMIN_PRIVILEGE(
      grantee                 => 'synjqadmin',
      privilege_type          => 'CAPTURE',
      grant_select_privileges => TRUE
   );
END;
/         

5. 创建登录数据库的用户 synjq：
         CREATE TABLESPACE synjq_tbs DATAFILE '/根据客户实际情况修改/synjq_tbs.dbf' SIZE 25M REUSE AUTOEXTEND ON MAXSIZE UNLIMITED;

         CREATE USER synjq IDENTIFIED BY synjq DEFAULT TABLESPACE synjq_tbs QUOTA UNLIMITED ON synjq_tbs;

         GRANT CREATE SESSION TO synjq;
         GRANT SELECT ON V_$DATABASE to synjq;
         GRANT FLASHBACK ANY TABLE TO synjq;
         GRANT SELECT ANY TABLE to synjq;
         GRANT LOCK ANY TABLE TO synjq;
         GRANT select_catalog_role to synjq;

6. 使用 synjqadmin 用户登录数据库:
   (创建出站服务器，下面例子为创建名称为 debug 的出站服务器,抓取 FZY 和 FZS1 用户下所有表的增量数据)
conn synjqadmin/synjqadmin

DECLARE
  tables  DBMS_UTILITY.UNCL_ARRAY;
  schemas DBMS_UTILITY.UNCL_ARRAY;
BEGIN
    tables(1)  := NULL; 
    schemas(1) := 'TEST';   
  DBMS_XSTREAM_ADM.CREATE_OUTBOUND(
    server_name     =>  'debug0',
    table_names     =>  tables,
    schema_names    =>  schemas);
END;
/

<font color="#cd5c5c">提示：开几个并发创建几个出站服务器后面加数字例如：一个并发就是debug0</font>

7. 执行以下命令允许 synjq 用户连接 XStream 出站服务器
BEGIN
   DBMS_XSTREAM_ADM.ALTER_OUTBOUND(
      server_name  => 'debug0',
      connect_user => 'synjq'
   );
END;
/

8. 删除出站服务器

BEGIN
   DBMS_XSTREAM_ADM.DROP_OUTBOUND(
   server_name => 'DEBUG'
   );
END;
/ 


9. 查询出站服务器有几个
   SELECT CLIENT_NAME ,client_status,status ,captured_scn,last_enqueued_scn, error_message FROM ALL_CAPTURE；
  </code>
</pre>

### logminer配置
<pre>
  <code style="white-space: pre-wrap;">
1. 开启归档：
          alter database archivelog;

2. 开启Minimal supplemental logging:
          ALTER DATABASE ADD SUPPLEMENTAL LOG DATA  (all) columns;  (整库开启)
          ALTER TABLE test1.test2 ADD SUPPLEMENTAL LOG DATA (ALL) COLUMNS; (表级开启)
  </code>
</pre>

### oragent配置
<pre>
  <code style="white-space: pre-wrap;">
1.开启归档：
         alter database archivelog;

2. 进入oracle-database配置

create or replace view XKCCLE as select * from sys.x$kccle;
create or replace view XKCCCP as select * from sys.x$kcccp;
create or replace view ORAGENT_XKSPPI as select * from X$KSPPI;
create or replace view ORAGENT_XKSPPSV as select * from X$KSPPSV;
alter database force logging;
alter database add supplemental log data;
create user oragent1 identified by oragent1;
create role oragent_role identified by oragent_role;
grant connect,lock any table,select any table,select any dictionary,alter system,execute any type to oragent_role,oragent1;
grant oragent_role to oragent1;
grant dba to oragent1;
grant execute on dbms_flashback to oragent_role,oragent1;
alter user  oragent1 quota 2m on users;
create table oragent1.oragenttemp (f1 int);
  </code>
</pre>
<hr style="border: 2px solid grey;">

## 达梦源端配置
>1.开启附加日志

SP_SET_PARA_VALUE(1,'RLOG_APPEND_LOGIC',2);

>2.开启归档

ALTER DATABASE MOUNT; </br>
alter database add archivelog 'type=local,dest=(自己实际的路径),file_size=64,space_limit=0'; </br>
ALTER DATABASE ARCHIVELOG;</br>
ALTER DATABASE OPEN;</br>

<hr style="border: 2px solid grey;">

## TIDB源端配置

>1.查看GC的配置

select VARIABLE_NAME,VARIABLE_VALUE from mysql.tidb where VARIABLE_NAME like "tikv_gc%";

>2.预留足够的时间给cdc同步修改tikv_gc_life_time

update mysql.tidb set VARIABLE_VALUE='24h' where VARIABLE_NAME='tikv_gc_life_time';


<hr style="border: 2px solid grey;">

## Mysql源端配置

>1.创建用户并授权

CREATE USER 'xstrem'@'localhost' IDENTIFIED BY 'xstrem'; </br>

GRANT SELECT, RELOAD, SHOW DATABASES, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'xstrem' IDENTIFIED BY 'xstrem';
FLUSH PRIVILEGES;</br>

>2.开启binlog

cat \<< EOF >>/etc/my.cnf </br>
server-id         = 223344</br>
log_bin           = mysql-bin</br>
binlog_format     = ROW</br>
binlog_row_image  = FULL</br>
expire_logs_days  = 10</br>
EOF

service  mysqld restart </br>
show variables like 'log_%'; </br> (查看日否开启)

>设置gtid

SET GLOBAL ENFORCE_GTID_CONSISTENCY = 'WARN';</br>
SET GLOBAL ENFORCE_GTID_CONSISTENCY = 'ON';</br>
SET GLOBAL GTID_MODE = 'OFF_PERMISSIVE';</br>
SET GLOBAL GTID_MODE = 'ON_PERMISSIVE';</br>
SET GLOBAL GTID_MODE = 'ON';</br>

show global variables like '%GTID%'; //查询 </br>

>配置会话超时

set global wait_timeout=28800;</br>
set global interactive_timeout=28800;</br>

>启动查询日志时间

set binlog_rows_query_log_events=on;</br>

show global variables where variable_name = 'binlog_row_value_options';</br>
<hr style="border: 2px solid grey;">


