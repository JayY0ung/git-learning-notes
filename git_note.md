# Git 初始化

## 1. 基本设置

```bash
git --version # 查看Git版本
git config --global user.name "John Kelly"  # 全局配置中设置用户名
git config --global user.email "johnkelly@hotmail.com"  # 全局配置中设置邮件地址
git config --global color.ui true # Git命令输出中开启颜色显示
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

## 1. 区分工作区、暂存区和版本库之间文件差异

```bash
git diff    # 工作区与提交暂存区中文件相比的差异
git diff HEAD     # 工作区与HEAD（当前工作分支）中文件相比的差异
git diff --staged   # 提交暂存区和版本库中文件的差异
```

## 2. 搁置问题，暂存状态

```bash
git stash     # 保存当前工作进度，待其它先处理事件完成后再回来
```
