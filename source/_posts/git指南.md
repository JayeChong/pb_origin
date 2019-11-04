---
title: git指南
date: 2019-09-04 11:29:03
tags: [技能,git]
---

### Git基础

[gitbook](https://git-scm.com/book/zh/v2)

Git 和其它版本控制系统（包括 Subversion 和近似工具）的主要差别在于 Git 对待数据的方法。

概念上来区分，其它大部分系统以文件变更列表的方式存储信息。 这类系统（CVS、Subversion、Perforce、Bazaar 等等）将它们保存的信息看作是一组基本文件和每个文件随时间逐步累积的差异。反之，Git 更像是把数据看作是对小型文件系统的一组快照。 每次你提交更新，或在 Git 中保存项目状态时，它主要对当时的全部文件制作一个快照并保存这个快照的索引。 为了高效，如果文件没有修改，Git 不再重新存储该文件，而是只保留一个链接指向之前存储的文件。 Git 对待数据更像是一个**快照流**。

Git 的分支实质上仅是包含所指对象校验和（长度为 40 的 SHA-1 值字符串）的文件，所以它的创建和销毁都异常高效。 创建一个新分支就相当于往一个文件中写入 41 个字节（40 个字符和 1 个换行符），如此的简单能不快吗？

这与过去大多数版本控制系统形成了鲜明的对比，它们在创建分支时，将所有的项目文件都复制一遍，并保存到一个特定的目录。 完成这样繁琐的过程通常需要好几秒钟，有时甚至需要好几分钟。所需时间的长短，完全取决于项目的规模。而在 Git 中，任何规模的项目都能在瞬间创建新分支。 同时，由于每次提交都会记录父对象，所以寻找恰当的合并基础（译注：即共同祖先）也是同样的简单和高效。 这些高效的特性使得 Git 鼓励开发人员频繁地创建和使用分支。

<!-- more -->

#### 三种状态

![三种状态](./areas.png)

基本操作流程：
- 在工作目录中修改文件。
- 暂存文件，将文件的快照放入暂存区域。
- 提交更新，找到暂存区域的文件，将快照永久性存储到 Git 仓库目录。

#### Git配置

Git 自带一个 git config 的工具来帮助设置控制 Git 外观和行为的配置变量。 这些变量存储在三个不同的位置：

1. /etc/gitconfig 文件: 包含系统上每一个用户及他们仓库的通用配置。 如果使用带有 --system 选项的 git config 时，它会从此文件读写配置变量。

2. ~/.gitconfig 或 ~/.config/git/config 文件：只针对当前用户。 可以传递 --global 选项让 Git 读写此文件。

3. 当前使用仓库的 Git 目录中的 config 文件（就是 .git/config）：针对该仓库。

每一个级别覆盖上一级别的配置，所以 .git/config 的配置变量会覆盖 /etc/gitconfig 中的配置变量。

当安装完 Git 应该做的第一件事就是设置你的用户名称与邮件地址。 这样做很重要，因为每一个 Git 的提交都会使用这些信息，并且它会写入到你的每一次提交中，不可更改：

```bash
$ git config --global user.name "John Doe"
$ git config --global user.email johndoe@example.com
```

检查配置信息
如果想要检查你的配置，可以使用 git config --list 命令来列出所有 Git 当时能找到的配置。

```bash
$ git config --list
user.name=John Doe
user.email=johndoe@example.com
color.status=auto
color.branch=auto
color.interactive=auto
color.diff=auto
...
```
你可能会看到重复的变量名，因为 Git 会从不同的文件中读取同一个配置（例如：/etc/gitconfig 与 ~/.gitconfig）。 这种情况下，Git 会使用它找到的每一个变量的最后一个配置。

你可以通过输入 `git config <key>`： 来检查 Git 的某一项配置

```bash
$ git config user.name
John Doe
```

#### 跳过使用暂存区域
 Git 提供了一个跳过使用暂存区域的方式， 只要在提交的时候，给 git commit 加上 -a 选项，Git 就会自动把所有已经跟踪过的文件暂存起来一并提交，从而跳过 git add 步骤。

#### [配置Git log](https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E6%9F%A5%E7%9C%8B%E6%8F%90%E4%BA%A4%E5%8E%86%E5%8F%B2)

 - 一个常用的选项是 -p，用来显示每次提交的内容差异。 你也可以加上 -2 来仅显示最近两次提交
 - git log --stat 媒体提交的简略统计信息
 - --pretty。 这个选项可以指定使用不同于默认格式的方式展示提交历史。 这个选项有一些内建的子选项供你使用。 比如用 oneline 将每个提交放在一行显示，查看的提交数很大时非常有用。 另外还有 short，full 和 fuller 可以用，展示的信息或多或少有些不同，请自己动手实践一下看看效果如何。

#### [remote命令](https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E8%BF%9C%E7%A8%8B%E4%BB%93%E5%BA%93%E7%9A%84%E4%BD%BF%E7%94%A8)

你可以通过 git ls-remote (remote) 来显式地获得远程引用的完整列表，或者通过 git remote show (remote) 获得远程分支的更多信息。 然而，一个更常见的做法是利用远程跟踪分支。

#### tag 打标签
- git tag 列出已有的标签
- git tag -l 'v1.8.5*' 查询1.8.5系列的标签
- 创建标签
    + 轻量标签：git tag v1.4
    + 附注标签：git tag -a v1.4 -m 'my version 1.4'
- 后期补打标签：git tag -a v1.2 9fceb02 最后的参数是要补打标签的那个commitId

#### 共享标签
默认情况下，`git push` 命令并不会传送标签到远程仓库服务器上。 在创建完标签后你必须显式地推送标签到共享服务器上。 这个过程就像共享远程分支一样——你可以运行 `git push origin [tagname]`。

如果想要一次性推送很多标签，也可以使用带有 --tags 选项的 git push 命令。 这将会把所有不在远程仓库服务器上的标签全部传送到那里。
```bash
$ git push origin --tags
```
#### 删除标签

- 删除本地标签 `git tag -d <tagname>`
- 远程标签不会随本地标签删除而删除，你需要同步`git push <remote> :refs/tags/<tagname>` 来更新你的远程仓库标签

### Git分支

Git 保存的不是文件的变化或者差异，而是一系列不同时刻的文件快照。

在进行提交操作时，Git 会保存一个提交对象（commit object）。知道了 Git 保存数据的方式，我们可以很自然的想到——该提交对象会包含一个指向暂存内容快照的指针。 但不仅仅是这样，该提交对象还包含了作者的姓名和邮箱、提交时输入的信息以及指向它的父对象的指针。首次提交产生的提交对象没有父对象，普通提交操作产生的提交对象有一个父对象，而由多个分支合并产生的提交对象有多个父对象，

为了更加形象地说明，我们假设现在有一个工作目录，里面包含了三个将要被暂存和提交的文件。 暂存操作会为每一个文件计算校验和（使用我们在 起步 中提到的 SHA-1 哈希算法），然后会把当前版本的文件快照保存到 Git 仓库中（Git 使用 blob 对象来保存它们），最终将校验和加入到暂存区域等待提交

![提交对象及树结构](./commit-and-tree.png)

 - 98ca9 是一个提交（commit）对象，保存着树对象（文件变更树）的指针和所有提交信息。
 - 92ec2 是上面的树对象，记录着目录结构和文件快照
 - 右边三个是三个blob对象，保存着文件快照

多次修改提交后，那么产生的提交对象会包含一个上次提交对象（父对象）的指针。链式结构

![commit记录](./commits-and-parents.png)

Git 又是怎么知道当前在哪一个分支上呢？ 也很简单，它有一个名为 HEAD 的特殊指针。 请注意它和许多其它版本控制系统（如 Subversion 或 CVS）里的 HEAD 概念完全不同。 **在 Git 中，它是一个指针，指向当前所在的本地分支**（将 HEAD 想象为当前分支的别名）。

```bash
# 等价于下面两条命令
$ git checkout -b iss53

$ git branch iss53
$ git checkout iss53
```

#### fast-forward
你应该注意到了"快进（fast-forward）"这个词。 由于当前 master 分支所指向的提交是你当前提交（有关 hotfix 的提交）的直接上游，所以 Git 只是简单的将指针向前移动。 换句话说，当你试图合并两个分支时，如果顺着一个分支走下去能够到达另一个分支，那么 Git 在合并两者的时候，只会简单的将指针向前推进（指针右移），因为这种情况下的合并操作没有需要解决的分歧——这就叫做 “快进（fast-forward）”。

#### recursive

![recursive](./basic-merging-1.png)

你的开发历史从一个更早的地方开始分叉开来（diverged）。 因为，master 分支所在提交并不是 iss53 分支所在提交的直接祖先，Git 不得不做一些额外的工作。 出现这种情况的时候，Git 会使用两个分支的末端所指的快照（C4 和 C5）以及这两个分支的工作祖先（C2），做一个简单的三方合并。

结果如下：
![recursive result](./basic-merging-2.png)

Git 会自行决定选取哪一个提交作为最优的共同祖先，并以此作为合并的基础；这和更加古老的 CVS 系统或者 Subversion （1.5 版本之前）不同，在这些古老的版本管理系统中，用户需要自己选择最佳的合并基础。 Git 的这个优势使其在合并操作上比其他系统要简单很多。

#### 分支开发工作流
稳定分支的指针总是在提交历史中落后一大截，而前沿分支的指针往往比较靠前。

![分支工作流1](./lr-branches-1.png)

![分支工作流2](./lr-branches-2.png)

### [Git远程分支](https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E8%BF%9C%E7%A8%8B%E5%88%86%E6%94%AF)

---

### 实用技能及rebase

[link1](https://juejin.im/post/5d5b4c6951882569eb570958) [⚠️link2](https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E5%8F%98%E5%9F%BA)

#### rebase的用法和惊艳之处，以及在什么情况下慎用。

**普通的merge操作**

![merge-before](./basic-rebase-1.png)

![merge-after](./basic-rebase-2.png)

整合分支最容易的方法是 merge 命令。 它会把两个分支的最新快照（C3 和 C4）以及二者最近的共同祖先（C2）进行三方合并，合并的结果是生成一个新的快照（并提交）。

**rebase怎么做合并呢**
你可以提取在 C4 中引入的补丁和修改，然后在 C3 的基础上应用一次。 在 Git 中，这种操作就叫做 变基。 
> 你可以使用 rebase 命令将提交到某一分支上的所有修改都移至另一分支上，就好像“重新播放”一样。

```bash
# document
$ git rebase --help
```

原理是首先找到这两个分支（即当前分支 experiment、变基操作的目标基底分支 master）的最近共同祖先 C2，然后对比当前分支相对于该祖先的历次提交，提取相应的修改并存为临时文件，然后将当前分支指向目标基底 C3, 最后以此将之前另存为临时文件的修改依序应用。

![rebase-before](./basic-rebase-1.png)

```bash
$ git checkout experiment
$ git rebase master
First, rewinding head to replay your work on top of it...
Applying: added staged command
```

![rebaseing](./basic-rebase-3.png)

```bash
$ git checkout master
$ git merge experiment
```

![rebase-after](./basic-rebase-4.png)

此时，C4' 指向的快照就和上面使用 merge 命令的例子中 C5 指向的快照一模一样了。 这两种整合方法的最终结果没有任何区别，但是变基使得提交历史更加整洁。 你在查看一个经过变基的分支的历史记录时会发现，尽管实际的开发工作是并行的，但它们看上去就像是串行的一样，提交历史是一条直线没有分叉。



#### rebase的风险

关键点：
**不要对（在你仓库之外有副本的分支）进行变基（rebase）**

`git checkout branch xxx origin/master` 后面的 `origin/master` 代表我们是**基于远程分支** `origin/master` 进行创建的。

```bash
git checkout -b test2 origin/master
Branch 'test2' set up to track remote branch 'master' from 'origin'.
Switched to a new branch 'test2'
```

#### Git cherry-pick

我们可能会有这样一个使用场景，在分支 branch-a 需要分支 branch-b 的某次提交，这个时候我们就可以先找到 branch-b 的那次提交记录的 id，然后在 branch-a 分支进行 git cherry-pick b-commit-id 将 branch-b 分支的提交记录拿过来了
那如果我们只需要 branch-b 分支的某个文件呢该怎么办呢，莫慌再交你一个操作，git checkout xxx file 这个操作可以将其他分支的文件拉取到当前分支，注意这个操作是覆盖式的，也就是如果你这个分支的这个文件已将存在，那么这个操作将会覆盖你当前分支的这个文件。可以发挥一下 xxx 也可以是远程分支。

```bash
$ git checkout branch -- file
```

那么这个操作又是干嘛的呢，这个操作可以将你的某个文件还原到某个分支的版本。如果某次你手误提交了错误的文件，但是改动又忘了，那么这个命令或许可以帮到你。


