# Synjq源端数据库配置
## oracle源端配置
>分为三种同步方式：

1. xstream
2. logminer
3. oragent

## ora-xstream配置
<details>
<summary>xstream</summary>
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

8.删除出站服务器

BEGIN
DBMS_XSTREAM_ADM.DROP_OUTBOUND(
server_name => 'DEBUG'
);
END;
/


9.查询出站服务器有几个
SELECT CLIENT_NAME ,client_status,status ,captured_scn,last_enqueued_scn, error_message FROM ALL_CAPTURE；
</code>
</pre>

</details>

## ora-logminer配置
<details>
<summary>logminer</summary>
<pre>
  <code style="white-space: pre-wrap;">
1.开启归档：
          alter database archivelog;

2.开启Minimal supplemental logging:
ALTER DATABASE ADD SUPPLEMENTAL LOG DATA  (all) columns;  (整库开启)
ALTER TABLE test1.test2 ADD SUPPLEMENTAL LOG DATA (ALL) COLUMNS; (表级开启)
</code>
</pre>
</details>

## ora-oragent配置
<details>
<summary>oragent</summary>
<pre>
  <code style="white-space: pre-wrap;">
1.开启归档：
         alter database archivelog;

2.进入oracle-database配置

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

3.上传oragent安装包，这一步需要联系九桥工程师进行配置！
</code>
</pre>
</details>
<hr style="border: 2px solid grey;">

## 达梦(DM)源端配置
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

<details>
<summary>Mysql</summary>

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
</details>

## SqlServer源端配置
>为指定的数据库开启CDC

USE MyDB</br>
GO</br>
EXEC sys.sp_cdc_enable_db</br>
GO</br>

>为指定表开启 CDC

USE MyDB </br>
GO</br>
EXEC sys.sp_cdc_enable_table</br>
@source_schema = N'dbo',</br>
@source_name   = N'MyTable',</br>
@role_name     = N'MyRole',  </br>
@filegroup_name = N'MyDB_CT',</br>
@supports_net_changes = 0</br>
GO</br>

<hr style="border: 2px solid grey;">
