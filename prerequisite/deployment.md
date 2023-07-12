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
