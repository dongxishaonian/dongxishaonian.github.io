[返回上层](index)
# Gitflow分支管理策略

Gitflow存在两个记录项目历史的分支

- **Master分支**：存储（官方的，正式的）项目发布历史记录的分支。
- **develop分支**：充当功能的集成分支。

![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-112404.jpg)

Develop分支将包含项目的完整历史记录，而master将包含简化版本。现在，其他开发人员应该克隆中央存储库，并为develop创建跟踪分支。

⚠️：**基于Master分支创建develop分支。**

---

## 特性分支（feature branch）

每个新功能应驻留在其自己的（特性）分支中，可以将其推送到中央存储库以进行备份/协作。但是，特性（feature）分支不是基于master分支创建，而是将develop分支作为自己的父分支。功能完成后，它会重新合并到develop分支中，并删除当前的feature分支。feature分支绝不能直接与master分支进行交互。

![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-112408.jpg)

⚠️：**通常我们基于最新的develop分支创建feature分支。**

---

## Release分支

一旦develop分支获得了足够的发布功能（或临近预定的发布日期），你就需要基于develop分支创建release分支，创建此分支将开始下一个发行周期，因此此刻之后release分支不能添加任何新功能-除了错误修复，文档生成以及其他面向发行版的任务的改动。一旦准备好发布，release分支将合并到master分支并用版本号标记。另外，应该将其合并回develop分支，因为release分支可能已经存在新的提交。

![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-112413.jpg)

使用专用的分支进行发布可以让我们在发布的同时，其他团队可以继续为下个发行版本开发新功能，而不影响此次发布。它还创建了明确定义的开发阶段（例如，可以很容易地说：“本周我们正在为版本4.0做准备，并且可以在存储库的结构中实际看到它“）。

一旦release分支准备好发布，它将被合并到master和develop分支中，然后release分支将被删除。重新合并到develop分支中很重要，因为关键更新可能已添加到release分支，并且新功能需要访问这些更新。如果您的组织强调code review，那么这将是合并到develop的理想场景。

⚠️：**release分支基于develop分支。**

---

## hotfix分支

维护或“hotfix”分支用于快速修补生产版本。hotfix分支与release分支和feature分支很像，只是hotfix分支是基于主版本而不是开发版本的。

**hotfix是唯一应直接从master创建的分支**

修复程序完成后，应将其合并到master和develop（或当前release分支）中，随后删除hotfix分支，并应使用更新的版本号（tag）标记master分支。

![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-112417.jpg)

拥有专门的错误修复开发线，您的团队可以在不中断其余工作流程或不等待下一个发布周期的情况下解决问题。您可以将hotfix分支视为直接与master一起使用的临时发行分支。

## Gitflow的总体流程为：

1. A `develop` branch is created from `master`
2. A `release` branch is created from `develop`
3. `Feature` branches are created from `develop`
4. When a `feature` is complete it is merged into the `develop` branch
5. When the `release` branch is done it is merged into `develop` and `master`
6. If an issue in `master` is detected a `hotfix` branch is created from `master`
7. Once the `hotfix` is complete it is merged to both `develop` and `master`

---

![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-112422.jpg)

## 总结

本文介绍了gitflow的工作流程

---

**关注笔者公众号，推送各类原创/优质技术文章 ⬇️**

![WechatIMG6](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-112428.jpg)


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

