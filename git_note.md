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
