---

layout: post
title:  开发人员必知的Git技能及Git工作流总结！--20211115
tagline: by 揽月中人
categories: Git
tags:
- 揽月中人
---

<!--more-->

Git作为最流行的代码版本控制工具，基本上已经成为了程序员的一个标配技能。无论使用GitHub，GitLib，Gitee等进行代码托管，均基于Git。下面聊一聊开发人员必会的几个Git技巧和团队协作的一些Git工作流。

### 1 Git 常用的超级实用命令

#### 1.1 与仓库相关的操作

克隆代码仓库到本地，开发必用

```shell
git clone  <url>
```

 查看本地仓库配置了那些对应的远程仓库。

```shell
git remote -v 
```

添加远程仓库

```shell
git remote add <name><url>
```

更新远程仓库地址

```shell
git remote set -url --push <name><newUrl> 
```

拉取远程仓库

```shell
git pull <remoteName><localBranchName> 
```

 推送本地仓库到远程仓库，默认是当前所在branch

```shell
git push <remoteName><localBranchName>
```



#### 1.2 分支的创建切换等相关操作

 查看本地分支/所有分支  

```shell
git branch  / git branch -a
```

查看远程分支

```
git branch -r 
```

创建本地分支

```shell
git branch <name> 
```

本地创建分支并和远程分支关联，再切换到该分支。

```shell
git checkout -b <本地分支名> <远程仓库名>/<远程分支名>
git branch -b <本地分支名> <远程仓库名>/<远程分支名>  
```

 根据远程分支创建本地分支，但是不会切换到新分支，需要手动checkout

```shell
git fetch <远程仓库名> <远程分支名>:<本地分支名>
```

创建新分支并立刻切换到改分支

```shell
git checkout -b <name> 
```

创建远程分支： 

```shell
git push origin <name>
```

删除远程分支

```shell
git push origin:heads/<name>
也可以push一个空的本地分支，那么也将删除远程分支
```

修改分支名

```shell
git branch -m <oldName> <newName>
git branch -m <newName>  (修改当前BranchName)
```

#### 1.3 Tag相关

查看Tag

```shell
git tag
```

创建Tag

```shell
git tag <name>
```

删除Tag

```shell
git tag -d <name>
```

查看远程Tag

```shell
git tag -r
```

push Tag到远程仓库

```shell
git push origin <tagName>
```

删除远程Tag

```shell
git push origin:refs/tags/<tagName>
```

#### 1.4 git 提交相关

先add 然后在提交，不过add大多时候利用开发工具来做比较方便。

```shell
git add newfile.txt 
git commit -m "the commit message"
```

reset相关的命令可以回滚刚才的add或者提交，重设当前分支

```shell
git reset [--hard|soft|mixed] [<commit>或HEAD]
```

最后一个参数默认为HEAD，HEAD~2表示上上一个版本，也可以是某一个commit id处。

**常用的三个参数hard/mixed/spft**  

--hard 将之前的提交全部删除stage区清空，

--mixed 将之前的提交删除，但是将改动移动到stage区（也就是index中）。

--soft 提交不改变变，将HEAD指向某commit id，有点像checkout

#### 1.5 合并

合并其他分支到当前的分支

```shell
git merge <branchName>
```

合并分支`fixes`和`enhancements`在当前分支的顶部

```shell
git merge fixes enhancements
```

将一个commit 合并到当前分支

```
git cherry-pick <commit id>
```

合并几个连续的commit

```shell
git rebase -i 4e08572
```

下面给出一组Rebase 的详细示例

（1）windows 下，输入上述命令之后, 输入i 进入编辑窗口，更改rebase策略。详细解释都有提示，只需根据提示输入即可。

![image-20211109023846328](http://www.javanorth.cn/assets/images/2021/lyj/rebase1.png)

（2）选好rebase策略之后按Esc推出 输入"：x" 执行 刚才的rebase操作，然后会看到修改提交的信息界面

![image-20211109023928694](http://www.javanorth.cn/assets/images/2021/lyj/rebase-commitmessage.png)



（3）修改提交信息，按Esc退出，并输入 "：x" 执行rebase操作

然后看到rebase成功 

查看刚才的log 

![	e](http://www.javanorth.cn/assets/images/2021/lyj/gitlog.png)



以上就是一个简单的rebase操作。

#### 1.6 log & show

查看最近的5次提交，按Q就直接退出。

```shell
git log -5
```

使用ASCII图形表示分支合并历史

```shell
git log --graph
```

显示最近5次提交的详细内容

```shell
git show -5
```













### 2 Git Work Flow

#### 2.1 Centralized Workflow

Centralized WorkFlow 这种模式对比熟悉SVN的开发人员比较容易适应。在Git 远程仓库有一个主分支，开发人员也在自己的本地克隆了一个完全一样的分支，开发人员可以在本地的分支上随意的进行代码编写，测试。当完成当前开发工作之后，可以直接push到远程主分支上。就算是有冲突也可以快速的解决。

中心仓库式工作流对于小型开发团队比较适用，人员少，改动也就相对比较少，出现冲突的几率也要少很多。就算是有冲突，合并也不会有太大问题。显然这种工作方式却不适合大型团队，几个开发组同时开发维护一个项目，这样代码会非常混乱，每个人的提交都会影响其他组开发的工作。如果再有不同组的开发任务上线时间点不一样，那么就是一场灾难。

#### 2.2 Feature Branch Workflow

基于Feature Branch WorkFlow 的Git工作流，其基于一个中心仓库，主分支（也可以是其他的生产代码分支）代表了项目历史。每次有新的开发任务的时候，基于主分支新创建一个Feature分支，和当前任务相关的代码都提交的这个Feature分支。

当开发工作进行到一定阶段，就可以提一个Pull Request代主分支，这个时候就可以让同事去Review自己的代码了。此时Review阶段可能会有一些小问题，或者同事给出了更好的建议，我们都可以针对性的修改代码。

最后让测试等工作都完成之后，就可以将Feature 分支合并到主分支上。最终测试完成之后就可以部署到生产上面。之后Feature分支可以视情况删除，或者保留一段时间之后再删除。

#### 2.3 GitFlow WorkFlow

GitFlow WorkFlow是一种比较经典的工作模式。

其主要有如下的一些分支协作。

**Main/Master Branch** （主分支）：维护着项目的历史记录，会记录每一次上线的标签，并且可以看到每一次上线的一些新特性等。

**Develop Branch**（开发分支）：基于主分支，主要是为了下一次上线所使用的开发分支，可以接受Feature分支的Pull Request。当所有的新特性都已经合并完毕，那么就创建。

**Release Branch**（部署分支）：当开发分支已经合并了足够的特性分支代码之后，会创建一个Release分支。此时Release分支不会接受任何Feature分支的Pull Request，仅接受一些Bug fix的Pull Request。Release分支部署上线之后，测试全部通过后会将代码合并到Develop 分支和主分支。

**Feature Branch**（特性分支）： 一般会基于开发分支创建出一条针对某个功能的分支，开发人员会基于这条分支进行开发和单元测试的工作，当本Feature开发完成之后。 会将代码合并到Develop分支上，之后可以选择适时删除次分支。

**Hotfix Branch**（修补分支）：针对与线上出现的紧急问题，从主分支，也就是上一次上线的标签出创建一个Hotfix 分支。 当问题解决之后，可能会有一次紧急的上线来解决之前发现的问题。当问题解决之后，会将代码合并到主分支，和开发分支。 以便下一次上线也会包含本次的紧急修复代码。

**Support Branch**（备用支撑分支）：这个分支的应用比较灵活，有时候如果项目上线的时候基于各种可能性, (比如某个依赖的系统上线失败，或者其他不必要的特性上线失败) 可能会需要准备多个版本，那么Support Branch 就可以作为一条备用分支来使用。这个分支可以根据项目的实际情况来看是否需要使用！

各个Branch的示例关系可以看下图

![Git flow workflow - Hotfix Branches](http://www.javanorth.cn/assets/images/2021/lyj/04 Hotfix branches.svg)

我们也可以使用git flow 的一些命令来时用这个工作模式，会有一些既有的命令，使用起来会比较规范，但是也会有一定复杂，项目中可以根据实际情况食用！

![image-20211107115253322](http://www.javanorth.cn/assets/images/2021/lyj/GitFlow1.png)

git flow 相关的一些命令

![image-20211108011549169](http://www.javanorth.cn/assets/images/2021/lyj/gitFlowCommand.png)

#### 2.4 Forking Workflow

Forking WorkFlow 相对与上面的几种协同工作方式有较大的不同。

其主要**公共远程仓库**，**私人远程仓库**，和**本地仓库**。

开发人员一般会从公共远程仓库Fork一份完全一样的代码仓库到自己的**私人远程仓库**。对于私人远程仓库来说，开发人员是拥有者，拥有所有的权限。 开发人员可以在本地仓库提交之后随时push到自己的私人远程仓库，随时进行Revert、Reset、Rebase等操作。

公共远程仓库为项目的组织官方仓库，保留有不同的分支，以及标签等。可以由私人远程仓库提交Pull Request到公共远程仓库的某个分支，待同事或者一些Owner Review过之后，就可以合并到公共仓库。

Forking WorkFlow 也是比较推荐的一种方式，开发人员push代码之后，可以随时在自己的提交基础之上进行Rebase操作，合并之前的许多提交，这样提交历史比较整洁，同时Review也比较方便。



### 3 Git 常见问题及处理方法以及建议。

1. 使用Github合并PullRequest的时候，如果代码有冲突，通过网页解决冲突合并之后。Github会进行**双向合并**。
2. 关于Git协作，可以考虑Git Flow与Feature flow 还有Forking flow结合使用，Forking Flow有利于整理提交历史，Feature branch对于代码的管理比较友善。
3. 本地的代码跑完一次测试的时候就可以提交一版，然后在需要push的时候在rebase合并一次。尽量完善自己的commit信息，写好每一次提交记录。
4. 如果merge有问题可以使用git merge --abort 解除merge， 然后再重新合并。
5. 多使用Git命令行来进行日常的提交等工作，有助于更好的理解Git的工作原理，这样在不同的IDE上都能比较容易的使用Git插件了。



### 总结

选择什么样的Git工作流，什么样的工作流最适合自己的团队呢。

其实关于这个问题有如下的几个方面可以值得考虑：

1. 当前选择的团队协作方式是不是可扩展性的，随着团队的大小变化，都可以比较容易适应团队。
2. 当前的工作流相比于其他的形式，可不可以更容易的发现和避免错误，比如Review是否有效。
3. 当前选择的工作流对于团队工作来说需要额外花费多少精力，是不是会浪费一些不必要的精力。

