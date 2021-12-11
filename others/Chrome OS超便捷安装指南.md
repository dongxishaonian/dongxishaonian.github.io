[返回上层](index)

# Chrome OS超便捷安装指南

> Chrome OS是一款Google开发的基于[PC](https://baike.baidu.com/item/PC/107)的操作系统。 Google Chrome OS是一款基于Linux的[开源操作系统](https://baike.baidu.com/item/开源操作系统/4581071)。Google在自己的官方博客表示，初期，这一操作系统将定位于[上网本](https://baike.baidu.com/item/上网本/4046734)、紧凑型以及低成本电脑。这款[开源软件](https://baike.baidu.com/item/开源软件/8105369)将被命名为Chrome OS，[谷歌](https://baike.baidu.com/item/谷歌)公司于2010年12月7日（北京时间12月8日2点30分）在美国举行Chrome相关产品发布会，发布会上正式发布[Chrome Web store](https://baike.baidu.com/item/Chrome Web store)和Chrome OS。
>
> 百度百科-GOOGLE CHROME OS

最近在操作系统领域的一则新闻引起了大家的注意，在 [IDC 的数据统计中](https://www.sohu.com/a/451216653_161062)我们看到了谷歌的Chrome OS操作系统的市场份额超越了MacOS升至第二，仅次于微软的Windows。第一眼看到这个统计非常令人意外，平时都不怎么听说过的Chrome OS的增长势头居然如此的凶猛。不过一想原因其实也能理解，在国内由于某些原因，对于Google的产品大家本身接触的机会就不多，更别说会使用Chromebook笔记本了。

**随手一搜就能看到很多关于Chrome OS的新闻：**

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-111141.jpg)Chrome OS市场份额超越MacOS

出于好奇，我决定体验一下这个听起来很厉害的Chrome OS。下面是整个安装体验过程：

------

## 安装步骤

1. 制作U盘启动器
2. 安装Chrome OS

### 制作U盘启动器（适用于Windows环境下）

由于谷歌方面并没有开放Chrome OS镜像文件的下载，不过类似于Chrome浏览器，Chrome OS也拥有自己的开源版本系统Chromium OS，所以第三方可以基于Chromium OS来开发出类似于Chrome OS的系统。所以在本文中我将安装基于Chromium OS的[CloudReady系统](https://www.neverware.com/#intro)（一个基于Chromium OS开发出来的操作系统，国内也有类似的操作系统如：[fyde OS](https://fydeos.com/)）来体验Chrome OS的魅力。

#### 1.进入[CloudReady](https://www.neverware.com/#intro)官网页面

进入[CloudReady](https://www.neverware.com/#intro)官网页面，鼠标下拉到如下位置：

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-111151.jpg)选择版本

CloudReady提供了三种版本的系统供用户选择，这里我们选择第三种面向个人和家庭的免费版本。点击“GET THE FREE VERSION”按钮，进入系统镜像的下载页面。

#### 2.下载U盘启动器制作工具

**在制作U盘启动器之前需要准备一个容量大于8GB的U盘**，CloudReady官方也提供了制作U盘启动器的工具，不过目前此工具只针对Windows用户，如果是Mac用户可以使用后面我提供的方法来制作系统启动盘。

进入下载页面，页面下拉到如下位置：

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-111157.jpg)下载u盘启动器制作工具

点击“DOWNLOAD USB MAKER”将工具下载至本地，并安装。

#### 3.制作USB启动盘

将准备的U盘插入电脑中，随后打开刚刚安装的启动盘制作工具，按如下步骤进行操作：

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-111206.gif)制作U盘启动器

操作至此，在windows下，我们的CloudReady U盘启动器已制作完成。

### 制作U盘启动器（适用于MacOS、ChromeOS环境下）

我们看到了在Windws下制作CloudReady的U盘启动器非常的方便快捷，不过如果我们的操作系统是Mac OS或者Chrome OS的时候，U盘启动器的制作过程会略微复杂一点。

#### 1.下载镜像

进入下载页面之后，鼠标下拉到如下位置：

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-111221.gif)镜像下载

点击图中的“DOWNLOAD 64-BIT IMAGE”按钮将镜像下载至本地。

#### 2.安装并启动Chromebook Recovery Utility

下载[Chromebook Recovery Utility](https://chrome.google.com/webstore/detail/chromebook-recovery-utili/jndclpdbaamdhonoechobihbbiimdgai/related) 扩展并将其添加到您的Chrome浏览器中（如果没有安装Chrome浏览器，则需要先安装chrome浏览器）。

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-111436.jpg)添加扩展

随后在Add "Chrome Recovery Utility"的提示下，点击“**Add app**”：

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-111444.jpg)add app

此时Chrome Recovery Utility应用已被添加至系统的应用程序列表中，打开Chrome Recovery Utility：

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-111452.png)打开Chrome Recovery Utility

如果找不到该页面，可以手动在浏览器地址栏中输入**chrome://apps**来访问。

打开Chrome Recovery Utility后，点击应用窗口右上角的设置图标，并点击“Use Local Image”，随后选择刚刚下载的镜像文件：

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-111458.jpg)use local image

点击“Get started”按钮，进入下一步，此时插入我们准备好的U盘，并进行如下操作：

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-111504.jpg)插入u盘

选中我们插入的U盘，继续点击下一步。

此时出现启动盘制作进度条，等待完成，至此U盘启动器制作完成：

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-111508.jpg)制作完成

### 安装CloudReady

将制作完成的U盘启动器插入计算机，随后重启我们的计算机并进入BIOS系统（关于如何进入BIOS读者可以自行百度，不同品牌的计算机进入方式不同），在BIOS的Boot选项中修改计算机的启动顺序（将从U盘启动放在第一位），然后退出BIOS重启计算机，稍等片刻CloudReady将在计算机中启动。

如果是Mac用户，在计算机重启的过程中一直按住“option”键，进入启动项选择界面，选择我们插入的U盘作为启动项启动。

至此，类Chrome OS系统CloudReady就已经成功启动，等待片刻，剩下的步骤就是体验或者安装到本机。
![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-111858.gif)
![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-111828.jpg)启动中...

------

**注意事项：由于在国内访问Google相关站点时会受到一定的限制，所以需要提前准备一些工具手段来连接到Google的服务。**

**参考：**

https://www.lifewire.com/install-chrome-os-on-pc-4174289

https://guide.neverware.com/build-installer/working-mac-os/#download-cloudready

------

🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟

**欢迎访问笔者博客：[https://www.techflower.cn](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.techflower.cn%2F)**

**关注笔者公众号，推送各类原创/优质技术文章 ⬇️**

<img src="http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-162353.jpg" alt="image-20200410104030284" style="zoom:50%;" />

**😂🤣🤨🤩🥴😘🥸我也是一名低调的Up主：[https://space.bilibili.com/330000719](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fspace.bilibili.com%2F330000719)**
