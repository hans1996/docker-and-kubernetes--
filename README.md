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
```gherkin=

```



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

