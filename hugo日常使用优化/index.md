# Hugo日常使用优化


<!--more-->

转载请注明出处：`https://hts0000.github.io/`

欢迎与我联系：`hts_0000@sina.com`

# 1 引言
本文用于记录日常使用hugo的优化建议，阅读时长10分钟。

**适用于以下人员：**
- 已经搭建好Hugo博客
- 希望使用图床功能
- 希望自己的博客能在百度和谷歌上搜索到

**本文将从以下几点进行优化：**
- 使用GitHub作为博文的图床
- 让百度和谷歌收录站点

# 2 使用GitHub作为图床储存图片

## 什么是图床
图床是一个专门存放图片的服务器，使用图床可以让我们的博客直接从网上获取图片，而无需管理本地图片路径，让我们获得更好的博文书写体验。

## 优雅的使用图床
方案为：`VsCode+PicGo插件+GitHub`。

VsCode用于书写博客文章，PicGo插件用于快速生成网络图片链接，GitHub作为图片服务器存放图片。

### **操作步骤如下：**
### 1.安装VsCode
点击[`VsCode`](https://code.visualstudio.com/Download)即可前往下载界面。安装十分简单，不再赘述。

### 2.下载PicGo插件
打开VsCode，在左侧的扩展管理中输入PicGo下载插件。
![下载PicGo插件](https://cdn.jsdelivr.net/gh/hts0000/images/20200216161051.png "下载PicGo插件")

### 3.创建GitHub图床仓库
创建一个名字为`<你github账户名>.github.io`的仓库，我这里的是博客使用的仓库，并使用images目录作为图片存放的目录。
![创建GitHub图床仓库](https://cdn.jsdelivr.net/gh/hts0000/images/20200216172535.png "创建GitHub图床仓库")

### 4.生成令牌(Token)
接下来需要生成令牌，让PicGo插件能通过令牌免密上传图片到图床目录。

![生成令牌](https://cdn.jsdelivr.net/gh/hts0000/images/20200216172707.png "生成令牌")
![生成令牌](https://cdn.jsdelivr.net/gh/hts0000/images/20200216172915.png "生成令牌")
![生成令牌](https://cdn.jsdelivr.net/gh/hts0000/images/20200216173149.png "生成令牌")
![生成令牌](https://cdn.jsdelivr.net/gh/hts0000/images/20200216173547.png "生成令牌")

Token生成之后要及时复制，之后就无法复制了。
![20200216173745.png](https://cdn.jsdelivr.net/gh/hts0000/images/20200216173745.png)

### 5.配置PicGo插件
打开VsCode上的PicGo插件，点击小齿轮进行配置。
![20200216174042.png](https://cdn.jsdelivr.net/gh/hts0000/images/20200216174042.png)

在配置中找到GitHub的相关配置项，从上到下分别配置：图床目录(必须加上/)，github仓库，Token令牌。
![配置PicGo插件](https://cdn.jsdelivr.net/gh/hts0000/images/20200216174243.png "配置PicGo插件")

### 6.测试图床
VsCode上的PicGo插件配置好之后，使用`Ctrl+Alt+u`可以把剪切版中的图片上传到图床中，并且会在当前VsCode打开的文件中生产一个图片的网络链接。同时我们看GitHub上的images目录时，会发现图片被自动上传了。

![测试图床](https://cdn.jsdelivr.net/gh/hts0000/images/20200216185606.png "测试图床")

# 2 在百度和谷歌上添加网站

## 为什么要添加网站
在百度和谷歌添加博客的网站之后，就可以通过百度和谷歌的搜索引擎找到自己的博客。

## 收录和站点管理
站点收录：让百度或谷歌知道有这个站点的存在，添加之后需要等1~7天（不同搜索引擎不一样）的时间后才能找到自己的博客。

站点管理：向百度或谷歌证明某个站点是属于你的，即你是网站的管理员。可以通过如下方式进行证明：
- 在网站首页中增加一个校验值标签
- 在网页目录中增加一个校验页面

## 添加收录
测试网址是否被收录：site:hts0000.github.io。

### 百度
![检查收录](https://cdn.jsdelivr.net/gh/hts0000/images/20200216175931.png "检查收录")

添加之后百度大概需要一星期的时间进行收录。
![检查收录](https://cdn.jsdelivr.net/gh/hts0000/images/20200216180208.png "检查收录")

### 谷歌
![检查收录](https://cdn.jsdelivr.net/gh/hts0000/images/20200216175823.png "检查收录")

添加之后谷歌大概需要1~2天的时间来收录网站。
![检查收录](https://cdn.jsdelivr.net/gh/hts0000/images/20200216182402.png "检查收录")

收录成功后在谷歌中搜索：site:hts0000.github.io，就可以发现我们的网站可以通过浏览器搜索到了。
![检查收录](https://cdn.jsdelivr.net/gh/hts0000/images/20200216185252.png "检查收录")

## 站点管理
通过百度账号和谷歌账号，可以绑定网站，通过百度和谷歌的统计功能，可以看到网站的访问次数和访问情况。

让账号和网站绑定，需要向百度和谷歌证明你是网站的管理员，证明的方法有很多种，这次选择校验页面的方式进行验证。

### 百度

首先在账号中添加网站
![站点管理](https://cdn.jsdelivr.net/gh/hts0000/images/20200216182844.png "站点管理")

![站点管理](https://cdn.jsdelivr.net/gh/hts0000/images/20200216183344.png "站点管理")

把下载好的校验页面放到网页目录下，向百度证明你是网站的管理员。
![站点管理](https://cdn.jsdelivr.net/gh/hts0000/images/20200216183517.png "站点管理")

校验成功后可以在站点管理中看到刚刚添加的站点。
![站点管理](https://cdn.jsdelivr.net/gh/hts0000/images/20200216184212.png "站点管理")

点击站点可以看到百度为我们统计的站点信息，如点击率搜索量等等。

![站点管理](https://cdn.jsdelivr.net/gh/hts0000/images/20200216184322.png "站点管理")

### 谷歌
谷歌的操作与百度类似，也是在网页的目录下添加校验页面，向谷歌证明你是网站的管理员。

![站点管理](https://cdn.jsdelivr.net/gh/hts0000/images/20200216184606.png "站点管理")

添加完成之后也可以通过谷歌统计的站点信息查看网站的点击量等等。

![站点管理](https://cdn.jsdelivr.net/gh/hts0000/images/20200216185052.png "站点管理")
