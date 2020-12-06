---
title: '簡易使用者系統'
disqus: hackmd

---

---

簡易使用者系統
===

[TOC]


主程式建立
---
```gherkin=
#!/bin/bash

while :
do
clear

echo -e '
        2.create multiple account
        4.change single password
        5.change multiple password
        6.given multiple sudo authorization
        7.break
        '
read -p "chose 1~6 " number
case $number in

2)
    ./createMULTIaccount.sh
    ;;
4)
    ./changepasswd.sh
    ;;
5)
    ./changemultipasswd.sh
    ;;
6)
    ./addsudogroup.sh
    ;;
7)
    break
    ;;
8)
    echo "noob"
    ;;
esac

done
```

批次創建帳號and密碼
---
```gherkin=
#! /bin/bash
clear
read -p "enter Account" account
read -p "enter Start numer" startnum
read -p "enter End number" endnum

grep -iw "${account}" /etc/passwd &> /dev/null

if [ $? == 0 ]; then
    echo "Username exists"
else
    for ((i=$startnum; i<=$endnum; i++))
    do
        sudo useradd -m -s /bin/bash "${account}$i"
        echo "${account}${i}:${account}${i}" > accountNpasswd.txt
        sudo chpasswd < accountNpasswd.txt
    done
fi

read -p "enter any number"

```
更改單一密碼
---

```gherkin=
#! /bin/bash
clear
read -p "enter Account: " account

grep -i "${account}" /etc/passwd &> /dev/null

if [ $? == 0 ]; then
    read -p "enter new password: " password
    echo "${account}:${password}" | sudo chpasswd
else
    echo "Account is not founded: "
fi

read -p "enter any number: "
```


批次更改密碼
---

```gherkin=
#! /bin/bash
clear
read -p "enter Account: " account
read -p "enter start number " startnum
read -p "enter end number " endnum


grep -i "${account}" /etc/passwd &> /dev/null


if [ $? == 0 ]; then
    read -p "enter new password: " password
    for ((i=$startnum; i<=$endnum; i++))
    do
        echo "${account}${i}:${password}" > passWORD.txt
        sudo chpasswd < passWORD.txt
    done
else
    echo "account not found"
fi

read -p "enter any number"

```
批次給定sudo權限
---
```gherkin=
#! /bin/bash
clear
read -p "enter Account: " account
read -p "enter start number: " startnum
read -p "enter end number: " endnum


grep -i "${account}" /etc/passwd &> /dev/null

if [ $? == 0 ]; then
    grep "sudo" /etc/group | grep -w "${account}"
    if [ $? == 0 ]; then
        for ((i=$startnum; i<=$endnum; i++))
        do
            sudo usermod -aG sudo ${account}${i}
        done
    else
        echo "account already in sudo group"
    fi
else
    echo "account not founded"
fi

read -p "enter any number"
```