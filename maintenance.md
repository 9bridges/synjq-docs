# Synjq 运维操作

> 对于运维人员的常规运维操作

## 怎么启动和关闭 synjq 容器？</br>

```javascript {.line-numbers}
首先Synjq产品是使用docker-compose.yaml文件一键启停的，不需要一个个的停止关闭容器

#：docker-compose down 关闭所有容器
#：docker-compose up -d 启动所有Synjq相关的容器
```

## 怎么查看日志信息? </br>

```javascript {.line-numbers}
查看日志的方法有两种：
       1.可以进入taskmanager容器进行查看
       2.直接在web界面上查看

#： docker exec -it synjq-taskmanager-1 /bin/bash
#:  cd log查看后缀为log的日志信息
```

## Web 数据节点和链路配置

<details>
<summary>Web</summary>

> 添加数据节点信息

![](https://image-1302181629.cos.ap-beijing.myqcloud.com/synjq--%E6%B7%BB%E5%8A%A0%E8%8A%82%E7%82%B9.png)

> 填写节点的 ip，端口等相关信息

![](https://image-1302181629.cos.ap-beijing.myqcloud.com/synjq-%E8%8A%82%E7%82%B9%E4%BF%A1%E6%81%AF.png)

> 配置链路

![](https://image-1302181629.cos.ap-beijing.myqcloud.com/synjq-%E6%B7%BB%E5%8A%A0%E9%93%BE%E8%B7%AF.png)

> 填加链路信息，填写完保存启动链路即可

![](https://image-1302181629.cos.ap-beijing.myqcloud.com/synjq--%E9%93%BE%E8%B7%AF%E4%BF%A1%E6%81%AF.png)

</details>
以上配置为通用配置，个别数据库需要填写其他参数，请联系九桥工程师进程配置！
