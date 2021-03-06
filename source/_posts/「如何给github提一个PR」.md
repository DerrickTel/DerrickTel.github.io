---
title: 「如何给开源库提一个PR」
date: 2021-01-22 16:39:14
tags: [github]
categories: [github]
cover: /image/cover/github.jpg
---

## 前言

之前一直想给`github`上面优秀的项目做一些力所能及的事情，比如解决一下`issues`，或者提一个`pr`，修改一下`bug`。但是苦于一直不知道怎么正确的去做这件事情。所以搁置了，后来发现了[pro-components](https://github.com/ant-design/pro-components)这个项目，和我的理念非常契合，就是一个包装版的[ant-design](https://github.com/ant-design/ant-design)，可以更快更好的使用[ant-design](https://github.com/ant-design/ant-design)。

因为一个比较出名的项目的`pr`，可能在简历筛选的时候会比较吃香，这个应该大家都知道，而且还可以混个脸熟。哈哈哈。

主要是为了记录自己的操作过程，和分享。

接下来会涉及到实际操作，所以请大家先点赞收藏再看。

## github三个按钮

我们知道，`github`上面的代码是可以直接看的，方式有以下几种：

- 下载`zip`
- `fork`到自己的项目中，然后自己`clone`下来(重点介绍，和提`pr`密切相关)
- 直接`clone`下来
- 在网页上直接观看



我们看到`github`项目的时候，发现不仅有我们都知道的`star`，还有`watch`和`fork`。

![pornhub](/image/pr/title.png)

### Star

这里说点废话，小伙伴有没有人和我一样英语不好，经常记成`start`的！！！

`star` 翻译过来应该是星星，这里解释为`关注`或者`点赞`更合适

`star`的话就相当于是我给这个项目做一个标记，或者说是给这个项目点了一个「好评」

通常我们认为一个项目的受欢迎程度基本上都是由star来进行评判的。（注意！是基本上，当然还有npm的下载数，这个有点类型观看数

![pornhub](/image/pr/pornhub.jpg)

有点类型上图的，喜欢数（`start`，和观看次数（`npm`下载量



### Watch

`Watch`翻译过来可以称之为观察。

默认每一个用户都是处于`Not watching`的状态，当你选择`Watching`，表示你以后会关注这个项目的所有动态，以后只要这个项目发生变动，如被别人提交了`pull request`、被别人发起了`issue`等等情况，你都会在自己的个人通知中心，收到一条通知消息，如果你设置了个人邮箱，那么你的邮箱也可能收到相应的邮件。

如果你不想接受这些通知，那么点击` Not Watching` 即可。



### Fork

这里要重点说一下，因为和我们现在的主题：RP（Pull Request）息息相关。

> 接下来要说的pr就是pull request

当选择` fork`，相当于你自己有了一份原项目的拷贝，当然这个拷贝只是针对当时的项目文件，如果后续原项目文件发生改变，你必须通过其他的方式去同步。

一般来说，我们不需要使用` fork` 这个功能，至少我一般不会用，除非有一些项目，可能存在` bug` 或者可以继续优化的地方，你想帮助原项目作者去完善这个项目，那么你可以` fork `一份项目下来，然后自己对这个项目进行修改完善，当你觉得项目没问题了，你就可以尝试发起 `pull request`给原项目作者了，然后就静静等待他的 `merge`。

当前很多人错误的在使用` fork`。很多人把` fork `当成了收藏一样的功能，包括一开始使用 `github `的我，每次看到一个好的项目就先 `fork`，因为这样，就可以我的` repository`(仓库)列表下查看 `fork` 的项目了。

> 而且还有一个问题，就是你fork的是当前，此时此刻的代码，假如后续作者更新了新的内容，你fork的就不会变，但是有一个方式可以pull到最新的代码，这个也是很重要的，不如你提交你的pr的时候，会有冲突

总得来说，`fork`就是给我们提`pr`使用的。



## 实操

### fork项目到自己的仓库

找到自己想要提pr的项目，然后点击图上的「fork」

![pornhub](/image/pr/title.png)

加入有Organizations（组织）的同学可能会需要选择是自己个人的项目还是的Organizations（组织）项目。

选择完之后就会出现在自己的项目里面了。



### clone代码到本地

在自己的` repository`(仓库)里面找到刚刚`fork`的项目，通常是同名的，然后`clone`到自己的机器上面之后。



### 新建分支

虽然，可以不用新建分支，直接在master分支上面commit和push，但是！不符合规范，因为master分支我们接下来可能要用作于与真正的要修改的项目的master联通要用的，所以，最好是再弄一个分支来区别开来，之后操作完可以删除。

理论上我们还是要符合一个过程化的规范的，虽然也可以不用这样，但是我们也可以学习别的优秀的开源代码的规范来命名，帮助自己提前感受一下大神的感觉。

我们根据master分支新建完分支，然后切换分支之后就可以开始修改代码了。



### 提交代码

这里要说一下，一般来说，每个开源库都有自己的代码规范，比如我最近参与的[pro-components](https://github.com/ant-design/pro-components)这个项目，只需要符合他的eslint规范就好了。后面可能还会涉及到一些单元测试的问题，后面会说到。

你改完之后，就需要commit了，有的项目会要求你按规范写commit message。比如[pro-components](https://github.com/ant-design/pro-components)这个项目，如图

![pornhub](/image/pr/commit.png)

是的，你发现了，左下角有一个跳过...最好还是要符合规范，毕竟在别人的地盘办事，而且你的所有记录都是可以查的哦，以后被人查你的pr，会发现你是一个会规范的人很加分哦！！



### 开始提PR

跑远了，在我们正常的`commit`， `pull `，`push`之后，就可以打开我们想修改的那个仓库。

会在头顶出现一个东西。

![pornhub](/image/pr/push.png)

我们需要点击图中框起来的「Compare & pull request 」，之后就会出现如下图

![pornhub](/image/pr/create.png)

默认会帮我们选好分支的，我们只需要完善其中的信息，还有我们之前提交的message也可以修改。最好可以用英文来解释，本次提交的内容。

然后点击提交之后就好了。



#### 方法二

我们可以直接点击

![pornhub](/image/pr/pr.png)

然后选择和之前一样的东西就好，具体是选择项目以及要合并的分支，然后写内容。点击提交。



### 后续

一个优秀项目通常会有代码审核，这个时候，你可能需要修改一下测试用例，然后我们只需要在前面创建的分支上继续`push`就好了。可能还会有作者给你的意见。改完之后继续`push`



### 填坑



前面说的，我们fork的只是当时的代码，后续作者更新的我们需要重新拉取，操作如下：

#### 添加

```
$ git remote add upstream https://github.com/xx/xx.git
```

#### 查看

```
$ git remote -v 

origin	ssh://xxx (fetch)
origin	ssh://xxx (push)
upstream	ssh://yyy (fetch)
upstream	ssh://yyy (push)
```

#### 取消

```
$ git branch --unset-upstream
```

#### 拉取

覆盖本地的 master。

```
$ git fetch upstream
$ git checkout master
$ git rebase upstream/master
```



### 最后

只需要等待作者`merge`啦。然后祝大家顺利，有任何问题都可以在评论区指出或者于我讨论。