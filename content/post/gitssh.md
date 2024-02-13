---

title:      "Git SSH 配置"
date:       2021-11-11
author:     "Rezzo"
categories: [ "Tips" ] 
tags: ["Tutorials", "SSH"]

---

#### 1.查看是否配置过密匙
```
	cd ~/.ssh
```
不加cd，就是看存不存在 .ssh 文件夹
加cd，如果存在这个文件夹，就直接跳转到这个文件夹
如果不存在这个文件夹，会显示 No such file or directory
![Hasssh](/img/gitssh/Hasssh.png)
#### 2.创建ssh
输入如下命令，将引号内的邮箱换成你的GitHub注册邮箱
```
	ssh-keygen -t rsa -C 'youremail@qq.com'
```
然后无脑回车，完成后出现
![sshkey](/img/gitssh/sshkey.png)
#### 3.查看公匙
```
	cat ~/.ssh/id_rsa.pub
```
输入上面的命令，得到如图所示字符串
![pub](/img/gitssh/pub.png)
或者看上面保存的地址，打开id_rsa.pub 文件复制文件内容
![keypos](/img/gitssh/keypos.png)
#### 4.添加 SSH key
![gitssh1](/img/gitssh/gitssh1.png)
Title 可以随意填
![gitssh2](/img/gitssh/gitssh2.png)
#### 5.验证是否成功
输入下面的命令
```
	ssh -T git@github.com
```
出现下图所示提示就成功了
![success](/img/gitssh/success.png)