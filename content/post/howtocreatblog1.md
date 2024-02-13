---

title:      "保姆级免费建博客教程①"
date:       2021-11-11
author:     "Rezzo"
categories: [ "Tips" ] 
tags: ["Tutorials", "Blog"]
---
# Part One

# 免费制作自己的博客！手把手保姆级教学！
[第二部分 点击跳转](/post/howtocreatblog2)

## 第一步：申请一个GitHub账号

### 够宝宝吧，从申请账号开始
- - -
#### 点击[GitHub](https://github.com/) ： 点击右上角 sign up（注册）, 输入邮箱，密码，昵称，完成后面的验证，然后一些小问题都不怎么重要，随便看着选一下就行了，看不懂可以机翻看一下
![注册](/img/how-to-build-blog/signup.png)
- - -
#### 然后右上角点击"Your repositories"
![repositories](/img/how-to-build-blog/repositories.png)
- - -
#### 然后点击那个绿色的new ,新建一个仓库
![new](/img/how-to-build-blog/new.png)
- - -
#### Repository name 必须填自己的 ID名称 + .github.io。这个仓库里放的东西，就是你的博客网页文件
![create-githubio](/img/how-to-build-blog/create-githubio.png)
- - -
#### 建好以后，还什么都没有
![after-create-githubio](/img/how-to-build-blog/after-create-githubio.png)
- - -
## 第二步：前期准备

#### 我们需要下载一个[git](https://git-scm.com/)，本地进行一些操作
![download-git](/img/how-to-build-blog/download-git.png)
- - -
#### 安装git，基本上无脑下一步，这有个选择默认编辑器的，我用的是[Notepade++](https://notepad-plus.en.softonic.com/)，可以选自己熟悉的用
![install-git](/img/how-to-build-blog/install-git.png)
- - -
#### 安装[hugo](https://gohugo.io/)，这个就是将你的网站本地生成好的东西，然后把生成好的文件上传到上面那个仓库里，就直接能显示了
#### 如果你在使用 Windows，我推荐使用 [Scoop](https://scoop.sh/) 来安装 Hugo：
![powershell](/img/how-to-build-blog/powershell.png)
在powershell里输入下面的命令
安装scoop
```
	Invoke-Expression (New-Object System.Net.WebClient).DownloadString('https://get.scoop.sh')
	# 或者更短的命令
	iwr -useb get.scoop.sh | iex
```
Note: if you get an error you might need to change the execution policy (i.e. enable Powershell) with
如果遇到error或者need to change the execution policy
输入
```
	Set-ExecutionPolicy RemoteSigned -scope CurrentUser
```
安装hugo
```
	scoop install hugo-extended
```
- - -
#### 再新建一个叫blog的仓库，用来存hugo的网站文件
![blognew](/img/how-to-build-blog/blognew.png)
#### [git ssh配置](/post/gitssh)，方便以后的下载上传
#### 找一个位置
![workspace](/img/how-to-build-blog/workspace.png)
- - -
#### 将两个仓库拉到本地
![gitlink](/img/how-to-build-blog/gitlink.png)
clone 到本地，两个连接均可，网络不好就用ssh，网络状态好可以用https的
```
	git clone git@github.com:ReggieRezzo/ReggieRezzo.github.io.git
```
![clone](/img/how-to-build-blog/clone.png)
下载好后，显示下载好的文件夹
```
	ls
```
进入ReggieRezzo.github.io文件夹
```
	cd ReggieRezzo.github.io
```
创建README文件
```
	touch README.md
```
添加全部文件
```
	git add .
```
填写说明
```
	git commit -m "Init commit"
```
上传到GitHub
```
	git push origin main
```
- - -
初始化完成，blog仓库同理

相当于你复制了一份GitHub仓库到本地，然后在本地做了修改，将修改的内容再上传给GitHub

你本地是一个离线的GitHub仓库，只有你push的时候才和GitHub在线仓库同步
![readme](/img/how-to-build-blog/readme.png)

### 可能的麻烦
最大的麻烦就是GitHub的网络问题了，可以试着Git 挂代理，科学上网，改DNS等方式百度解决
