+++
title = 'Docker'
date = 2023-11-26T01:39:58+08:00
tags = ["Python", "Virtual Environment"]
category = ["程式"]
+++

## Links
* [安裝教學1](https://blog.gtwang.org/virtualization/ubuntu-linux-install-docker-tutorial/)
* [安裝教學2](https://azole.medium.com/docker-container-%E5%9F%BA%E7%A4%8E%E5%85%A5%E9%96%80%E7%AF%87-1-3cb8876f2b14)
* 可以把現有的 image pull 下來用 -> [Docker Hub](https://hub.docker.com/search?q=&type=image)
* [基礎入門](https://azole.medium.com/docker-container-%E5%9F%BA%E7%A4%8E%E5%85%A5%E9%96%80%E7%AF%87-1-3cb8876f2b14)
* [聖經?!](https://philipzheng.gitbook.io/docker_practice/)
* [建立 docker 環境1](https://peihsinsu.gitbooks.io/docker-note-book/content/docker-build.html)
* [建立 docker 環境2](https://israynotarray.com/docker/20230127/3334908414/)
* [官方文件](https://docs.docker.com/)
* [官方文件 (Dockerfile)](https://docs.docker.com/engine/reference/builder/)

---
## 指令
* 查看 docker 版本

```shell=
$ docker version
```

* 查看現有的 images (還沒跑起來的映象檔)
```shell=
$ docker images

# (Optional)
$ docker images ls
```

* 查看執行中的 containers
```shell=
$ docker container ls -a

# or 
$ docker ps -a
```

* Shutdown container
```shell=
$ docker container stop <CONTAINER_ID>

# 然後還要執行這個
$ docker rm <container_ID>
```

* Pull image
```shell=
$ docker pull <docker_name>
```

* Remove image
```shell=
$ docker image rm <IMAGE_ID>
```

* 啟動 image，且進入其 bash 模式
```shell=
$ docker run -it <docker_name or IMAGE_ID> bash
```

--- 
## 創建 Docker 環境
- Step1: 創立 **Dockerfile**
    - **FROM**: 指定base image

    - **WORKDIR**: 指定 docker 執行起來時候的預設目錄位置

    - **RUN**: 指定 build 過程中所要執行的指令與安裝動作
    
    - **ENV**: 指令啟動後的環境變數

    - **COPY**: 複製檔案 (通常預設路徑會在根目錄，因此先複製要用的檔案後，再創立 WORKDIR)

- Step2: **Build**
```shell=
# tag: 通常用作區分版本
$ docker build -t <image-name>:<tag> <path-to-dockerfile>

# example
$ docker build -t image_test:v1 .
```

- Step3: **啟動**
    - ```-p```: 指定 Port
    - ```-d```: Container 背景執行
    - ```-i```: 互動模式
```shell=
$ docker run -p <local_port>:<container_port> -d <image_name>

# example
$ docker run -p 3000:3000 -dit image_test:v1 bash
```