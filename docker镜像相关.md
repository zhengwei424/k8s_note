# Docker Hub

## 创建一个镜像仓库

==仓库名就是镜像名==

例：创建一个busybox镜像仓库

![image-20200412170718186](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200412170718186.png)



## 账号登录

```shell
# docker login <直接回车>
```



## 推送镜像（前提是先docker login）

```shell
# docker tag local-image:tagname new-repo:tagname
# docker push new-repo:tagname
```

例：

```shell
# docker tag busybox:1.28 zhengwei424/busybox:1.28
# docker push zhengwei424/busybox:1.28
```



## 搜索镜像

```shell
# docker search 镜像名称
```



## 拉取镜像

```shell
# docker pull 镜像名称
```



# 阿里云docker镜像仓库

使用方法：

登录阿里云——>在搜索栏输入"容器镜像服务"——[点击进入](https://cr.console.aliyun.com/cn-hangzhou/instances/repositories)

![image-20200412172323217](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200412172323217.png)

## 使用方法

1. 创建命名空间

2. 创建镜像仓库（即待上传的镜像名）

3. 操作镜像

   1. 登录阿里云Docker Registry

      ```shell
      $ sudo docker login --username=运维青年 registry.cn-hangzhou.aliyuncs.com
      ```

      

   2. 从Registry中拉取镜像

      ```shell
      $ sudo docker pull registry.cn-hangzhou.aliyuncs.com/zw_k8s/busybox:[镜像版本号]
      ```

      

   3. 将镜像推送到Registry

      ```shell
      $ sudo docker login --username=运维青年 registry.cn-hangzhou.aliyuncs.com
      $ sudo docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/zw_k8s/busybox:[镜像版本号]
      $ sudo docker push registry.cn-hangzhou.aliyuncs.com/zw_k8s/busybox:[镜像版本号]
      ```

4. [获取镜像加速](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors)

   ![image-20200412173137527](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200412173137527.png)

