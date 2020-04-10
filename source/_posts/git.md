---
title: git
date: 2018-12-17 18:41:40
tags:
    - git
categories: git
---

&emsp;&emsp;git是非常先进的分布式版本控制系统，使用git可以将本地文件托管到github之类的服务器上并进行高效的版本控制。

<!-- more -->

## Git简单配置

### 查看配置信息

```bash
git config --system
git config --global user.name
git config --local --list
```

- 配置优先级
&emsp;&emsp;作用域越小，优先级越高，即`local`的配置会覆盖`global`的配置；`global`的配置会覆盖`system`的配置。

### 用户信息

```bash
git config --global user.name ferry
git config --global user.email ferry_sy@163.com
```

### 记住密码

&emsp;&emsp;有时候我们使用https方式clone项目，这时我们每次`git pull`和`git push`的时候都要求输入用户名密码，这里不提供切换ssh的方式，采用另一个方式，让本地记住你的用户名密码。

```bash
git config --global credential.helper store
```

```bash
git config --global credential.helper cache
```


&emsp;&emsp;这条指令的效果是在.gitconfig文件最后添加以下配置：

```bash
[credential]
helper = store/cache
```

&emsp;&emsp;这时，你在git pull时会要求你输入用户名密码，输入正确后该用户名密码会被记录在本地，下次就不用输入了。store永久存储密码，cache临时记忆密码，可以通过`git config --global credential.helper ‘cache –timeout=3600’`来指定临时记忆密码的时间，单位秒

### 本文编辑器



## git应用

### git init

- 方式一：
```bash
mkdir git_study
cd git_study/
git init
ls -al
```

![初始化git项目方式1](/git/git_init_1.png)


- 方式二：
```bash
git init git_study
cd git_study/
ls -al
```

![初始化git项目方式2](/git/git_init_2.png)

&emsp;&emsp;方式一是先创建项目目录，然后进入项目根目录创建git的管理目录`.git`；方式二是在当前目录创建项目目录的同时在项目根目录中生成`.git`目录。

### git add

- 添加单个或者多个指定文件到暂存区，可以使用通配符进行批量添加

```bash
git add test [test2]
```

- 批量添加已经被git管理的文件到暂存区

```bash
git add -u
```

- 提交所有变化到暂存区

```bash
git add .
```

```bash
git add -A
```

&emsp;&emsp;在 Git2.* 版本中 `git add .` 和 `git add -A` 没有区别，都是提交所有变化到暂存区；但是在1.*版本中 `git add .` 只处理修改的和新增的文件，删除的文件不做处理，`git add -A` 处理所有变化。

### git commit

- 无参数提交

```bash
git commit -m '提交的注释'
```

&emsp;&emsp;使用`git commit`不加任何参数则提交暂存区的变更到本地仓库,`-m`写注释信息，如果使用`git commit`指令的使用没有加`-m`那么会弹出编辑界面，在此界面输入提交信息。需要注意，其他参数需要在`-m`参数之前，`-m`参数后面的内容会被当作注释提交上去。

```bash
git commit -a
```

&emsp;&emsp;使用`git commit -a`将已经被git跟踪的文件的变更提交到暂存区，然后将暂存区的变更提交到本地仓库，此时没有加`-m`参数，所以会弹出编辑界面，在此界面输入提交信息。

```bash
git commit -a -m'提交信息'
git commit -am'提交信息'
```

&emsp;&emsp;将`-a`和`-m`指令结合起来的形式有如上两种。

- `--amend`参数

&emsp;&emsp;使用`--amend`参数表示本次提交是一次增补提交，git会将当前提交清除，在父节点上再此生成一次提交记录。

![增补提交](/git/amend.png)

- 指定文件(目录)提交

```bash
git commit test -m'submit test'
```

### git log

- 无参数查看

```bash
git log
```

![无参数查看历史](/git/git_log_none_param.png)

&emsp;&emsp;按 q 键退出

- 查看指定数量的历史记录

```bash
git log -n5
```

```bash
git log -5
```

- 以单行显示历史记录

```bash
git log --oneline
```

- 查看指定分支历史

```bash
git log master
```

- 查看分支演变历史

```bash
git log --graph
```

- 查看每次提交的文件列表

```bash
git log --stat
```

&emsp;&emsp;更多参数可以参考`git log --help`

- 多个参数结合

```bash
git log --oneline -n10 --all --graph --no-merges
```

![定制化查看历史](/git/git_log_params.png)

### git show

- git show hash值

&emsp;&emsp;hash值对应的类型有三种：

> blob: 查看这个文件的具体内容
> tree: 查看这颗树的目录结构
> commit: 查看本次提交的变更内容

&emsp;&emsp;`git show`后面也可以跟branch/tag，但究其本质还是使用的branch/tag所在的commit_id,所以此时查看的也就是这个branch/tag最新的提交的变更内容。


### gitk

- git图形化工具

```bash
gitk
```

&emsp;&emsp;在gitk中，你可以通过这个图形化工具查看log，diff等信息。

### git reset

&emsp;&emsp;reset总共会有三个步骤，通过不同的指令可以是reset在某一步骤停住脚步从而达到我们要的效果。

1. 更新HEAD，通过参数`--soft`使reset在此停留；

2. 更新暂存区，通过参数`--mixed`使reset在此停留，缺省参数则默认为`--mixed`;

3. 更新工作目录，通过参数`--hard`使reset在此停留，此操作会将工作目录下修改但为提交的内容移除，无法再通过git找回，慎用！！！

&emsp;&emsp;新建一个项目并创建三次提交来演示reset。

![创建项目演示reset](/git/create_test_project.png)

- `git reset --soft commit_id`

```bash
git reset --soft de08188aa
```

![--soft](/git/reset--soft.png)

&emsp;&emsp;此时git将 HEAD 指向 de08188aa 提交上，但是工作区和暂存区不变。

- `git reset --mixed commit_id`

```bash
git reset HEAD
```

![--mixed](/git/reset--mixed.png)

&emsp;&emsp;基于刚才的重置HEAD，此时再重置暂存区，此步操作将暂存区重置，与HEAD保持一致。

- `git reset --hard commit_id`

```bash
git reset --hard HEAD
```

&emsp;&emsp;此时将工作目录重置，保持与暂存区和HEAD的一致。

### git checkout

&emsp;&emsp;`git checkout`和`git reset`功能有一定的相似性，在这里我简单的了解了一下它们的区别。`git checkout`后面跟一个commit_id，缺省的情况下默认为HEAD，在这里有必要说一点，**branch,tag,HEAD本质上也是指向一个具体的提交**，所以我们可以用branch,tag,HEAD替换对应的commit_id。checkout具体的提交那么git会处于分离头指针的状态，chekcout分支/标签会检出到对应的分支/标签的最新的提交上。

> 不带路径: `git checkout commit_id`是检出到指定的提交，将HEAD移动到指定提交上，git将工作区和暂存区未做修改的内容先更新为当前检出的提交版本，然后将修改的文件进行合并更新，如果发生冲突，那么会要求你进行merge操作。checkout并不会修改git提交日志。但是`git reset`不一样，当你加上`--hard`时，它会将工作区，暂存区和HEAD重置到指定的提交，它是会修改git提交日志的。例如，当你执行`git reset HEAD^ --hard`后，工作区，暂存区，HEAD都重置为原HEAD的父节点，此时你查看git提交日志会发现，原提交记录不见了，HEAD指向最新的提交也就是原HEAD的父节点。

> 带路径: `git checkout commit_id`是检出指定文件的指定的提交版本。

![检出](/git/git_checkout.png)

### git branch

- 新建分支

```bash
git branch tmp HEAD
```

![新建分支](/git/git_branch_new.png)

&emsp;&emsp;语法为`git branch 分支名 commit_id`，缺省commit_id默认为HEAD。

- 查看本地所有分支

```bash
git branch
```

![查看分支](/git/git_branch.png)

&emsp;&emsp;分支名前有`*`号的就是当前HEAD所在分支。

- 查看本地和远端的所有分支

```bash
git branch -a
```

![查看分支](/git/git_branch-a.png)

- 查看分支并显示分支最新的提交信息

```bash
git branch -v
```

![查看分支](/git/git_branch-v.png)

- 显示其他与当前分支合并关系

```bash
git branch --merged
git branch --no-merged
```

![查看分支](/git/git_branch-merged.png)

- 删除分支

```bash
git branch -d tmp
git branch -D tmp
```

![删除分支](/git/git_branch-d.png)

&emsp;&emsp;在删除分支的时候，如果git检测出待删除的分支上还有提交没有合并到其他分支，那么就会弹出提示，如果一定要删除，那么使用-D强制删除该分支。

### git reflog

&emps;&emsp;`git reflog`记录了HEAD的所有变化，而`git log`显示的是但不包括删除的提交，所以如果误操作删除了某次提交记录，那么可以使用`git reflog`找到这次的提交，然后使用`git reset`或者`git cherry-pick`恢复这次提交。`git reflog`参数查询和`git log`差不多，基本可以通用，这里简单记录一下怎么恢复误删除的提交。

&emsp;&emsp;在这里先创建三次提交，然后使用reset回退到第二次提交，然后恢复。

![演示背景](/git/git-reflog-bg.png)

```bash
git reflog
git checkout HEAD@{0}
```

![演示](/git/git-reflog-checkout.png)

&emsp;&emsp;`git reflog`查看HEAD的变动记录，**最前面的一段哈希值是本次操作返回的HEAD所指向的commit_id；括号中的是返回的提交所处的分支或者标签位置；HEAD@{n}表示HEAD更改历史记录,可以通过HEAD@{n}引用存在reflog中的提交；后面的内容则是展示HEAD变更的行为**。使用`git checkout`检出到指定的提交上，此时处于分离头指针状态。

&emsp;&emsp;使用reflog命令查看到HEAD的所有变化后，可以使用reset命令直接将HEAD恢复到指定的提交上，如果所在分支后续又进行了开发，然后想将之前的某次提交内容还原，那么就可以使用cherry-pick命令将提交挑拣到当前分支。

### git merge

&emsp;&emsp;`git merge`用于将两个或多个提交合并在一起。

&emsp;&emsp;**fast-forward**：现有分支master，上面有提交记录A，现基于A创建一个分支develop，在develop上创建提交B，master未做任何修改。此时，将develop分支合并到master分支时会将master分支的文件指针直接指向提交B。这一过程成为`fast-forward`，触发`fast-forward`默认不会创建一个合并提交。

- 简单合并

```bash
git merge commit_id
```

&emsp;&emsp;不使用任何参数时，将会弹出vi编辑器，可编辑本次合并的提交信息，保存后会自动提交本次合并的结果。

- 取消合并

```bash
git merge --abort
```

&emsp;&emsp;在发生冲突的时候需要手动解决冲突然后再提交。这里可以使用`--abort`参数取消本次合并，将HEAD努力的还原到合并前的状态。但是，如果在合并前暂存区还有未提交的内容，那么冲突可能会修改未提交的内容，那么在还原的时候可能就无法重现合并前的状态。

- 合并不自动提交

```bash
git merge commit_id --no-commit
```

&emsp;&emsp;`--no-commit`参数将是合并的脚步停止在提交前，即本次合并的结果将在HEAD中；缺省时默认为`--commit`，而且`--commit`会覆盖`--no-commit`。

- 编辑合并提交信息

```bash
git merge commit_id -m'merge branch dev' 
```

&emsp;&emsp;`-m'merge infomation'`参数编辑合并成功提交时的提交信息

```bash
git merge commit_id --no-edit
```

&emsp;&emsp;`--no-edit`参数指定合并成功后提交时不需要编辑提交信息，使用默认的提交信息。

```bash
git merge commit_id --edit
```

&emsp;&emsp;`--edit`或者`-e`参数表明合并成功后提交将进入vi编辑器，编辑提交信息，保存后提交成功，缺省时默认为此参数

```bash
git merge commit_id -m'merge infomation' --edit
```

&emsp;&emsp;`-m'merge infomation' --edit`参数编辑合并成功提交时的提交信息，并在提交时进入vi编辑器进一步编辑`-m`所含提交信息。

- 快进合并

```bash
git merge commit_id --ff
```

&emsp;&emsp;`--ff`采用`fast-forward`模式合并，缺省时默认为此值。`--no-ff`表明即使能够使用`fast-forward`模式合并也会创建一个新的合并提交节点，`git merge`合并tag是默认为此行为。`--ff-only`除非当前HEAD节点已经up-to-date（更新指向到最新节点）或者能够使用`fast-forward`模式进行合并，否则的话将拒绝合并，并返回一个失败状态。

- 合并提交时合并提交信息

```bash
git merge commit_id --log
```

&emsp;&emsp;`--log`合并分支时创建的合并提交节点的提交信息将包含合并的分支名，以及合并分支的每个不在当前分支的提交的提交信息。`--no-log`不合并提交信息，缺省时的默认选项。

- 合并显示文件差异

```bash
git merge commit_id --stat
```

&emsp;&emsp;`--stat`在合并完成后在shell上显示本次合并的文件差异状态，缺省时的默认选项。`--no-stat`或`-n`不显示合并的文件差异状态。

- 压缩合并

```bash
git merge commit_id --squash
```

&emsp;&emsp;`--squash`将另一分支上 从两个分支共同祖先节点的下一个节点开始，至最新的节点 压缩成一个节点然后提交，和`--no-ff`参数冲突。`--no-squash`作用相反，缺省时的默认选项。

&emsp;&emsp;[merge合并详解](https://www.jianshu.com/p/58a166f24c81)

### git stash

&emsp;&emsp;`stash`可以获取你工作目录的中间状态——也就是你修改过的被追踪的文件和暂存的变更——并将它保存到一个未完结变更的堆栈中，随时可以重新应用。在实际工作中，我们可能有各种各样的原因不得不先切换分支去完成别的任务，但是此时又不想将完成部分的修改提交，此时就可以使用`git stash`将修改暂存起来，等完成了别的任务再切换回来然后还原我们暂存的修改内容继续工作。

&emsp;&emsp;为了演示，这里创建一个文件，修改一个文件，再`git add`一个修改内容。

![修改内容](/git/git_stash_init.png)

&emsp;&emsp;此时，我们要去修改另一个分支的bug，但是又不想提交修改或者放弃本次修改，那就暂存起来，等改完bug回来可以继续工作。

- 暂存

```bash
git stash
```

![暂存结果](/git/git_stash_status.png)

&emsp;&emsp;shell会输出一句内容，告知工作目录和index已经保存了，此时再`git status`查看会发现工作目录和index是干净的，但是我们新建的文件并没有暂存起来，这是因为新建的文件是untracked的，也就是未被git跟踪的，所以它不会被stash。

### git rebase

### git cherry-pick



