---
title: Window 、 Linux 下 Git - 生成 SSH 公钥
tags:
      - SSH
      - Git
description: 先查看是否已经生成好SSH公钥
---

### 先查看是否已经生成好SSH公钥

1.  window 为 C:\Users\linyaqin\.ssh  

     linux/mac 为 ~/.ssh
     
     文件名为 id_rsa （私钥）,  id_rsa.pub （公钥）

2.  配置github账号信息 
      ``` bash
      $ git config --glogbal user.email "lincoln@gmail.com"
      $ git config --glogbal user.name "lincoln"
      
      ```
     
3.  如果文件、目录不存在，则生成:

     linux 直接使用 ssh-keygen 
     
     window 使用 gitBash.ext 下面的   ssh-keygen 
     
  ``` bash
  $ ssh-keygen -t rsa -C "lincoln@gmail.com"
  
  ```
   此时生成 文件名为 id_rsa （私钥）,  id_rsa.pub （公钥）
  
4, 登录 github,把 id_rsa.pub 添加到SSH key

5, 验证 ： ssh -T git@github.com
    
  