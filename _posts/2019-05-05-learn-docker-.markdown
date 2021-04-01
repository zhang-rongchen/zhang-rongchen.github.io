---
layout: post
title:  "learn docker"
date:   2019-05-05 15:51:45 +0200
categories: docker
---
docker ps -a 查看已有docker信息

docker ps 查看正在运行docker

docker stop 停止docker

docker start 开启docler

docker exec -it 9a9ce4ffe1b5 /bin/bash 进入docker shell

docker cp host_path docker_iddocker_path 主机host复制文件到docker容器



新建docker无法配置conda环境的问题解决方案：

RUN /bin/bash -c "source activate tf && conda install .. &&.."

因为在docker build过程中默认使用sh运行命令，因此需要单独打开/bin/bash，而且需要执行的命令要在一个/bin/bash进程中完成



拉取基础镜像 

docker pull daocloud.io/daocloud/tensorflow:1.14.0-py3



启动基础镜像 

-p 映射端口，将容器的80端口映射到主机的2306端口

-v 挂载目录，将主机的 /usr/local/CloudRecognition路径挂载到容器的 /usr/local/CloudRecognition路径下

docker run -p 2306:80 -v /usr/local/CloudRecognition:/usr/local/CloudRecognition -it --name=ImageRetrieval_DL daocloud.io/daocloud/tensorflow:1.14.0-py3 bash 



在基础镜像中配置环境并测试代码

pip install ...

apt-get install ..



完成测试后，将测试通过的镜像提交到仓库中

docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]

docker commit -a "zrc" -m "add psutil" 80b336f8ae48 image_retrieval_dl:v2 





跨服务器拷贝文件

scp [host服务器路径] [[user@]host2:][目标服务器路径]

scp trt_retrieval.tar slash@10.10.27.66:~



docker程序挂起

nohup [commands] &