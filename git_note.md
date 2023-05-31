# Git 初始化

## 1. 基本设置

```bash
git --version   # 查看Git版本
git config --global user.name "John Kelly"  # 全局配置中设置用户名
git config --global user.email "johnkelly@hotmail.com"  # 全局配置中设置邮件地址
git config --global color.ui true       # Git命令输出中开启颜色显示
git config --unset --global user.name   # 删除Git全局配置文件中的用户名设置
```

## 2. 版本库初始化并完成第一次提交

_进入需版本控制的目录后，执行以下命令_

```bash
git init  # 初始化
git add 文件名  # 也可以使用 . 或 -A 添加当前工作区的全部待加入暂存区文件
git commit -m '提交说明'  # 提交说明是必须的
```

## 3. 其它常用命令

```bash
git grep "工作区文件内容搜索"
git status -s   # 精简格式的状态输出
git log -l  # 查看提交记录
git rev-parse --git-dir # 显示版本库.git目录所在位置
git rev-parse --show-toplever   # 显示工作区根目录
git rev-parse --show-prefix   # 相对于工作区根目录的相对目录
git rev-parse --show-cdup   # 显示从当前目录后退到工作区的根的深度
git commit --allow-empty -m "允许执行空白提交"  # Git默认不会提交工作区未做任何修改的提交
git commit --amend 文件名 -m "更新后的提交信息"   # 对刚刚的提交进行修补
git push origin master  # 推送版本库至服务器
git clone demo demo-copy  # 备份
```

# Git 暂存区

> 暂存区市一个介于工作区和版本库的中间状态，当执行提交时，实际上
> 是将暂存区的内容提交到版本库中，而 Git 的很多命令都会涉及暂存区概念。

## 1. 区分工作区、暂存区和版本库之间文件差异

```bash
git diff            # 工作区与提交暂存区中文件相比的差异
git diff HEAD       # 工作区与HEAD（当前工作分支）中文件相比的差异
git diff --staged   # 提交暂存区和版本库中文件的差异
```

## 2. 搁置问题，暂存状态

```bash
git stash       # 保存当前工作进度，待其它先处理事件完成后再回来
```

# Git 对象

## 1. Git 对象库探秘

```bash
git log -1 --pretty=raw     # 查看日志详尽输出
```

_研究 Git 对象 ID 的一个重量级武器就是 `git cat-file` 命令_

```bash
git cat-file -t id    # 查看对象ID的类型
git cat-file -p id    # 查看对象ID的内容
```

> 通过提交对象之间的相互关联，可以清楚的显示一条跟踪链，可通过
> `git log --pretty=raw --graph id`命令查看。最早的那个
> 提交没有 _parent_ 属性，所以跟踪链到此终结。

```bash
git status -s -b    # 查看工作区和暂存区是否有改动
git branch          # 显示当前的工作分支
git rev-parse master 或 HEAD 或 refs/head/master  # 显示引用对应的提交ID
```

_在当前版本库中，HEAD、master、refs/heads/master 具有相同的指向_

> Git 提供了很多方法可以方便地访问 Git 库中的对象：
>
> - 采用部分的 sha1 哈希值。不必把 40 位的哈希值写全，可采用开头部分只要不与现有的其它哈希值冲突即可。
> - 使用 master，代表分支 master 中最新的提交，也可使用全称 refs/heads/master 或 heads/master。
> - 使用 HEAD，代表版本库中最近的一次提交。
> - 使用符号^可以用于指代父提交。如 HEAD^代表版本库中的上一次提交，即最近一次提交的父提交。
> - 对于一个提交有多个父提交，可以在符号^后面用数字表示是第几个父提交。如 HEAD^2 即相当于 HEAD^^。
> - 符号~<n>也可以用于指代祖先提交。如 a573106~5 即相当于 a573106^^^^^
> - 提交所对应的树对象，可用如下语法访问：a573106^{tree}

# Git 重置

## 1. master 游标探秘

_引用 master 就好像是一个游标，在有新的提交发生时指向新的提交。Git 提供了 `git reset`命令，可以将游标指向任意一个存在的提交 ID_

**慎用下一条命令**

```bash
git reset --hard HEAD^    # 将 master 重置到上一个父提交上，会破坏工作区未提交的改动
```

## 2. 用 reflog 挽救错误的重置

_如果没有记下重置前 master 分支指向的提交 ID，想要重置回原来的提交似乎是一件麻烦的事情。幸好 Git 提供了一个挽救机制，通过 .git/logs 目录下日志文件记录了分支的变更。_

> Git 提供了一个 git reflog 命令，对这个文件进行操作，使用 show 子命令可以显示此文件的内容。

```bash
git reflog show master | head -5
```

_此命令行的输出将最新改变日志放在了最前面显示，还提供了一个方便易记的表达式：<refname>@{<n>}，该表达式的含义是引用<refname>之前第<n>次改变时的哈希值。如，_

```bash
git reset --hard master@{2}
```

_通过此方式，提交历史也就回来了。_

## 3. 深入了解 git reset 命令

_重置命令是 Git 最常用的命令之一，也是最危险最容易误用的命令。_

> 用法一: `git reset [-q] [<commit>] [--] <paths>...`
>
> 用法二: `git reset [--soft | --mixed | --hard] [-q] [<commit>]`

_其中【commit】都是可选项，可以使用引用或提交 ID，如果省略~~commit~~，则相当于使用了 HEAD 的指向作为提交 ID。_

_两种用法的区别在于，第一种用法在命令中包含路径【paths】。为避免路径和引用同名而发生冲突，可在【paths】前用两个连续的短线作为分隔。_

> 第一种用法不会重置引用，更不会改变工作区，而是用指定提交状态【commit】下的文件【paths】替换掉暂存区中的文件。如，`git reset HEAD <paths>` 相当于取消之前执行的 `git add <paths>` 命令时改变的暂存区。
>
> 第二种用法则会重置引用。根据不同的选项，可以对暂存区或工作区进行重置。
>
> - 使用参数 --hard，如 `git reset --hard <commit>`
>
>   1. 替换引用的指向，引用指向新的提交 ID。
>   2. 替换暂存区。替换后，暂存区的内容和引用指向的目录树一致。
>   3. 替换工作区。替换后，工作区的内容变得和暂存区一致，也和 HEAD 所指向的目录树内容相同。
>
> - 使用参数 --soft，如 `git reset --soft <commit>` 即只更改引用的指向，不改变暂存区和工作区。
> - 使用参数 --mixed 或不使用参数，如 `git reset <commit>` 即更改引用的指向及重置暂存区，但不改变工作区。

_重置命令的不同用法_

```bash
git reset   # 仅用 HEAD 指向的目录树重置暂存区，引用也未改变因引用重置到 HEAD，相当于没有重置
git reset HEAD    # 同上
git reset -- filename   # 仅将文件filename的改动撤出暂存区，暂存区中其他文件不变
git reset HEAD filename   # 同上
git reset --soft HEAD^    # 工作区和暂存区不变，但引用向前回退一次。当对最新提交的提交说明或提交的更改不满意时，撤销最新的提交以便重新提交。与 `git commit --amend` 类似，但不同于它。
git reset HEAD^   #  工作区不改变，但是暂存区和引用会回退一次
git reset --mix HEAD^   # 同上
git reset --hard HEAD^    # 彻底撤销最近的提交
```
