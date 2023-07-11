# Synjq 数据闪送平台使用指南

## 前言

此文档包含了 Synjq 数据闪送用户操作相关的说明,包括产品的基本功能介绍，支持的数据上下游节点的类型,数据节点,链路,管理等相关内容的配置，此外 Synjq 是一款

- 跨平台
- 完全容器化
- 高可用
- 高容错
- 高拓展
- 高一致性

同时支持多源异构的数据库同步高效产品

## 产品注意事项

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

## 产品拓扑结构

![](https://image-1302181629.cos.ap-beijing.myqcloud.com/synjq-%E6%8B%93%E6%89%91%E7%BB%93%E6%9E%84.png)</br>

> Synjq[数据的闪送]是一款支持异构数据源的数据同步工具，它分为以下 4 个模块：</font>

1. Synjq Engine 集群</br>
2. Synjq Web 端</br>
3. Synjq Query 中间件 </br>
4. Synjq.jar 主程序 </br>

## 产品交付

> 在产品交付的 synjq.zip 压缩包中，包含了 4 个模块的软件包 + 1 个启动服务的配置文件：

1. Synjq-engine.tar
2. Synjq-query.tar
3. Synjq-web.tar
4. Synjq-daemon.tar
5. Docker-compose.yaml

## 产品支持的上下游节点

> Synjq 支持的上游节点信息：

![](https://image-1302181629.cos.ap-beijing.myqcloud.com/synjq--%E4%B8%8A%E6%B8%B8%E8%8A%82%E7%82%B9.png)

> Synjq 支持的下游节点信息：

![](https://image-1302181629.cos.ap-beijing.myqcloud.com/synjq--%E4%B8%8B%E6%B8%B8%E8%8A%82%E7%82%B9.png)
