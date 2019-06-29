---
title: Linux SSH 免密登陆
---


```
### 生成 公钥与密钥
[huangkai@localhost ~]$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/huangkai/.ssh/id_rsa):  回车
Created directory '/home/huangkai/.ssh'. 
Enter passphrase (empty for no passphrase):   回车
Enter same passphrase again:  回车
Your identification has been saved in /home/huangkai/.ssh/id_rsa. 
Your public key has been saved in /home/huangkai/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:C3VMLg1yw3XndMjouLyESTeGsDwBaFe2lFsZW3j6vmA huangkai@localhost.localdomain
The key's randomart image is:
+---[RSA 2048]----+
|   ..o=o==+ .oo..|
|  o .o++=X...+o. |
| . . ..*++=o  .  |
|      =.+o= .    |
|      .oSB o     |
|       .o.=      |
|        Eo .     |
|       . .o      |
|          ..     |
+----[SHA256]-----+
[huangkai@localhost ~]$ ll
total 0
[huangkai@localhost ~]$ ls -la
total 12
drwx------. 3 huangkai huangkai  74 May 24 19:24 .
drwxr-xr-x. 3 root     root      22 May 24 09:40 ..
-rw-r--r--. 1 huangkai huangkai  18 Oct 31  2018 .bash_logout
-rw-r--r--. 1 huangkai huangkai 193 Oct 31  2018 .bash_profile
-rw-r--r--. 1 huangkai huangkai 231 Oct 31  2018 .bashrc
drwx------. 2 huangkai huangkai  38 May 24 19:24 .ssh
[huangkai@localhost ~]$ cd .ssh/
[huangkai@localhost .ssh]$ ll
total 8
-rw-------. 1 huangkai huangkai 1679 May 24 19:24 id_rsa
-rw-r--r--. 1 huangkai huangkai  412 May 24 19:24 id_rsa.pub
[huangkai@localhost .ssh]$ ls -la
total 8
drwx------. 2 huangkai huangkai   38 May 24 19:24 .
drwx------. 3 huangkai huangkai   74 May 24 19:24 ..
-rw-------. 1 huangkai huangkai 1679 May 24 19:24 id_rsa
-rw-r--r--. 1 huangkai huangkai  412 May 24 19:24 id_rsa.pub
[huangkai@localhost .ssh]$ ssh-copy-id -i ./id_rsa.pub 192.168.117.101
/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "./id_rsa.pub"
The authenticity of host '192.168.117.101 (192.168.117.101)' can't be established.
ECDSA key fingerprint is SHA256:f2azvRSutV5arHrwEdir2iHn/kpRZ729PxLaBZnPnHc.
ECDSA key fingerprint is MD5:d3:5b:4b:bf:07:94:c8:5e:40:ea:6d:a8:36:cb:f2:54.
Are you sure you want to continue connecting (yes/no)? yes
/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
huangkai@192.168.117.101's password:   #### 输入 192.168.117.101 的 huangkai 账号的密码

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh '192.168.117.101'"
and check to make sure that only the key(s) you wanted were added.

[huangkai@localhost .ssh]$ cd ..
[huangkai@localhost ~]$ ssh 192.168.117.101 ##### 使用 ssh 连接
Last login: Fri May 24 19:24:40 2019
[huangkai@localhost ~]$ ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:46:62:9c brd ff:ff:ff:ff:ff:ff
    inet 192.168.117.101/24 brd 192.168.117.255 scope global noprefixroute ens33
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe46:629c/64 scope link 
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:d9:a3:82:77 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
[huangkai@localhost ~]$ clear
```