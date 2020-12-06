---
title: 'SSH'
disqus: hackmd

---

---

SSH 
===

[TOC]


---
---

```gherkin=
# ssh  
ssh 帳號@ip
```


產生SSH公鑰、與私鑰
---
```gherkin=
# 設定
ssh-keygen -t rsa -P ''
```
路徑下產生 id_rsa , id_rsa.pub
![](https://i.imgur.com/AUDQ6wp.png)


複製本帳號公鑰到指定機器(host)的某帳號(name)
---

```gherkin=
# 設定
ssh-copy-id name@host
```

### 切換到指定機器，產生檔案 ~/.ssh/authorized_keys
