---

title:      "保姆级免费建博客教程②"
date:       2021-11-11
author:     "Rezzo"
categories: [ "Tips" ] 
tags: ["Tutorials", "Blog"]
---
# Part Two

# 免费制作自己的博客！手把手保姆级教学！
[第一部分 点击跳转](/post/howtocreatblog1)
- - - 
#### 前期准备工作做好以后，文件夹应该是这个状态
![workspace2.png](/img/how-to-build-blog/workspace2.png)
进入blog文件夹
![blog1.png](/img/how-to-build-blog/blog1.png)
- - - 
#### 右键，点击Git Bash Here，打开命令窗口
![gitbushhere.png](/img/how-to-build-blog/gitbushhere.png)
#### 输入如下命令，新建一个hugo site文件夹
```
	hugo new site ReggieRezzoBlog
	# 你可以把后面的文件夹名称换成你的
```
![newsite.png](/img/how-to-build-blog/newsite.png)
- - - 
#### 选择一个hugo 主题
![hugo.png](/img/how-to-build-blog/hugo.png)
![choose.png](/img/how-to-build-blog/choose.png)
![downloadtheme.png](/img/how-to-build-blog/downloadtheme.png)
跳转到主题的GitHub页面
- - -
#### 将这个项目clone到本地的themes文件夹
![clonetheme.png](/img/how-to-build-blog/clonetheme.png)
![downloadtheme2.png](/img/how-to-build-blog/downloadtheme2.png)
```
	git clone "你选的主题的链接"
```
![config.png](/img/how-to-build-blog/config.png)
将网址改成自己的github.io地址 如：https://ReggieRezzo.github.io/

标题随意，之后添加主题，名称就是上面下载的文件夹的名称
- - -
#### 本地运行
在/blog/ReggieRezzoBlog/文件夹下（能看到你刚才配置的文件），右键打开命令行
```
	hugo server
```
![hugoserver.png](/img/how-to-build-blog/hugoserver.png)
出现了错误

根据提示，我去主题作者的演示网站里，复制了一份他的配置（上面那几条用自己的）
![even.png](/img/how-to-build-blog/even.png)
OK,本地hugo服务器运行成功
![hugolocal.png](/img/how-to-build-blog/hugolocal.png)
打开浏览器在地址栏输入：
```
	127.0.0.1:1313
```
可以看到网站正确显示了
![hugosuccess.png](/img/how-to-build-blog/hugosuccess.png)
遇上错误，先去作者那看看文档，看看演示网站，一般都能解决，实在解决不了，就换一个主题吧
- - -
#### 添加文章
Ctrl + C 先停掉临时本地服务器

输入命令，生成文章，格式是md（MarkDown）格式
```
	hugo new post/POST_NAME.md
```
这个文件夹，有的是posts 有的 是post
![post.png](/img/how-to-build-blog/post.png)
文件名，标题这都可以自己决定，draft要么删掉要么改成 false，否则不会显示在网页上
![firstpost.png](/img/how-to-build-blog/firstpost.png)
点进去的效果：
![show.png](/img/how-to-build-blog/show.png)
- - - 
#### 同步到网络上显示
我们创建了两个仓库，ReggieRezzo.github.io 仓库，用来放hugo准备好，直接就能显示的网页

blog 用来放原始的文件数据

##### 将 ReggieRezzo.github.io 作为 blog 的子模块
```
	# 改成你对应的仓库地址
	git submodule add -b main git@github.com:ReggieRezzo/ReggieRezzo.github.io.git public
```
![submodule.png](/img/how-to-build-blog/submodule.png)
##### 生成静态网站代码
此时public文件夹还是空的
```
	# 引号内改成你的主题名称
	hugo -t "hugo-theme-even"
```
![staticpublic.png](/img/how-to-build-blog/staticpublic.png)
这表示网页生成成功了
![publicsucess.png](/img/how-to-build-blog/publicsucess.png)
##### 上传到GitHub
进入public文件夹
```
	git remote -v
```
![remote.png](/img/how-to-build-blog/remote.png)
这里显示的就不是blog的仓库地址了，而是作为子模块的仓库的地址

输入下面的命令
```
	git add .
	git commit -m "Init"
	git push origin main
```
成功上传后
![finally.png](/img/how-to-build-blog/finally.png)
![result1.png](/img/how-to-build-blog/result1.png)
![result2.png](/img/how-to-build-blog/result2.png)
- - - 
##### 显示成功
![result3.png](/img/how-to-build-blog/result3.png)
![result4.png](/img/how-to-build-blog/result4.png)
### 备注
添加新文章：在/content/post/里创建新的md文件，然后在rezzoblog里用hugo生成到public里，最后在public里上传到仓库

其它的网站详细配置，就看作者的demo和操作文档看着配置了，这个自己慢慢研究吧