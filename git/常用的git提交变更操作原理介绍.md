[返回上层](index)
# 常用的git提交变更操作介绍

## git的基本对象

git中有四种原子对象，由这四种原子对象构成了git高层数据结构的基础。

### 块（blob）

一个blob保存一个文件的内容数据，但不包括该文件的元数据，甚至连文件的名称也没有。

### 目录树（tree）

一个目录树代表一层目录信息，他记录blob的标识符、路径名、目录里所有文件的元数据。他可以依赖其他的树/子树对象，从而建立一个完整的包含文件和子目录的项目结构。

### 提交（commit）

一个提交对象保存版本库中每一次变化的元数据（作者，提交日期，message等等）。每一个提交对象指向一个目录树对象（该目录树是提交发生的那一刻的整个项目的目录树快照）。每一个提交（除了项目的第一次提交）还会指向他的父提交（前一次提交），由此根据提交间的引用能构建出一个完整的提交历史。

### 标签（tag）

一个标签对象用于分配给其他对象，赋予其他对象一个人类可读的名字（通常是一个提交对象）。

## git commit

### 图解git对象关系

#### 一个提交

![未命名绘图](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-080122.jpg)

#### 新增一个提交

![未命名绘图](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-080130.jpg)

一个新产生的提交，除了初始提交，都会指向一个父提交（即新提交之前的那次提交）。

## git merge

### 图解

![image-20211209180101735](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-080138.jpg)

如上图发生一次feature→master的合并，在git中，合并产生一个新的树对象（如S提交指向的树对象），该树对象包含合并之后的文件，但它只在目标分支（matser）上引入一个新的提交对象。

### 注意点

和普通提交不一样的是，合并产生的提交会指向两个父提交，如图中的D,J。

## git cherry-pick

### 使用场景

cherry-pick 命令通常用于把版本库中一个分支的特定提交引入一个不同的分支中。

git cherrt-pick提交命令会在当前分支上应用给定提交引入的变更。这将引入一个新的独特的提交。使用cherry-pick并不改变版本库中的现有历史记录，而是添加历史记录。

### 图解

如下图，主干分支正在开发中，在提交B处签出分支fix进行修复操作，于此同时master分支在继续往前开发。

![image-20211209180134051](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-080144.jpg)

上图中涉及命令：

```bash
git checkout master
#逐个cherry-pick
git cherry-pick fix~2
git cherry-pick fix^
git cherry-pick fix
#或者批量
git cherry-pick fix~2..fix
```

## gitlab的cherry-pick

gitla提供了对mr的cherry-pick功能

### 使用方式

找到一个已经被合并的mr记录，会看到页面中有一个cherry-pick的按钮

![image-20211209185733562](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-080151.jpg)

点击cherry-pick按钮，选择要cherry-pick的目标分支

![image-20211209180412937](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-080155.jpg)

根据选项，点击cherry-pick按钮后，将新建一个mr还是直接进行cherry-pick。

### 图解

假设feature分支在开发过程中，需要cherry-pick hotfix上的一些提交。

不生成mr的情况下cherry-pick

![image-20211209180444610](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-080202.jpg)

生成mr的情况下 cherry-pick

![image-20211209180505180](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-080210.jpg)

### 注意点

1. **和merge操作不一样，不会产生一条merge的提交记录。**
2. **并不能保证cherry-pick后的代码和fix上一样（由于master在B提交后继续演进，所以在cherry-pick时可能需要解决代码冲突）。**
3. **cherry-pick的冲突解决基于目标分支、原分支指定提交、原分支指定提交的父提交（被当做共同祖先）。**

## git revert

git revert的 原理和git cherrty-pick的原理相似，只不过它是应用给定提交的逆过程。

### 使用场景

撤销某个提交的改动内容

### 图解

![image-20211209180550411](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-080218.jpg)



涉及命令：

```
git revert master~2
```

## git rebase

git rebase也叫”变基提交“，用来改变一串提交以什么为基础。

### 使用场景

git rebase一个常见的使用场景是保持你正在开发的一系列提交相对于另一个分支是最新的。

### 图解

有两个分支正在开发中，最初topic分支是从master分支的提交B处开始的。在此期间master分支已经进展到提交E

![image-20211209180627970](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-080225.jpg)



可以改写提交让他们基于提交E而不是提交B，这样提交相对于master就是最新的了。

![image-20211209180654903](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-080233.jpg)





涉及命令：

```bash
git checkout topic
git rebase master
#或者
git rebase master topic
```

### 注意事项

1. **变基操作一次只迁移一个提交，从各自原始提交位置迁移到新的提交基础，因此每个移动的提交都有可能需要解决冲突。**
2. **在某个提交变基发生冲突时，可以选择解决冲突或者跳过并开始迁移下一个提交（不建议）。**



# 参考

[Git版本控制管理（第2版）](https://www.epubit.com/bookDetails?id=N8405)

https://docs.gitlab.com/ee/user/project/merge_requests/cherry_pick_changes.html

https://blog.experteer.engineering/what-are-parents-on-git-merge-commits.html


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

