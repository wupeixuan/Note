你使用过 Git 吗？也许你已经使用了一段时间，但它的许多奥秘仍然令人困惑。

Git 是一个版本控制系统，是任何软件开发项目中的主要内容。通常有两个主要用途：代码备份和代码版本控制。你可以逐步处理代码，在需要回滚到备份副本的过程中保存每一步的进度！

常见的问题是 Git 很难使用。有时版本和分支不同步，你会花很长时间试图推送代码！更糟糕的是，不知道某些命令的确切工作方式很容易导致意外删除或覆盖部分代码！

这就是我写本文的原因，从而学习到如何正确使用 Git，以便在开发中共同进行编码！

# 安装和配置
## Git 安装
首先，我们必须安装 Git 才能使用它！这里分 Linux 和 Windows 来演示：

**在 Linux 上安装 Git**

我们可以使用 yum 轻松快速地做到这一点：

```
sudo yum install git
```

**在 Windows 上安装 Git**

直接在 `https://git-scm.com/downloads` 里面，下载最新版的 Git，默认安装就可以了。

安装完成后，在开始菜单里找到 `Git->Git Bash`，点击后出现一个类似命令行窗口的东西，就说明 Git 安装成功。

## Git 配置
可以保存 Git 用户名和电子邮件，这样就不必在以后的 Git 命令中再次输入它们。

在命令行中配置本地仓库的账号和邮箱：
```
$ git config --global user.name "wupx"  
$ git config --global user.email "wupx@qq.com"  
```

好多人都不知道的小技巧是，你可以为 Git 启用一些额外的颜色，这样就可以更容易地阅读命令的输出！

```
git config --global color.ui true
```

# Git 基本版本控制

![](https://img-blog.csdnimg.cn/20191105205605446.png)

## 初始化 Git

现在，我们可以开始对项目进行版本控制。使用 cd 命令导航到要在终端中设置版本控制的目录，现在你可以像这样初始化 Git 存储库：

```
git init
```

这将创建一个名为 .git 的新子目录（Windows 下该目录为隐藏的），其中包含所有必需的存储库文件（Git 存储库框架）。至此，你的项目中尚未跟踪任何内容。

## 添加并提交

要开始对现有文件进行版本控制，你应该先跟踪这些文件并进行初始提交。要做到这一点，你首先需要将文件添加到 Git 中，并将它们附加到 Git 项目中。

```
git add <file>
git commit -m 'first commit'
```

## 远程备份
很棒！你现在已经开始在本地对项目进行版本控制。如果你想远程保存和备份项目，则需要在 GitHub 上创建一个远程存储库（它是免费的！）。因此，首先转到 github.com 并创建一个存储库。然后，使用存储库的链接将其添加为本地 git 项目的来源，即该代码的存储位置。

```
# 示例
git remote add origin \
https://github.com/wupeixuan/repo.git 
# 以我的一个仓库为例
git remote add origin \
https://github.com/wupeixuan/JDKSourceCode1.8.git
```

然后，你可以继续将代码推送到 GitHub！哇，你已经成功备份了你的代码！

```
git push origin master
```

![](https://img-blog.csdnimg.cn/2019110521023889.png)

# 处理文件

## 状态检查
git status 命令用于确定哪些文件处于哪种状态，它使你可以查看哪些文件已提交，哪些文件尚未提交。如果在所有文件都已提交并推送后运行此命令，则应该看到类似以下内容：

```
$ git status
# On branch master
nothing to commit (working directory clean)
```

如果你将新文件添加到项目中，而该文件之前不存在，则在运行 `git status` 时，你应该看到未跟踪的文件，如下所示：

```
$ git status
# On branch master
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#
#   README
nothing added to commit but untracked files present (use "git add" to track)
```

使用 `git status` 对于快速检查你已经备份的内容和你仅在本地拥有的内容非常有用。

## 高级文件添加
还有一些更高级的方法可以将文件添加到 Git 中，从而使你的工作流程更高效。我们可以执行以下操作，而不是试图查找所有有更改的文件并逐个添加它们：

```
# 逐个添加文件
git add filename

# 添加当前目录中的所有文件
git add -A

# 添加当前目录中的所有文件更改
git add .

# 选择要添加的更改（你可以 Y 或 N 完成所有更改）
git add -p
```

## 高级提交
我们可以使用 `git commit -m '提交信息'` 来将文件提交到 Git。对于提交简短消息来说，这一切都很好，但是如果你想做一些更精细的事情，你需要来学习更多的操作:

```
# 提交暂存文件，通常用于较短的提交消息
git commit -m 'commit message'

# 添加文件并提交一次
git commit filename -m 'commit message'

# 添加文件并提交暂存文件
git commit -am 'insert commit message'

# 更改你的最新提交消息
git commit --amend 'new commit message' 

# 将一系列提交合并为一个提交，你可能会用它来组织混乱的提交历史记录
git rebase -i
# 这将为你提供核心编辑器上的界面：
# Commands:
#  p, pick = use commit
#  r, reword = use commit, but edit the commit message
#  e, edit = use commit, but stop for amending
#  s, squash = use commit, but meld into previous commit
#  f, fixup = like "squash", but discard this commit's log message
#  x, exec = run command (the rest of the line) using shell
```

# 分支与合并

GitHub存储库的master分支应始终包含有效且稳定的代码。但是，你可能还希望备份一些当前正在处理的代码，但这些代码并不完全稳定。也许你要添加一个新功能，你正在尝试和破坏很多代码，但是你仍然希望保留备份以保存进度！


分支使你可以在不影响master分支的情况下处理代码的单独副本。首次创建分支时，将以新名称创建master分支的完整克隆。然后，你可以独立地在此新分支中修改代码，包括提交文件等。一旦你的新功能已完全集成并且代码稳定，就可以将其合并到master分支中！

## 分支

这是你在分支上创建和工作所需的所有东西：

```
# 创建一个本地分支
git checkout -b branchname

# 在2个分支之间切换
git checkout prc/dev-wupx
git checkout master

# 将新的本地分支作为备份
git push -u origin branch_2

# 删除本地分支，这不会让你删除尚未合并的分支
git branch -d branch_2

# 删除本地分支，即使尚未合并，这也会删除该分支！
git branch -D branch_2

# Viewing all current branches for the repository, including both # local and remote branches. Great to see if you already have a # branch for a particular feature addition, especially on bigger # projects
# 查看存储库的所有当前分支，包括本地和远程分支。
git branch -a

# 查看已合并到您当前分支中的所有分支，包括本地和远程。 非常适合查看所有代码的来源！
git branch -a --merged

# 查看尚未合并到当前分支中的所有分支，包括本地和远程
git branch -a --no-merged

# 查看所有本地分支
git branch

# 查看所有远程分支
git branch -r

# 将主分支重新设置为本地分支
$ git rebase origin/master

# 将分支推送到远程存储库源并对其进行跟踪
$ git push origin branchname
```

## 合并
很棒！现在，你已经学习了如何创建分支并开始敲代码！将新功能添加到分支中之后，你需要将其合并回master分支，以便您的master具有所有最新的代码功能。

方法如下：

```
# 首先确保你正在查看 master 分支
git checkout master

# 现在将你的分支合并到 master 
git merge prc/dev-wupx
```

你可能必须修复分支与主服务器之间的任何代码冲突，但是 Git 将向你展示在键入该 merge 命令后如何执行所有这些操作。

# 修复错误和回溯

发生错误......它们经常在编码中发生！重要的是我们能够修复它们。

不要慌！Git 提供了你所需的一切，以防你在所推送的代码中犯错，改写某些内容或者只是想对所推送的内容进行更正。

```
# 切换到最新提交的代码版本
git reset HEAD 
git reset HEAD -- filename # for a specific file
# 切换到最新提交之前的代码版本
git reset HEAD^ -- filename
git reset HEAD^ -- filename # for a specific file
# 切换回3或5次提交
git reset HEAD~3 -- filename
git reset HEAD~3 -- filename # for a specific file
git reset HEAD~5 -- filename
git reset HEAD~5 -- filename # for a specific file
# 切换回特定的提交，其中 0766c053 为提交 ID
git reset 0766c053 -- filename
git reset 0766c053 -- filename # for a specific file
# 先前的命令是所谓的软重置。 你的代码已重置，但是git仍会保留其他代码的副本，以备你需要时使用。 另一方面，--hard 标志告诉Git覆盖工作目录中的所有更改。
git reset --hard 0766c053
```

# 对 Git 有用的提示和技巧
我们已经完成了所有细节部分！以下是一些 Git 提示和技巧，你可能会发现它们对改善工作流程非常有用！

## 搜索

```
# 搜索目录中的字符串部分
git grep 'project'

# 在目录中搜索部分字符串，-n 打印出 git 找到匹配项的行号
git grep -n 'project'

# git grep -C <行数> 'something' 搜索带有某些上下文的字符串部分（某些行在我们正在寻找的字符串之前和之后）
git grep -C<number of lines> 'project'

# 搜索字符串的一部分，并在字符串之前显示行
git grep -B<number of lines> 'project'

# 搜索字符串的一部分，并在字符串之后显示行
git grep -A<number of lines> 'something'
```

## 看谁写了什么

```
# 显示带有作者姓名的文件的更改历史记录
git blame 'filename'

# 显示带有作者姓名和 git commit ID 的文件的更改历史记录
git blame 'filename' -l
```

## 日志

```
# 显示存储库中所有提交的列表 该命令显示有关提交的所有信息，例如提交ID，作者，日期和提交消息
git log

# 提交列表仅显示提交消息和更改
git log -p

# 包含您要查找的特定字符串的提交列表
git log -S 'project'

# 作者提交的清单
git log --author 'wupx'

# 显示存储库中提交列表的摘要。显示提交ID和提交消息的较短版本。
git log --oneline

# 显示昨天以来仓库中的提交列表
git log --since=yesterday

# 显示作者日志，并在提交消息中搜索特定术语
git log --grep "project" --author "wupx"
```

![image](https://img-blog.csdnimg.cn/20191127194052721.png)