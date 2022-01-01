[返回上层](index)
# 本地运行runner并注册到gitlab

有时候我们在通过gitlab-ci来执行一些特殊的任务时，公司内提供的共享的gitlab-runner可能并不能支持。这个时候我们可以通过向gitlab注册一个在我们本地运行的私有的runner来执行这些特殊的任务。

要在本地运行runner并注册到gitlab，大致需要三个步骤：

- 本地安装git-runner程序
- 新建一个runner并注册到gitlab
- 启动本地的runner

## 本地安装git-runner程序

gitlab-runner支持在很多操作系统环境下的安装（https://docs.gitlab.com/runner/install/），本文将演示在MacOS环境下安装gitlab-runner。

1.为您的系统下载二进制文件：

```bash
sudo curl --output /usr/local/bin/gitlab-runner "https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-darwin-amd64"
```

您可以下载每个可用版本的二进制文件，[点击这里查看](https://docs.gitlab.com/runner/install/bleeding-edge.html#download-any-other-tagged-release)。

2.赋予其执行权限：

```bash
sudo chmod +x /usr/local/bin/gitlab-runner
```

这个时候，gitlab程序已经在我们的机器上安装成功。可以通过在命令行中执行相关命令进行验证：

![image-20211011080132042](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-161508.jpg)

### 使用macOS的brew进行安装

macOS下还支持通过brew安装gitlab-runner程序，使用以下命名即可安装：

```bash
brew install gitlab-runner

brew services start gitlab-runner

```

## 新建一个runner并注册到gitlab

macOS下新建并注册一个runner也很简单，可以通过下面命令来交互式地注册一个本地runner到gitlab：

```bash
gitlab-runner register
```

运行该命令之后，会有一些列的runner信息让你填写：

![image-20211011080821103](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-161515.jpg)

命令结束之后，我们就已经将runner注册到gitlab之上了，这个时候我们可以在gitlab的项目页面上(**项目页面>settings>CI/CD>Runners>Specific runners**)看到我们注册的runner信息：

![image-20211011081252561](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-161520.jpg)

在Available specific runners栏目下可以看到我们新注册上去的runner，不过我们可以看到我们新注册的runner还处于无法连接的状态，这是因为我们的runner还没有启动。

## 启动本地的runner

注册完成之后，我们只需要在本地运行我们的gitlab-runner就可以让在gitlab中正常使用这个runner了。首先使用以下命令将系统用户切换为当前操作用户：

```bash
su - <username>
```

将 GitLab Runner 作为服务安装并启动它：

```bash
cd ~
gitlab-runner install
gitlab-runner start
```

到此我们的runner已经注册，并启动。稍等几分钟，再去gitlab页面中观察，可以看到我们的runner已经处理在线状态了，我们可以用它来执行任务了。

![image-20211011082006634](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-161530.jpg)

## 使用私有的runner来运行任务

在.gitlab-ci.yml中编辑要使用本地runner运行的任务

![image-20211011082159261](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-161536.jpg)

如上图，将其tag指定为本地runner的tag，提交代码后可以看到触发的gitlab ci job将会在我们新注册的本地runner中运行👌：

![image-20211011082444717](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-161542.jpg)


---
---
---


## 🤔  💭 👇👇👇

<script src="https://utteranc.es/client.js"
        repo="dongxishaonian/issue-posted"
        issue-term="pathname"
        label="🙂🙃😡🥶😬🤣😄"
        theme="github-light"
        crossorigin="anonymous"
        async>
</script>

