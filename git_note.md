# Git 初始化

## 1. 基本设置

```bash
# 查看Git版本
git --version

# 全局配置中设置用户名
git config --global user.name "John Kelly"

# 全局配置中设置邮件地址
git config --global user.email "johnkelly@hotmail.com"

# Git命令输出中开启颜色显示
git config --global color.ui true

# 删除Git全局配置文件中的用户名设置
git config --unset --global user.name
```

## 2. 版本库初始化并完成第一次提交

_进入需版本控制的目录后，执行以下命令_

```bash
# 初始化
git init

# 也可以使用 . 或 -A 添加当前工作区的全部待加入暂存区文件
git add 文件名

# 提交说明是必须的
git commit -m '提交说明'
```

## 3. 其它常用命令

```bash
git grep "工作区文件内容搜索"

# 精简格式的状态输出
git status -s

# 查看3条提交记录
git log -3

# 显示版本库.git目录所在位置
git rev-parse --git-dir

# 显示工作区根目录
git rev-parse --show-toplevel

# 相对于工作区根目录的相对目录
git rev-parse --show-prefix

# 显示从当前目录后退到工作区的根的深度
git rev-parse --show-cdup

# Git默认不会提交工作区未做任何修改的提交
git commit --allow-empty -m "允许执行空白提交"

# 对刚刚的提交进行修补
git commit --amend 文件名 -m "更新后的提交信息"

# 推送版本库至服务器
git push origin master

# 备份
git clone demo demo-copy
```

# Git 暂存区

> 暂存区是一个介于工作区和版本库的中间状态，当执行提交时，实际上是将暂存区的内容提交到版本库中，而 Git 的很多命令都会涉及暂存区概念。

## 1. 区分工作区、暂存区和版本库之间文件差异

```bash
# 工作区与提交暂存区中文件相比的差异
git diff

# 工作区与HEAD（当前工作分支）中文件相比的差异
git diff HEAD

# 提交暂存区和版本库中文件的差异
git diff --staged
```

## 2. 搁置问题，暂存状态

```bash
# 保存当前工作进度，待其它先处理事件完成后再回来
git stash
```

# Git 对象

## 1. Git 对象库探秘

```bash
# 查看日志详尽输出
git log -1 --pretty=raw
```

_研究 Git 对象 ID 的一个重量级武器就是 `git cat-file` 命令_

```bash
# 查看对象ID的类型
git cat-file -t id

# 查看对象ID的内容
git cat-file -p id
```

> 通过提交对象之间的相互关联，可以清楚的显示一条跟踪链，可通过
> `git log --pretty=raw --graph id`命令查看。最早的那个提交没有 _parent_ 属性，所以跟踪链到此终结。

```bash
# 查看工作区和暂存区是否有改动
git status -s -b

# 显示当前的工作分支
git branch

# 显示引用对应的提交ID
git rev-parse master 或 HEAD 或 refs/head/master
```

_在当前版本库中，HEAD、master、refs/heads/master 具有相同的指向_

> Git 提供了很多方法可以方便地访问 Git 库中的对象：
>
> - 采用部分的 sha1 哈希值。不必把 40 位的哈希值写全，可采用开头部分只要不与现有的其它哈希值冲突即可。
> - 使用 master，代表分支 master 中最新的提交，也可使用全称 refs/heads/master 或 heads/master。
> - 使用 HEAD，代表版本库中最近的一次提交。
> - 使用符号^可以用于指代父提交。如 HEAD^代表版本库中的上一次提交，即最近一次提交的父提交。
> - 对于一个提交有多个父提交，可以在符号^后面用数字表示是第几个父提交。如 HEAD^^2 即相当于 HEAD^的多个父提交中的第二个父提交。
> - 符号~<n>也可以用于指代祖先提交。如 a573106~5 即相当于 a573106^^^^^
> - 提交所对应的树对象，可用如下语法访问：a573106^{tree}

# Git 重置

## 1. master 游标探秘

_引用 master 就好像是一个游标，在有新的提交发生时指向新的提交。Git 提供了 `git reset`命令，可以将游标指向任意一个存在的提交 ID_

**慎用下一条命令**

```bash
# 将 master 重置到上一个父提交上，会破坏工作区未提交的改动
git reset --hard HEAD^
```

## 2. 用 reflog 挽救错误的重置

_如果没有记下重置前 master 分支指向的提交 ID，想要重置回原来的提交似乎是一件麻烦的事情。幸好 Git 提供了一个挽救机制，通过 .git/logs 目录下日志文件记录了分支的变更。_

> Git 提供了一个 git reflog 命令，对这个文件进行操作，使用 show 子命令可以显示此文件的内容。

```bash
git reflog show master | head -5
```

_此命令行的输出将最新改变日志放在了最前面显示，还提供了一个方便易记的表达式：`<refname>@{<n>}`，该表达式的含义是引用`<refname>`之前第`<n>`次改变时的哈希值。如，_

```bash
git reset --hard master@{2}
```

_通过此方式，提交历史也就回来了。_

## 3. 深入了解 git reset 命令

_重置命令是 Git 最常用的命令之一，也是最危险最容易误用的命令。_

> 用法一: `git reset [-q] [<commit>] [--] <paths>...`
>
> 用法二: `git reset [--soft | --mixed | --hard] [-q] [<commit>]`

_其中 commit 都是可选项，可以使用引用或提交 ID，如果省略~~commit~~，则相当于使用了 HEAD 的指向作为提交 ID。_

_两种用法的区别在于，第一种用法在命令中包含路径 paths。为避免路径和引用同名而发生冲突，可在 paths 前用两个连续的短线作为分隔。_

> 第一种用法不会重置引用，更不会改变工作区，而是用指定提交状态 commit 下的文件 paths 替换掉暂存区中的文件。如，`git reset HEAD <paths>` 相当于取消之前执行的 `git add <paths>` 命令时改变的暂存区。
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
# 仅用 HEAD 指向的目录树重置暂存区，引用也未改变因引用重置到 HEAD，相当于没有重置
git reset

 # 同上
git reset HEAD

# 仅将文件filename的改动撤出暂存区，暂存区中其他文件不变
git reset -- filename

# 同上
git reset HEAD filename

# 工作区和暂存区不变，但引用向前回退一次。当对最新提交的提交说明或提交的更改不满意时，撤销最新的提交以便重新提交。与 `git commit --amend` 类似，但不同于它。
git reset --soft HEAD^

# 工作区不改变，但是暂存区和引用会回退一次
git reset HEAD^

# 同上
git reset --mix HEAD^

# 彻底撤销最近的提交
git reset --hard HEAD^
```

# Git 检出

_Git 检出命令的实质就是修改 HEAD 本身的指向，而不影响分支“游标”_

## 1. HEAD 的重置即检出

```bash
# 检出到该ID，此刻会导致分离头指针
git checkout ID
```

> 什么叫“分离头指针”？
>
> 分离头指针状态指的就是 HEAD 头指针指向了一个具体的提交 ID，而不是一个引用分支。

_`git checkout` 和 `git reset` 命令不同，分支的指向并没有改变，仍旧指向原有的提交 ID。_

> 以下例子说明检出的一些特征和特点

```bash
# 检出至父提交
git checkout HEAD^

# 该结果显示发现HEAD与master指向不同
git rev-parse HEAD master

touch detached-commit.txt
git add detached-commit.txt
git commit -m "commit in detached HEAD mode."

# 此时可看到新的提交是建立在之前头指针的提交基础上的
cat .git/HEAD
```

_由于这个提交没有被任何分支跟踪到，因此并不能保证这个提交会永久存在_

## 2. 挽救分离头指针

```bash
# 将不处于任何分支下的提交合并到当前分支中，也保证了当前分支原先提交进入版本库管理
git merge ID
```

## 3. 深入了解 git checkout 命令

> 检出命令用法如下：
>
> 用法一： `git checkout [<commit>] [--] <paths>...`
>
> 用法二： `git checkout [<branch>]`
>
> 用法三： `git checkout [-b <new_branch>] [<start_point>]`

> 第一种用法不会改变 HEAD 头指针，但 commit 是可选项。如果省略则相当于从暂存区进行检出，覆盖工作区。如果不省略，该 commit 会覆盖对应文件的工作区和暂存区。这个和重置命令不同，重置到默认值是 HEAD，而检出到默认值是暂存区。
>
> 第二种用法则会改变 HEAD 头指针，后面带参数 branch 表示切换到一个分支，只有这样才可以对提交进行跟踪，否则仍然进入“分离头指针”状态。此方法最主要的作用是切换到分支。如果省略 branch 参数，则相当于对工作区进行状态检查。
>
> 第三种用法主要是创建和切换到新的分支，新的分支从 start_point 指定的提交开始创建。

_以下命令非常危险，慎用！_

```bash
# 暂存区中filename文件覆盖工作区的filename文件，导致本地的修改会悄无声息的覆盖
git checkout -- filename

# 用branch所指向的提交中的filename替换暂存区和工作区中相应的filename文件
git checkout branch -- filename

# 用暂存区的所有文件覆盖本地文件，不给确认机会！
git checkout -- .
```

# 恢复进度

## 1. 继续暂存区未完成的实践

```bash
# 查看保存的进度
git stash list

# 从最近保存的进度恢复
git stash pop

# 删除本地多余的(未被跟踪的)目录和文件，测试避免误删
git clean -nd

# 强制删除本地多余的(未被跟踪的)目录和文件
git clean -fd
```

## 2. 使用 `git stash`

```bash
# 保存当前工作进度，分别对暂存区和工作区的状态进行保存
git stash

# 不使用任何参数，会恢复最新保存的工作进度，并删除恢复的进度列表。如果提供stash参数，则从该stash中恢复，并从进度列表中删除stash
git stash pop [--index] [<stash>]

# 对保存工作进度的指定说明，便于通过进度列表找到保存的进度。必须如下
git stash save 'message'

# 除了不删除恢复的进度之外，其余和 `git stash pop`命令一样
git stash apply [--index] [<stash>]

# 删除一个存储的进度，默认删除最新的进度
git stash drop [<stash>]

# 删除所有存储的进度
git stash clear

# 基于进度创建分支
git stash branch <branchname> <stash>
```

## 3. 探秘 `git stash`

_没有被版本控制系统跟踪的文件并不能保存进度。_

_`git stash`就是用引用和引用变更日志(reflog)来实现的_

```bash
ls -l .git/refs/stash .git/logs/refs/stash
```

_提交说明中有 WIP（work in progress），代表了工作区进度。而有 index on master 字样的提交，代表了暂存区的进度。每个进度的标识都是 `stash@{<n>}`_

# Git 基本操作

_现实中需应对各种情况，如，文件删除、文件复制、文件移动、目录的组织、二进制文件、误删文件的恢复_

## 1. 合影留恋

_在与之前实践遗留的数据告别前，可合影留恋。在 Git 里，留影用的命令叫做 tag，更专业的术语叫做“里程碑”。留过影之后，可以执行 `git describe` 命令将最新提交显示为一个易记的名称。_

```bash
# 打标签，即里程碑
git tag -m "message" tag-name

# 里程碑无非也是一个引用，通过创建tag对象来为当前版本库的状态进行“留影”
ls .git/refs/tags/tag-name
git rev-parse refs/tags/tag-name
```

## 2. 删除文件

_本地删除不是真的删除_

```bash
# 直接在工作区删除文件
rm filename

# 通过以下命令，可以看到在暂存区、版本库中的文件仍然存在，并未删除
git ls-files
```

_直接在工作区删除，对暂存区和版本库没有任何影响_

```bash
# 执行 git rm 命令删除文件，删除动作加入了暂存区
git rm filename1 filename2 filename3...

# 再执行提交后，对删除进行版本控制
git commit -m "delete trash files"
```

_不用担心，文件只是在本版本库的最新提交中被删除了，在历史提交中尚在。可通过以下命令查看历史版本的文件列表_

```bash
git ls-files --with-tree=HEAD^

# 查看历史版本中尚在的删除文件的内容
git cat-file -p HEAD^:welcome.txt
```

## 3. 命令 git add -u 快速标记删除

_因为命令`git rm`后面需写下所有要删除的文件名，太长了。我们可使用`git add -u`命令，将本地有改动（包括修改和删除）的文件标记到暂存区_

## 4. 恢复删除的文件

```bash
# 以下3条命令都可以完成删除文件的恢复，最后一条最为简洁实用
git cat-file -p HEAD~1:welcome.txt > welcome.txt

git show HEAD~1:welcome.txt > welcome.txt

git checkout HEAD~1 -- welcome.txt
```

```bash
# 将工作区中的所有改动及新增文件添加到暂存区
git add -A

# 执行提交操作，文件welcome.txt又回来了
git commit -m "restore file back"
```

## 5. 移动文件

```bash
# 改名，改名操作相当于对旧文件执行删除，对新文件执行添加。==> `mv old new; git rm old; git add new`
git mv old new
```

## 6. 显示留影后版本号

```bash
# 查看版本号
git describe

# 查看提交日志中显示提交对应的里程碑（tag）
git log --oneline --decorate
```

_命令`git describe`的输出可以作为软件版本号，这个功能非常有用。因为这样可以很容易地实现将发布的软件包版本和版本库中的代码对应在一起，当发现软件包包含 bug 时，可以最快、最准确地对应到代码上。_

## 7. 使用`git add -i`命令选择性添加

```bash
# 执行以下命令，进入一个交互式界面，首先显示的是工作区状态
git add -i
```

## 8. 文件忽略

_在版本库下创建一个.gitignore 文件，把需要忽略的文件写在其中，文件中可使用通配符。文件.gitignore 的作用范围是其所处的目录及其子目录。_

_文件忽略有错误，后果很严重_

```bash
# 只有使用了--ignored参数，才会在状态显示中看到被忽略的文件
git status --ignored -s

# 被忽略的文件使用`git add -A`和`git add .`都失效。只有在添加操作的命令行中明确地写入文件名，并且提供-f参数才能真正添加
git add -f filename_ignored
git commit -m "message"
```

_忽略只对未跟踪文件有效，对于已加入版本库的文件无效。文件.gitignore 设置的文件忽略是共享式的，是因为.gitignore 被添加到版本库后成为了版本库的一部分，当版本库共享给他人（如 clone）或推送（push）至集中式服务器时，这个忽略文件就会出现在他人的工作区中，同样生效。_

_Git 可设置独享式忽略_

> 独享式忽略的两种方式：
>
> 一种是针对具体版本库的“独享式”忽略，即在版本.git 目录下的一个文件.git/info/exclude 来设置文件忽略。另一种是全局的“独享式”忽略，即通过 git 的配置变量 core.excludesfile 指定的一个忽略文件，其设置的忽略对所有本地版本库均有效。

_Git 忽略语法_

- 忽略文件中的空行或以#开头的行会被忽略。
- 可以使用通配符，如（\*）代表任意多字符，（❓）代表一个字符，方括号代表可选字符范围。
- 如果名称的最前面是一个路径分隔符（/），表明要忽略的文件在此目录下，而非子目录的文件。
- 如果名称的最后面是一个路径分隔符（/），表明要忽略的是整个目录，同名文件不忽略，否则同名的文件和目录都忽略。
- 通过在名称的最前面添加一个感叹号（！），代表不忽略。

# 历史穿梭

## 1. 版本表示法 git rev-parse

_命令`git rev-parse`是 Git 的一个底层命令，其功能非常丰富，很多 Git 脚本或工具都会用到这条命令。_

```bash
# 显示当前版本
git describe

# 上一条命令的输出也可以解析出正确的哈希值
git rev-parse （git describe的输出）

# 里程碑 A 实际指向的是一个 Tag 对象而非提交 ID
git rev-parse A refs/tags/A

# 显示提交本身的哈希值而非里程碑对象 A 的哈希值
git rev-parse A^{} A^0 A^{commit}

# 显示里程碑 A 对应的目录树，以下两种写法都可
git rev-parse A^{tree} A:

# 显示树里面的文件，以下两种写法均可
git rev-parse A^{tree}:src/Makefile A:src/Makefile

# 暂存区里的文件和头指针中的文件相同
git rev-parse :gitg.png HEAD:gitg.png

# 通过在提交日志中查找字符串的方式显示提交
git rev-parse :/"Commit A"
```

## 2. 版本范围表示法 git rev-list

```bash
# 一个提交 ID 实际上就代表一个版本列表，含义是该版本开始的所有历史提交
git rev-list --oneline A

# 两个或多个版本，相当于每个版本单独使用时指代的列表的并集
git rev-list --oneline D F

# 在一个版本前面加上符号 ^ 含义是取反，即排除这个版本及其历史版本，参数顺序不重要
git rev-list --oneline ^G D
git rev-list --oneline D ^G

# 功能同上。注意 .. 表示法前后的版本顺序很重要，以下两条命令结果截然不同！
git rev-list --oneline G..D
git rev-list --oneline D..G

# 三点表示法，相当于两个版本共同能够访问到的除外
git rev-list --oneline B...C

# 某提交的历史提交，自身除外，用语法r1^@
git rev-list --oneline B^@

# 提交本身不包括其历史提交，用r1^!
git rev-list --oneline B^!
```

## 3. 浏览日志 git log

```bash
# 显示最近的几条日志，n代表显示数量
git log -n --oneline

# 显示每次提交的具体改动
git log -p -1

# 显示每次提交的变更概要。在不需要知道具体改动而只想知道改动在哪些文件时可使用 --stat 参数
git log --stat --oneline I..C

# 查看和分析某一个提交，也可使用`git show`或`git cat-file`命令
git show D
git cat-file -p D^0
```

## 4. 差异比较 git diff

```bash
# 比较里程碑 A 和 B
git diff A B

# 显示不同版本间该路径下文件的差异
git diff commit1 commit2 -- filename

# Git 版本库之外的差异性比较，可比较二进制文件，非常强大！
git diff path1 path2

# 差异比较默认为逐行比较，使用 --word-diff 参数可逐词比较
git diff --word-diff
```

## 5. 文件追溯 git blame

_在软件开发过程中，当发现 bug 并定位到具体的代码时，Git 的文件追溯命令 `git blame` 可以指出是谁在什么时候，以及什么版本引入的此 bug_

```bash
# 逐行显示文件，并在每一行行首显示此行最早是在什么版本引入的，由谁引入的
git blame filename

# 查看某几行，使用 -L n,m 参数
git blame -L 6,5 README
```

## 6. 二分查找 git bisect

_前面的文件追溯是建立在 bug 已经定位的基础之上进行的。二分查找应用于坏版本与好版本之间的某次提交上出了问题，建立在测试的基础上的。_

> Git 提供的 `git bisect`命令是基于版本库的，自动化的问题查找和定位工具，取代传统软件测试中针对软件发布版本的、无法定位到代码的、粗放式的测试方法。
>
> 二分查找流程
>
> 1. 工作区切换到已知的“好版本”和“坏版本”的中间一个版本。
> 2. 执行测试，如果问题重现，则将版本库当前版本标记为“坏版本”，如果问题没有重现，则将当前版本标记为“好版本”。
> 3. 重复 1-2，直至最终找到第一个导致问题出现的版本。

_用代码来演示二分查找过程。该演示中，如 doc 目录中含文件 B.txt，表面此版本是“坏版本”，否则为“好版本”_

```bash
# 切换至 master 分支，这里是要测试的有问题的分支
git checkout master

# 开始二分查找
git bisect start

# 再次确认坏提交和好提交，以便标记正确
git cat-file -t master:doc/B.txt    # 输出 blob
git cat-file -t G:doc/B.txt         # 输出未找到该文件

# 将当前版本（HEAD）标记为“坏提交”，将 G 版本标记为“好提交”
git bisect bad
git bisect good G

# 自动定位到 C 提交。测试是否存在文件 B.txt
git describe    # 输出 C
ls doc/B.txt    # 输出未找到该文件

# 标记当前版本（即 C 提交）为“好提交”
git bisect good

# 自动定位到 D 提交，测试后这也是一个“好提交”，标记当前版本（ D提交 ）为“好提交”
git bisect good

# 自动定位到 B 提交，这是一个“坏提交”
git bisect bad

# 自动定位到 E 提交，这是一个“好提交”。当标记 E 为“好提交”后，输出显示已成功定位到引入坏提交的最接近版本
git bisect good

# 最终定位的坏提交用引用 refs/bisect/bad 标识，用以下命令切换到该版本
git checkout bisect/bad

# 对bug定位和修复后，撤销二分查找在版本库中遗留的临时文件和引用。撤销二分查找后，版本库切换回执行二分查找之前所在的分支
git bisect reset
```

_“好提交”标记成了“坏提交”，反之亦然，导致前面的查找过程前功尽弃，为此 Git 的二分查找提供了一个恢复查找进度的解决办法，操作如下：_

```bash
# 首先将二分查找日志保存在logfile文件中
git bisect log > logfile

# 编辑这个logfile文件，删除记录了错误动作的行，以井号开始的行是注释，并保存退出
vim logfile

# 结束正在进行的出错的二分查找
git bisect reset

# 通过日志文件logfile恢复进度，重启二分查找
git bisect replay logfile

# 此刻可以查看到再次回到标记错误前的位置了
git describe
```

# 改变历史

应用场景：提交 tag 如图：A -> B -> C -> D -> E -> F

## 时间旅行一

第一幕：干掉 D，并将 E 和 F 重新嫁接到 C 上

> 拣选指令 - `git cherry-pick`，其含义是从众多的提交中挑选出一个提交应用在当前的工作分支中。该命令需要提供一个提交 ID 作为参数，操作过程相当于将该提交导出为补丁文件，然后在当前 HEAD 上重放，形成无论内容还是提交说明都一致的提交。

```bash
# 干掉 D
git checkout C

# 执行拣选操作将E提交在当前HEAD上重放
git cherry-pick E

# 执行拣选操作将F提交在当前HEAD上重放
git cherry-pick F

# 将master分支重置到新的提交ID（即拣选后的最新提交上）
git checkout master
git reset --hard HEAD@{1}
```

在进入第二幕之前记得重新布置舞台

```bash
git reset --hard F
```

第二幕：坏蛋 D 被感化，融入社会

```bash
git checkout D

# 悔棋两次，以便将 C 和 D 融合
git reset --soft HEAD~2

# 执行提交，提交说明重用 C 提交的提交说明
git commit -C C

# 执行拣选操作将E提交在当前HEAD上重放
git cherry-pick E

# 执行拣选操作将F提交在当前HEAD上重放
git cherry-pick F

# 将master分支重置到新的提交ID（即拣选后的最新提交上）
git checkout master
git reset --hard HEAD@{1}
```

## 时间旅行二

> 命令 `git rebase` 是对提交执行变基操作，即可以实现将指定范围的提交“嫁接”到另外一个提交之上。命令格式 `git rebase --onto <newbase> <since> <till>`
>
> 变基的操作过程：
>
> 1. 首先会执行 `git checkout <till>`
> 2. 将 `<since>..<till>` 所标识的提交范围写到一个临时文件中
> 3. 将当前分支强制重置到 `<newbase>`
> 4. 从保存在临时文件中的提交列表中，将提交逐一按顺序重新提交到重置之后的分支上
> 5. 如果遇到提交已经在分支中包含，则跳过该提交
> 6. 如果在提交过程中遇到冲突，则变基过程暂停。用户解决冲突后，执行 `git rebase --continue` 继续变基操作。或者执行 `git rebase --skip` 跳过此提交。或者执行 `git rebase --abort` 就此终止变基操作切换到变基前的分支上。

第一幕：干掉 D，并将 E 和 F 变基到 C 上

```bash
# 变基操作，并将 master 分支指向变基后的提交上
git rebase --onto C D
git reset --hard HEAD@{1}

# 重新布置舞台
git checkout master
git rest --hard F
```

第二幕：坏蛋 D 被感化，融入社会

```bash
git checkout D
git reset --soft HEAD^^
git commit -C C

# 记住这个提交ID的最好方法，用里程碑
git tag newbase

# 执行变基操作，将 E 和 F 提交嫁接到 newbase 上
git rebase --onto E^ master

# 删除刚创建的tag
git tag -d newbase
```

## 时间旅行三

执行交互式变基操作，会将 `<since>..<till>` 的提交悉数罗列在一个文件中，然后自动打开一个编辑器来编辑这个文件。

第一幕：干掉 D

```bash
# 执行交互式变基操作
git rebase -i D^
```

自动用编辑器修改文件，删除需去除的提交。保存退出即可完成变基操作。

第二幕：坏蛋 D 被感化，融入社会

```bash
git rebase -i C^
```

自动用编辑器修改文件，修改第二行（提交 D），将动作由 pick 修改为 squash。保存退出，自动开始变基操作，在执行到 squash 命令设定的提交时，进入提交前的日志编辑状态（显示 C 和 D 提交说明在一起了）。再次保存退出，即完成 squash 动作标识的提交合并及后续变基操作。

## 丢弃历史

思路：如果希望把里程碑 A 之前的历史提交全部清除，可以这样做。基于里程碑 A 对应的提交构造一个根提交（即没有父提交的提交），然后再将 master 分支在 A 之后的提交变基到新的根提交上，实现对历史提交的清除。

由里程碑 A 对应的提交构造出一个根提交至少有两种方法：

```bash
# 第一种方法使用 `commit-tree` 命令
echo "Commit from tree of tag A." | git commit-tree A^{tree}

# 第二种方法使用 `git hash-object` 命令
# 查看里程碑A指向的提交
git cat-file commit A^0

# 将上面的输出过滤掉以 parent 开头的行，并将结果保存到一个文件中
git cat-file commit A^0 | sed -e '/^parent/ d' > tmpfile

# 运行 `git hash-object` 命令，将文件 tmpfile 作为一个 commit 对象写入对象库
git hash-object -t commit -w -- tmpfile

# 执行变基，将 master 分支里程碑 A 之后的提交全部迁移到根提交上
git rebase --onto <root-commit> A master
```

## 反转提交

对历史的修改，对于一个人使用 Git 没有问题，但是如果多人协同就会有问题了。更改历史操作只能是针对自己的版本库，而无法修改他人的版本库。在这种情况下要想修正一个错误历史提交的正确做法是反转提交，即重新做一次新的提交，相当于用错误的历史提交来修正错误的历史提交。

```bash
git revert <commit-ID>
```

# Git 克隆

> Git 使用 `git clone` 命令实现版本库克隆，主要有如下三种用法：
>
> 用法一： `git clone <repository> <directory>`
>
> 用法二： `git clone --bare <repository> <directory.git>`
>
> 用法三： `git clone --mirror <repository> <directory.git>`
>
> 用法 1 将 `<repository>` 指向的版本库创建一个克隆到 `<directory>` 目录。该目录相当于克隆版本库的工作区，文件都会检出，版本库位于工作区下的.git 目录中。
>
> 用法 2 和用法 3 创建的克隆版本库都不包含工作区，直接就是版本库的内容，这样的版本库称为裸版本库。一般约定俗成裸版本库的目录名以.git 为后缀。他们之间的区别是，用法 3 克隆出来的裸版本对上游版本库进行了注册，可以在裸版本库中使用 `git fetch` 命令和上游版本库进行持续同步。

## 创建生产裸版本库

之前执行 `git init` 命令初始化的版本库是带工作区的，我们可以使用 `git init --bare` 命令创建一个空的裸版本库。

```bash
# 创建裸版本库
git init --bare /path/to/repos/demo-init.git

# 空版本库没有内容，需通过 push 操作为其创建内容。注意，一定要带上分支名，否则无法推送
git push /path/to/repos/demo-init.git master:master
```

此后的推送可以不带分支名了，因为远程版本库 `demo-init.git` 已经不再是空版本库了。

# Git 库管理

`git prune` 命令清除暂存区操作时引入的临时对象，对于使用重置命令抛弃的提交和文件需用 `git reflog` 命令中提供的 `--expire=<date>` 参数，强制让 `<date>` 之前的记录全部过期，会使得提交对应的 commit 对象、tree 对象和 blob 对象成为未被关联的 dangling 对象，可用 `git prune` 命令清除。

`git gc` 命令更为常用，它好比 Git 版本库的管家，会对版本库进行一系列的优化。

实际上，对于 1.6.6 及以后的版本库已经不需要手动执行 `git gc` 命令整理了。
