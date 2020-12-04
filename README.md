---
title: 'Docker'
disqus: hackmd

---

---

Cloud Native 雲原生系統
===

[TOC]

CNCF(cloud native computing foundation) 開放虛擬技術原始碼

三大企業軟體雲
1. 微軟 Microsoft Azure
2. google cloud
3. amazon  aws

contain 貨櫃
kubernetes 管理很多台實體電腦，每台實體電腦裡面有很多艘船(Docker)，可是我們只有一台電腦，所以要用虛擬技術(ex:VMware workstation 16 player)

目前使用 Nested virtualization 作為練習環境

![](https://i.imgur.com/baKMiPR.png)

---




dkh ps 
-m 2560  (2.5G)
Macadd(網路卡出廠代號(唯一識別))後兩碼52 與 虛擬電腦 ddg52 是網路卡mac的最後一碼
網路卡出廠代號

前兩碼 
52:54 代表linux kvm 的虛擬電腦

![](https://i.imgur.com/d33acsQ.png)

00:0c vmware workstaion 在用

![](https://i.imgur.com/XHCYjhB.png) 

C8-D9 代表intel出廠的代號

![](https://i.imgur.com/iiU5pxX.png)
    

Linux 程序管理
---
```gherkin=
# 查看 sudo 的 system call
sudo find / -name *.so

#條列式顯示所有程序關連
ps aux

#階層式顯示所有程序關連     (附註systemd的d就是daemon的意思)
pstree -ph

#批次顯示程序資訊
top -bin 2 -d 1 > pid.txt

```

```gherkin=
#撰寫永不停止程式

#!/bin/bash
# trap ctrl-c and call ctrl_c()
trap ctrl_c INT

function ctrl_c() {
   echo -e "\n** Trapped CTRL-C"
   # 下式顯示游標
   tput cnorm
   exit 0 
}

clear
# 下式隱藏游標
tput civis

while [ 1 ]
do
  echo -ne "\033[10;10f Hi Chen : "   
  date
done
```
> ctrl +z 可以背景執行命令 ,fg 可跳回前景執行

## K8S 練習環境與架構圖

![](https://i.imgur.com/RN1bTrz.png)




#### 查看ip
---
```gherkin=
 ifconfig
 hostname -I
```


> Set UID (ex 4755)) 設在檔案,
chmod 1755 設在目錄



#### 啟動虛擬機

```gherkin=
igred@cvn71:~/wk/cnt$ which ping
/usr/bin/ping
bigred@cvn71:~/wk/cnt$ ls -al /usr/bin/ping
-rwxr-xr-x 1 root root 72776  1月 31  2020 /usr/bin/ping
bigred@cvn71:~/wk/cnt$ k8s start cg60
cg60 started
bigred@cvn71:~/wk/cnt$ ssh 172.29.0.60
```

```gherkin=
Welcome to Kubernetes Cluster V 20.01
IP : 172.29.0.60

bigred@cg60:~$ which ping
/bin/ping
bigred@cg60:~$ ls -al /bin/ping     
-rwsr-xr-x 1 root root 64424 Jun 28  2019 /bin/ping
```

## Linux Capabilities 
```gherkin=
$ sudo useradd -m -s /bin/bash rbean

$ echo "rbean:rbean" | sudo chpasswd

#設定 rbean 家目錄中的 python3 命令檔, 具有 Capabilities setuid 權限, 代表 python3 所執行的 程式可設定 setuid 功能
$ sudo cp /usr/bin/python3  /home/rbean 

$ sudo setcap  cap_setuid+ep  /home/rbean/python3

$ exit


$ ssh rbean@172.29.0.52
rbean@172.29.0.52's password: rbean

#列出有設定 Linux capabilities 的所有命令
$ getcap -r / 2>/dev/null
/home/rbean/python3 = cap_setuid+ep
/bin/ping = cap_net_raw+ep
/usr/lib/x86_64-linux-gnu/gstreamer1.0/gstreamer-1.0/gst-ptp-helper = cap_net_bind_service,cap_net_admin+ep
/usr/bin/mtr-packet = cap_net_raw+ep
/usr/bin/traceroute6.iputils = cap_net_raw+ep



#由以下命令得知 python3 命令檔並沒有設定 setuid 功能
$ ls -al python3
-rwxr-xr-x 1 root root 5453504 Jul 28 04:56 python3


#在以下 python3 命令所執行的 程式可執行 os.setuid(0) 這行命令, 提升 /bin/bash 這命令為 root 權限
$ ./python3 -c 'import os;os.setuid(0);os.system("/bin/bash")'
root@ub204:~# id
uid=0(root) gid=1001(rbean) groups=1001(rbean)
root@ddg52:~# touch  /hi
root@ddg52:~# exit


$ exit

```



## Namespace 可以把一段程式隔離成一台電腦,一段程式是一個電腦

![](https://i.imgur.com/q1Ws6C9.png)

> container 就是namespace 的技術做出來的




### Isolating the Hostname 
```gherkin=
# unshare 就是 linux 的 namespace 隔離的意思
bigred@ddg52:~$ sudo unshare --uts sh
# hostname
ddg52
# hostname abc
### 隔離的程序hostname改為abc
# hostname
abc            
# exit
bigred@ddg52:~$ hostname
ddg52
```


> ps aux 所有資訊 來自 這個/proc資料夾 動態記憶體資料夾
![](https://i.imgur.com/Ucja9a0.png)
雖然啟用 PID Namespace, ps 命令還是會看到 Host 的所有 Process 資訊, 這是因爲 ps 命令會讀取 ddg52 的 /proc 目錄資訊
```gherkin=
mount 把新的資料夾 蓋到 原資料夾上(原資料夾還在)

sudo unshare --pid --fork --mount-proc sh
ps aux


sudo unshare --pid --fork --mount-proc sh
# ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.0   2608   608 pts/0    S    00:30   0:00 sh
root           2  0.0  0.1  10600  3304 pts/0    R+   00:30   0:00 ps aux

# sleep 60 &
# pstree -p
sh(1)─┬─pstree(4)
      └─sleep(3)

--mount-proc 這參數, 會將 sh 的 PID Namespace, 掛載到 ddg52 的 /proc 目錄, 由以下命令, 得知 /proc 目錄被掛載二次
# mount | grep -e "^proc"
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)

執行以下命令, 會自動卸載 /proc 目錄的第二次掛載
# exit
```



brctl show 
(bridge contral)

#### 產生橋接器
> sudo brctl addbr kvmbr29


### Network Namespace

![](https://i.imgur.com/yxFxhJ6.png)


> ifconfig 只可以看到 default network namespace，
ip 命令可以看到所有 network space (new network namespace 和 default network space)。
中間那條線就是乙太網路(硬體規格)

### Network Namespace 實作
```gherkin=
在 ddg52 終端機執行 unshare 命令
$ sudo unshare --pid --fork --mount-proc --net -R rootfs sh


在另一個終端機, 再次登入 ddg52 終端機
$ ssh 172.29.0.52
bigred@172.29.0.52's password: bigred

建立二片虛擬網卡
$ sudo ip link add v1 type veth peer name v2

$ ifconfig -a

啟用 v1 網卡, 並設定 IP 位址
$ sudo ip link set v1 up

$ sudo ip addr add 192.168.1.100/24 dev v1

取得 unshare process 的 Network Namespace ID
$ ps -ef | grep -E "^root.* sh$"
root        4214     467  0 02:50 ttyS0    00:00:00 sudo unshare --pid --fork --mount-proc --net -R rootfs sh
root        4215    4214  0 02:50 ttyS0    00:00:00 unshare --pid --fork --mount-proc --net -R rootfs sh
root        4216    4215  0 02:50 ttyS0    00:00:00 sh

設定 v2 網卡至 unshare process 的 Network Namespace
$ sudo ip link set v2 netns 4216

回到 unshare process 終端機
# ip a s
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
3: v2@if4: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether d6:4c:bf:e5:66:67 brd ff:ff:ff:ff:ff:ff link-netnsid 0

# ip addr add 192.168.1.200/24 dev v2

# ip link set v2 up

# ping -c 2 192.168.1.100
PING 192.168.1.100 (192.168.1.100): 56 data bytes
64 bytes from 192.168.1.100: seq=0 ttl=64 time=0.360 ms
64 bytes from 192.168.1.100: seq=1 ttl=64 time=0.086 ms
# exit

務必將 ddg52 虛擬主機重新開機
$ sudo reboot
```





#### busy box 抓取網頁

```gherkin=
在 cvn71 終端機執行以下命令 
$ cd cnt

$ dkh start ddg52

$ ssh 172.29.0.52
bigred@172.29.0.52's password: bigred
........

檢視 busybox 是否提供 httpd 功能 (grep –o 只找到 httpd 這幾個字其他不秀出來)

$ busybox | grep -o httpd   
Httpd 


製作首頁
$ mkdir -p www/cgi-bin

$ echo '<h1>MyWeb : /x/x/x <h1>'   >  www/index.html

啟動 HTTP Daemon
$ busybox httpd -p 8888 -h ~/www

$ curl http://localhost:8888
<h1>MyWeb 1.6<h1>


關閉 HTTP Daemon
#pgrep 是找 busybox 的 Pid
$ ps aux | pgrep busybox    
2227

$ kill -9 2227
```


#### 設計CGI程式
> 網站程式設計,網頁設計
```gherkin=
$ nano  www/cgi-bin/kungfu
#!/bin/sh
echo "Content-type: text/html; charset=utf-8"
echo ""
parm=$(echo $QUERY_STRING | tr '&' ' ')
echo $parm
echo ""

$ chmod +x  www/cgi-bin/kungfu

啟動 HTTP Daemon
$ busybox httpd -p 8888 -h ~/www

$ curl 'http://localhost:8888/cgi-bin/kungfu?name=car&size=small'
name=car size=small


關閉 HTTP Daemon
$ ps aux | pgrep busybox
2227

$ kill -9 2227
```


## Docker

![](https://i.imgur.com/89rhQwu.png)

> 在這邊我們的 Host os 就是 ddg52 os (kernel)
> 
> Bins & Libs 就是 docker image
> 
> 圖中的docker 是指 docker daemon (docker背景程式)
>
> docker 開機速度神速，因為不用開 kernel (kernel 已經開好了,DDG52的OS)
> 

### Docker container 運作原理
![](https://i.imgur.com/I7DrucN.png)

> runc 負責產生 container
> 
> shim 負責監控 container
> 
> shim 負責跟 containerd 匯報
>
>docker engine 則會詢問 containerd 的狀況

#dockerless
#podman (不需要docker daemon)

### 安裝 runc ，執行 runc

```gherkin=
$ ssh 172.29.0.52
bigred@172.29.0.52's password: bigred 

在 Ubuntu 20.04 系統中, 可直接安裝, 命令如下 :
$ sudo apt update; sudo apt install runc

$ runc  -v
runc version spec: 1.0.1-dev

因 runc 命令內定執行帳號為 root, 而 rootfs 目錄的 Owner 是 bigred, 所以需將 rootfs 目錄的 owner 改為 root
$  sudo chown root:root -R rootfs/
```
#### 產生 OCI 標準的 Container 設定檔

```gherkin=
$ runc spec

$ nano config.json 
{
	"ociVersion": "1.0.1",
	"process": {
		"terminal": true,
		"user": {
			"uid": 0,
			"gid": 0
		},
		"args": [
			"sh"
		],
.....
	"root": {
		"path": "rootfs",
		"readonly": false
	},
.....
```
#### 建立 bb8 Container (啟用 Namespace)

```gherkin=
$ sudo runc run bb8
/ # hostname
runc
/ # ip a s
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
/ # ps aux 
PID   USER     TIME  COMMAND
    1 root      0:00 sh
    8 root      0:00 ps aux
/ # ls -al | head -n 5
total 64
drwxrwxr-x   19 1000     1000          4096 May 31 04:10 .
drwxrwxr-x   19 1000     1000          4096 May 31 04:10 ..
```

#### 保護 Linux Kernel 系統目錄
```gherkin=
/# mkdir  /zzz

/# cat /proc/meminfo | head -n 3
MemTotal:        4030620 kB
MemFree:         3442880 kB
MemAvailable:    3653476 kB


Linux kernel 的目錄及檔案會受到保護
/ # rm /proc/meminfo
rm: remove '/proc/meminfo'? y
rm: can't remove '/proc/meminfo': Permission denied

關閉 bb8 Container
$ exit
```
### 探索 containerd

![](https://i.imgur.com/rHNro2I.png)

> containerd 的工作包含拉下 image 並解壓縮檔案並產生 rootfs 及container.json 


#### 安裝 Containerd
```gherkin=
$ sudo apt install containerd

在系統會多出 containerd 這個 daemon
$ ps aux | grep -v grep | grep containerd
root        1073  0.4  1.1 895428 47224 ?        Ssl  00:42   0:00 /usr/bin/containerd

$ containerd -v
containerd github.com/containerd/containerd 1.3.3-0ubuntu2
```

#### Containerd 原生管理命令

```gherkin=
下載 Container Image
$ sudo ctr image pull docker.io/library/busybox:latest

$ sudo ctr image ls -q
docker.io/library/busybox:latest

$ sudo ctr run --tty docker.io/library/busybox:latest b1 sh
/ # ps aux
PID   USER     TIME  COMMAND
    1 root      0:00 sh
    8 root      0:00 ps aux
/ # exit

$ sudo ctr container list
CONTAINER    IMAGE                                           RUNTIME                  
b1                 docker.io/library/busybox:latest        io.containerd.runc.v2 

$ sudo ctr container rm b1

$ sudo ctr image remove docker.io/library/busybox:latest docker.io/library/busybox:latest
```


### 探索 Docker
```gherkin=
開始安裝 Docker ， (如果已經裝過 runc , containerd , 就不會再安裝一次)
$ sudo  apt  install  docker.io

將 bigred 帳號加入 docker 群組後, 就不需使用 sudo 命令執行 docker
$ sudo  usermod  -aG  docker bigred

重新開機 (一定要執行)
$ sudo  reboot

再次登入 ddg52
$ ssh 172.29.0.52
bigred@172.29.0.52's password: bigred

顯示 Docker 版本
$ docker info
 .....
 Server Version: 19.03.8
 Storage Driver: overlay2
 ......
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 ......
```
#### 檢視 Docker 運作資訊

```gherkin=
沒有任何 Container 執行時, 只有 dockerd 及 containerd 這二個 Daemon 在運作
$ ps aux | grep -v grep | grep containerd
root         604  0.3  1.1 969256 46788 ?        Ssl  14:48   0:06 /usr/bin/containerd
root        1034  0.1  2.2 1008892 89944 ?       Ssl  14:52   0:03 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
```

![](https://i.imgur.com/Z5oN1Z8.png)
- my computer : ddg52
- 儲存在 /var/lib/docker
```gherkin=
在 ddg52 終端機執行以下命令 
$ docker search busybox
NAME                            DESCRIPTION                   STARS     OFFICIAL  AUTOMATED
busybox                        Busybox base image.        1912         [OK]       
progrium/busybox                                                58            [OK]
.............


$ docker pull busybox
Using default tag: latest
latest: Pulling from library/busybox
7520415ce762: Pull complete 
Digest: sha256:32f093055929dbc23dec4d03e09dfe971f5973a9ca5cf059cbfb644c206aa83f
Status: Downloaded newer image for busybox:latest

$ docker images
REPOSITORY   TAG        IMAGE ID      CREATED          SIZE
busybox      latest     1c35c4412082  5 days ago       1.22MB
```
#### 建立與執行軟體貨櫃 (Container) 主機
```gherkin=
建立軟體貨櫃 (再次確認 Linux Namespace 啟動)
$ docker run --name b1 -it busybox /bin/sh
/ # uname -r
5.4.0-33-generic

/ # hostname
54ab5a1a4690

/ # ps aux
PID   USER     TIME   COMMAND
    1 root       0:00 /bin/sh
   10 root       0:00 ps aux

/ #  ifconfig eth0
eth0      Link encap:Ethernet  HWaddr 02:42:AC:11:00:02  
          inet addr:172.17.0.2  Bcast:0.0.0.0  Mask:255.255.0.0
          inet6 addr: fe80::42:acff:fe11:2/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
         .........

/ # exit
```

#### 管理軟體貨櫃 (Container) 主機
```gherkin=
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
54ab5a1a4690        busybox             "/bin/sh"           5 minutes ago       Exited (0) 9 seconds ago                       b1

$ docker start b1

$ docker exec -it b1 sh      (只能本機連接)
/ # exit

$ docker stop b1

$ docker rm b1
```


### docker 使用 busybox 開網站練習


```gherkin=
#給 ddg52 開 80 port 連到 8888
docker run --name b2 -it -p 80:8888 busybox /bin/sh

mkdir -p www/cgi-bin

echo '<h1>MyWeb : /x/x/x <h1>'   >  www/index.html

vi  www/cgi-bin/kungfu

chmod +x  www/cgi-bin/kungfu

busybox httpd -p 8888 -h /www
```

> 之後用cvn71的firefox連到網路
![](https://i.imgur.com/mHTAEKv.png)

### 建立 Alpine 軟體貨櫃主機 並測試上傳到 docker hub
```gherkin=
$ docker run --name a1 -it alpine sh
/ # busybox | grep httpd

[重要] Docker Alpine Image 內建的 busybox 並沒提供 httpd 功能

下載 Busybox 執行檔
/ #  wget https://busybox.net/downloads/binaries/1.28.1-defconfig-multiarch/busybox-x86_64

安裝 Busybox 執行檔
/ #  chmod +x busybox-x86_64
/ #  mv busybox-x86_64 bin/busybox

執行 Busybox 命令
/ # busybox | grep httpd
	hexedit, hostid, hostname, httpd, hush, hwclock, i2cdetect, i2cdump, i2cget, i2cset, id, ifconfig,

```
#### 建立 Busybox Httpd 網站伺服器
```gherkin=
製作網站目錄及首頁
/ # mkdir www
/ # echo '<h1>Busybox HTTPd</h1>' > www/index.html

啟動 Busybox Httpd 網站伺服器
/ # busybox httpd -p 8888 -h www

安裝網頁工具
/ # apk update
/ # apk add elinks curl

取得網頁
/ # curl http://localhost:8888
<h1>Busybox HTTPd</h1>

檢視網頁
/ # elinks -dump 1 http://localhost:8888
                                   Busybox HTTPd

/ # exit
```

#### 製作 a1 image 並上載至 Docker HUB
```gherkin=
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
559336d9d286        alpine              "sh"                9 minutes ago       Exited (0) 2 minutes ago                       a1

$ docker commit a1 <Login Name>/a1
sha256:8684b18333c9b1bf1a87ff008819d554459d752739e4c312c515b5031285ef2a

$ docker login 
.........
Username: <Login Name>
Password: 
Login Succeeded

$ docker push <Login Name>/a1
$ docker logout

[重要] 關閉 Docker Host 之前, 記得執行 docker logout, 然後刪除 /home/bigred/.docker 目錄

$ docker rm a1 
```
#### 使用自製 a1 image
```gherkin=
$ docker run --name testa1 -d <login name>/a1 httpd -p 8888 -h www 

$ docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                     PORTS               NAMES
aa9b759e6f0e        dafu/busyweb        "httpd -p 8888 -h www"   6 seconds ago       Exited (0) 4 seconds ago                       testa1

因 httpd 沒在前景啟動, 以致 testa1 貨櫃主機沒啟動
$ docker rm testa1

重新建立 testweb 貨櫃主機
$ docker run --name testa1 -d <login name>/a1 httpd -f -p 8888 -h www

$ docker exec testa1 hostname -i
172.17.0.2

$ curl http://172.17.0.2:8888
<h1>Busybox HTTPd</h1>
```








### 建立 Nginx 網路
```gherkin=
# start=always 會下次開機自動重新啟動
$ docker run -d -p 7777:80 --restart=always --name n1 nginx

$ docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
5e45f40f48d1        nginx               "/docker-entrypoint.…"   5 seconds ago       Up 4 seconds        0.0.0.0:7777->80/tcp   n1


$ curl -s http://localhost:7777 | head -n 10
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }

$ exit

```
#### 重建 n1 image 並設定自動啟動

```bash =
在 CVN71 終端機執行以下命令 
$ dkh stop ddg52

$ dkh start ddg52

[註] Docker Host 重啟內定不會自動啟動 Container, 除非指定 --restart=always 這參數, Container 一但設定自動重啟, 並不會重新建立一個新的 Container, 由以下 Container ID 可以得知

在 ddg52 終端機執行以下命令 
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
5e45f40f48d1        nginx               "/docker-entrypoint.…"   2 minutes ago       Up About a minute   0.0.0.0:7777->80/tcp   n1

$ docker rm -f n1
```
### 撰寫 Dockerfile

```bash =

在 ddg52 終端機執行以下命令 
$ mkdir wulin; cd wulin

$ wget https://julialang-s3.julialang.org/bin/linux/x64/1.4/julia-1.4.1-linux-x86_64.tar.gz

[註] 下載時間大約 5 分鐘

nano Dockerfile

開始撰寫 Dockerfile 

FROM ubuntu:18.04
RUN apt-get update && apt-get install -y wget nano 
COPY julia-1.4.1-linux-x86_64.tar.gz /tmp
RUN tar xvfz /tmp/julia-1.4.1-linux-x86_64.tar.gz -C /opt
RUN rm /tmp/julia-1.4.1-linux-x86_64.tar.gz

#把 julia 執行命令，放入Path(usr/bin/ 一定    存在Path)
RUN ln -s /opt/julia-1.4.1/bin/julia /usr/bin/julia' > Dockerfile


# no cache 是不要把 ubuntu下載檔案存到快取(可以避免下次下載到快取檔案)

$ docker build --no-cache -t julia  .
```
> 註:
> 
> dockerfile的命令都要大寫
> 
> 命令都不是在ddg52目錄下操作
>
> dockerfile 中 run 越少越少(產生的唯獨暫存目錄越少)
> 
> 如果有壓縮檔案的話要使用ADD(不要用copy)

#### 檢視 julia image
```bash =
$ docker history julia
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
dd67d5e9f03b        9 seconds ago       /bin/sh -c ln -s /opt/julia-1.1.0/bin/julia …   26B                 
222ddc7d59f0        10 seconds ago      /bin/sh -c rm /tmp/julia-1.1.0-linux-x86_64.…   0B                  
51e4e25d188f        12 seconds ago      /bin/sh -c tar xvfz /tmp/julia-1.1.0-linux-x…  328MB               
1ed835d0130b        22 seconds ago      /bin/sh -c #(nop) COPY file:2c9e61d7a9dd148e…   98.9MB              
c5f89f082101        24 seconds ago      /bin/sh -c apt-get update && apt-get install…   31.9MB              
94e814e2efa8        12 days ago         /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B                  
<missing>           12 days ago         /bin/sh -c mkdir -p /run/systemd && echo 'do…   7B                  
<missing>           12 days ago         /bin/sh -c rm -rf /var/lib/apt/lists/*          0B                  
<missing>           12 days ago         /bin/sh -c set -xe   && echo '#!/bin/sh' > /…   745B                
<missing>           12 days ago         /bin/sh -c #(nop) ADD file:1d7cb45c4e196a6a8…   88.9MB   
            
$ docker images julia
REPOSITORY      TAG          IMAGE ID               CREATED             SIZE
julia                   latest       23103df60837        8 minutes ago       566MB

```

#### 測試 julia image
```bash =
$ docker run --rm -it julia sh
# julia	
               _
   _       _ _(_)_     |  Documentation: https://docs.julialang.org
  (_)     | (_) (_)    |
   _ _   _| |_  __ _   |  Type "?" for help, "]?" for Pkg help.
  | | | | | | |/ _` |  |
  | | |_| | | | (_| |  |  Version 1.4.1 (2018-08-08)
 _/ |\__'_|_|_|\__'_|  |  Official https://julialang.org/ release
|__/                   |

julia> CTRL+D

# exit
```
### julia image layer 最佳化 
```bash =
$ nano Dockerfile
FROM ubuntu:18.04
COPY julia-1.4.1-linux-x86_64.tar.gz /tmp
RUN apt-get update && apt-get install -y wget nano && \ 
    tar xvfz /tmp/julia-1.4.1-linux-x86_64.tar.gz -C /opt && \
    rm /tmp/julia-1.4.1-linux-x86_64.tar.gz && \
    ln -s /opt/julia-1.4.1/bin/julia /usr/bin/julia

$ docker rmi julia 
$ docker build --no-cache -t julia  .


$ docker history julia
IMAGE               CREATED              CREATED BY                                      SIZE                COMMENT
0c422c42d2f7    About a minute ago   /bin/sh -c apt-get update && apt-get install…   357MB               
c241d4f93a17    About a minute ago   /bin/sh -c #(nop) COPY file:9c0f24a51dc54d99…   98.9MB              
...........
```
#### 再次重建 julia image
```bash =
$ echo 'FROM ubuntu:18.04
ADD julia-1.4.1-linux-x86_64.tar.gz /opt
RUN ln -s /opt/julia-1.4.1/bin/julia /usr/bin/julia' > Dockerfile

[註] the best use for ADD is local tar file auto-extraction into the image

$ docker rmi julia; docker build --no-cache -t julia  .

$ docker images julia
REPOSITORY  TAG                 IMAGE ID              CREATED              SIZE
julia               latest              d4c87fb8bde0        9 seconds ago       430MB
```



### Golang Application Image
#### 開發 Golang 網站
```bash =
cd ~/wulin; mkdir mygo; cd mygo

$ echo 'package main
import (
	"fmt"
	"log"
	"net/http"
)
func handler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintln(w, "Hello, 世界")
}
func main() {
	http.HandleFunc("/", handler)
	log.Fatal(http.ListenAndServe(":8888", nil))
}' > main.go


$ CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o main
```

#### 自製原生 Docker Image

```bash =
$ dir
總計 6.3M
drwxr-xr-x 2 bigred bigred 4.0K  6月  8 19:38 .
drwxr-xr-x 3 bigred bigred 4.0K  6月  8 19:36 ..
-rwxr-xr-x 1 bigred bigred 6.3M  6月  8 19:38 main
-rw-r--r-- 1 bigred bigred  236  6月  8 19:37 main.go

$ echo 'FROM scratch
ADD main /
CMD ["/main"] ' > Dockerfile

$ docker build -t goweb .

$ docker images
REPOSITORY    TAG         IMAGE ID            CREATED             SIZE
goweb             latest       7f9652539fc0      14 minutes ago     7.39MB
```
#### 建立 g1 container
```bash =
$ docker run --rm --name g1 -d -p 88:8888 goweb

$ curl http://localhost:8888
Hello, 世界

$ docker stop g1
```
### Docker Image 內定的執行命令
```bash =
$  docker run --rm -it alpine
/ # ps
PID   USER     TIME  COMMAND
    1 root      0:00 /bin/sh
    7 root      0:00 ps
/ # exit

$ docker run --rm -it busybox
/ # exit

問題 : 上述二個命令中, 不需指定執行命令, 一樣可以啟動貨櫃主機, 請問內定執行命令為何 ?

使用 docker history 命令得知 Docker Image 內定執行命令
$ docker history alpine
IMAGE         CREATED       CREATED BY                           SIZE
3fd9065eaf02  4 months ago  /bin/sh -c #(nop) CMD ["/bin/sh"]   0B                  
<missing>     4 months ago  /bin/sh -c #(nop) ADD file:093f0…    4.15MB              

$ docker history busybox
IMAGE         CREATED        CREATED BY                                   SIZE  
8ac48589692a  6 weeks ago    /bin/sh -c #(nop) CMD ["sh"]                   0B                  
<missing>     6 weeks ago    /bin/sh -c #(nop) ADD file:c94ab8f8614 …   1.15MB 

```
```bash =
$ cd ~/wulin; mkdir base

$ echo 'FROM alpine:3.11.6
RUN apk update && apk upgrade && apk add --no-cache nano sudo wget curl \
    tree elinks bash shadow procps util-linux coreutils binutils findutils grep && \
    wget https://busybox.net/downloads/binaries/1.28.1-defconfig-multiarch/busybox-x86_64 && \
    chmod +x busybox-x86_64 && mv busybox-x86_64 bin/busybox1.28 && \
    mkdir -p /opt/www && echo "let me go" > /opt/www/index.html

CMD ["/bin/bash"] ' > base/Dockerfile

```


### 自製 Alpine OpenSSH Server 的 Docker Image


```bash =
cd ~/wulin; mkdir plus
$ nano plus/Dockerfile 
FROM alpine.base
RUN apk update && \
    apk add --no-cache openssh-server tzdata && \
    # 設定時區
    cp /usr/share/zoneinfo/Asia/Taipei /etc/localtime && \
    ssh-keygen -t rsa -P "" -f /etc/ssh/ssh_host_rsa_key && \
    echo -e 'Welcome to Alpine 3.11.6\n' > /etc/motd && \ 
    # 建立管理者帳號 bigred   
    adduser -s /bin/bash -h /home/bigred -G wheel -D bigred && echo '%wheel ALL=(ALL) NOPASSWD: ALL' >> /etc/sudoers && \
    echo -e "bigred\nbigred\n" | passwd bigred &>/dev/null && [ "$?" == "0" ] && echo "bigred ok"
 
EXPOSE 22
 
ENTRYPOINT ["/usr/sbin/sshd"]
CMD ["-D"]
```
```bash =
$ docker build --no-cache  -t alpine.plus plus/
$ docker run --rm --name s1 -d -h s1 alpine.plus sh
Extra argument sh.
[重要] 因使用 entrypoint 宣告內定命令, 便無法自行指定執行命令

$ docker run --rm --name s1 -d -h s1 alpine.plus 
$ ssh `docker exec s1 hostname -i`
```


### Docker Container 備份與還原
```bash =
$ docker run --name s2 -h s2 -d alpine.plus
$ docker export s2 > dkssh.tar
$ cat dkssh.tar | docker import - alpine.backup
$ docker history alpine.backup

上述做法會有問題

$ docker run --name s2 -h s2 -d alpine.plus
6e52bd5aee18c0aed8dbb17920037be0b9109158329b401c56b4f64417c2d784

[重要] 使用上述命令建立 s2 Container, 這個 Container 有啟動 openssh server, 所以這個 Container 有 openssh 的執行狀態資訊檔, 這時使用 docker export 命令 
匯出的 Tar 檔中就會有殘留 openssh 的執行暫存檔, 以至後續做出的 image 無法啟動 openssh server, 所以必須先關閉 s2 container, 才可備份此 container

$ docker run --name s2 -h s2 -d alpine.plus
$ docker stop s2             
$ docker export s2 > s2.tar

$ docker rm s2 && docker rmi alpine.plus
$ cat s2.tar | docker import - alpine.plus &>/dev/null

$ docker history alpine.plus
```
#### 使用重製 alpine.plus image
```bash =
$ docker run --name s2 -h s2 -d alpine.plus
docker: Error response from daemon: No command specified.
See 'docker run --help'.

[重要] 重製後的 alpine.plus image 必須指定 執行命令

$ docker run --name s2 -h s2 -d alpine.plus /usr/sbin/sshd -D

$ docker ps -a
CONTAINER ID        IMAGE               COMMAND               CREATED             STATUS              PORTS               NAMES
d4f93f15556c        alpine.plus         "/usr/sbin/sshd -D"   5 seconds ago       Up 3 seconds                            s2

$ docker rm -f s2
```





### student_server
```bash =

FROM ubuntu:20.04
RUN \
        sed -i 's/archive.ubuntu.com/free.nchc.org.tw/g' /etc/apt/sources.list && \
        apt update && DEBIAN_FRONTEND=noninteractive \
        apt install -y ssh nano wget curl sudo
RUN for myarg in student teacher ; do \
              useradd -m -s /bin/bash ${myarg} && \
              usermod -a -G adm,sudo ${myarg} && \
              echo "${myarg}:${myarg}" | chpasswd -m && \
              echo '%sudo ALL=(ALL) NOPASSWD: ALL' >> /etc/sudoers ; done
WORKDIR /home/student

USER student
RUN \
  ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa && \
  cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys

WORKDIR /home/teacher
USER teacher
RUN \
  ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa && \
  cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
USER root
RUN \
  echo 'sudo service ssh start' >> /etc/bash.bashrc
USER student

WORKDIR /home/student
CMD ["/bin/bash"]
```


### 網路
```
docker run --rm busybox cat /etc/resolv.conf
```
沒給dns的話 google 預設給 8.8.8.8