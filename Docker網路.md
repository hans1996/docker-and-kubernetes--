---
title: 'Docker網路'
disqus: hackmd

---

---

Docker 網路
===

[TOC]



網路
---
![](https://i.imgur.com/UTUD0Tv.png)

![](https://i.imgur.com/P0rLFmD.png)

![](https://i.imgur.com/M07bprk.png)



127.0.0.11 對內 : container 名子互通
            對外 : 用網址
            
docker 內部預設dns 是用 8.8.8.8 (自己設新增網路會比較好，container 不能用名子互 ping (用ip還是可以) , 自己設的網路 dns 會是 127.0.0.11)

rounter 連到


### Docker-Compose

目標:系統文件化

#### Docker Compose - 建置叢集應用系統

![Uploading file..._lou0yzvo3]()

#### 安裝 Docker Compose

```gherkin=
建立 ddg53 虛擬主機 (等 5 分鍾)
$ dkh start ddg53 terminal

$ sudo apt update; sudo apt install docker.io -y; sudo reboot

到此網站 https://github.com/docker/compose/releases 查詢 docker-compose 最新版本代號
$ sudo curl -L https://github.com/docker/compose/releases/download/1.27.4/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose

$ sudo chmod +x /usr/local/bin/docker-compose

顯示 docker-compose 版本
$ docker-compose version
docker-compose version 1.27.4, build 18f557f9
docker-py version: 4.3.1
CPython version: 3.7.7
OpenSSL version: OpenSSL 1.1.0l  10 Sep 2019
```
#### 建置叢集應用系統
```gherkin=
$ mkdir wulin; cd ~/wulin

$ nano myapp.yml 
version: '3'
services:
  s1:
    image: dafu/alpine.plus
    ports:
      - "22101:22"
  s2:
    image: dafu/alpine.plus
    ports:
      - "22102:22"

$ docker-compose -f myapp.yml up -d
Creating network "wulin_default" with the default driver
Creating wulin_s1_1 ... done
Creating wulin_s2_1 ... done


[註] -d  Detached mode: Run containers in the background
```

### ex: Docker compose 應用建立教學環境

#### myapp.yml
![](https://i.imgur.com/YfHEtZk.png)

#### student.yml
![](https://i.imgur.com/YWi1unO.png)

寫在不同的yml檔案共用同一個wulin的目錄，所以有相同的網路
![](https://i.imgur.com/tucNW2a.png)


