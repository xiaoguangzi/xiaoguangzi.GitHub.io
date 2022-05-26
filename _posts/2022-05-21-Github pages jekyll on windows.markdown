---
layout: single
title:  "GitHub pages jekyll on windows"
---

安装方法来自网络，适用于windows

**创建GitHub账号**

这个就不具体的写了，大家应该都有这个帐号了,没有的自己注册一个

**create a repository**

![image](https://user-images.githubusercontent.com/87790245/169654097-ad69afdb-c421-470b-a827-57f2391a266b.png)

进入页面后，在 Repository name 的位置填写域名，格式是 username.GitHub.io。

**这里需要注意的是需要跟你现在的用户名相同，不然访问提示404**

创建成功之后，点击右上角的 Settings

![image](https://user-images.githubusercontent.com/87790245/169654227-6ff53f18-e516-42f9-8942-5653a7e88484.png)

找到 GitHub Pages 选项，选择一个 GitHub 官方提供的主题

这个可以随便选一个就行了，反正自带的都不怎么样我感觉，但是因为我准备的是极简风格，所以感觉还行

选择完毕之后 GitHub Pages 就会自动帮你生成好网站，在他跳转的界面点击 Commit changes 按钮，网站就可以访问了。

在浏览器里输入你的项目名称，比如 xiaoguangzi.GitHub.io，就可以看到，你刚刚选择的主题的个人网站的页面了。

**安装jekyll**

windows并不是jekyll的默认支持平台，但是可以通过ruby的进行安装。在安装中有可能遇见问题就是jekyll安装完成后不能使用，
解决办法为重新安装ruby-Devkit，或者在安装ruby的时候选择安装MSYS2.

这里放上jekyll的链接: 

<https://jekyllrb.com/docs/installation/windows/>

**挑选主题**

jekyll的主题有很多，并且都与GitHub pages  100%兼容，可以根据自己的喜好去选择。因为在安装过程中出现了一些问题，导致了我的博客
主题提供的服务几乎都不能用，我调试了很长时间也没有成果，于是选择了直接更换主题。

<http://jekyllthemes.org/>

在这里你可以找到很多主题并且有非常详细的说明，可以根据自己的爱好选择搭建主题。

欢迎大家沟通交流。