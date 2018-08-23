title: Git Guide
author:
  name: Zhang.Xing-Long
  url: https://github.com/cobolbaby
output: git-guide.html
controls: true

--

# Git入门指南
## 2018/08/23

--

### 目标

* 了解核心的概念
* 掌握常用指令
* 明确工作流

--

# 核心概念

--

### A textual example

你的本地仓库由 git 维护的三棵“树”组成。第一个是你的 工作目录，它持有实际文件；第二个是 缓存区（Index），它像个缓存区域，临时保存你的改动；最后是 HEAD，指向你最近一次提交后的结果。

--

附图

--

# 常用指令

--

### 创建仓库

本地创建工作目录，在该目录的根目录下执行 git init 以初始化新的 git 仓库。

不过在实际工作中，更推荐的做法是在 Gitlab 管理页面创建一个项目，一个项目即代表一个 git 库，之后通过克隆的方式将远端的仓库拉取到本地。

--

### 克隆代码

执行如下命令可以将远端服务器上的仓库拷贝到本地：

git clone git@github.com:jdan/cleaver.git (推荐)

git clone https://github.com/jdan/cleaver.git

--

### 查看分支

在开发过程中，分支是绕不开的话题，不同的分支代表着推进中的各功能项以及bug修复，我们可以通过以下命令查看我们的功能分支

查看当前分支: git branch<br>
查看远程分支: git branch -r<br>
查看所有分支: git branch -a<br>
切换当前分支: git checkout <branch\><br>

--

### 新建分支

当你需要创建一个叫做 feature_x 新分支的时候，首先应该明确它基于哪个基础分支，是基于 master ，还是某一个 feature 分支，再或者是一个集成分支，必须先把这一点明确了。至于命令超简单：

git checkout -b <branch\>

--

### 修改及提交

当你需要将修改的内容提交的时候，与传统SVN的提交不同，此时需要经过两步操作，首先将已修改的内容添加至暂存区，当确认无误之后再将暂存区的内容进行提交，并且在提交过程中必须指定备注信息：

git status<br>
git add <filename\> <br>
git add .<br>
git status<br>
git commit -m "备注信息"<br>
git commit -am "备注信息"<br>

--

### 推送改动

上面提到的操作仅是将修改的内容保存至本地的 Git 库，并没有同步至远端，此时可以通过执行如下命令将其同步至远端仓库：

git push origin master

可以把 master 换成你想要推送的任何分支，但要明确的是Push代码的时候指定的分支参数应该与当前所在分支名一致

--

### 拉取更新

当我们 push 代码的时候，理想情况下一步到位没啥异常，但如果是多人协同开发的时候，代码冲突在所难免，即便没有冲突，那也需要在提交前先 pull 最新的代码,所以在提交之前，先习惯性的 git pull 最新的代码.

如果没有冲突，谢天谢地呀，那可以直接 push 了.<br>
如果存在冲突，就需要先解决冲突，解决冲突也很简单，只需要按照 git pull 时提示的异常信息操作即可，也可以依赖 IDE 解决冲突，这一点和 SVN 没啥本质区别。

--

### 查看Log

git log

--

### 合并分支

当开发的功能完成之后，需要将开发过程中创建的功能分支合并到主分支上。

git merge

需要注意的是，A 分支要合并至 B 分支的时候，需要先将当前分支切换至 B 分支，然后执行 git merge A，千万千万别想反了

--

### 标签

在软件发布时创建标签，是被推荐的。这是个旧有概念，在 SVN 中也有。可以执行如下命令以创建一个叫做 1.0.0 的标签：

git tag 1.0.0 1b2e1d63ff

1b2e1d63ff 是你想要标记的提交 ID 的前 10 位字符。使用如下命令获取提交 ID：

git log

你也可以用该提交 ID 的少一些的前几位，只要它是唯一的。

--

# 优势

--

### A list of things

* Item 1
* Item B
* Item gamma

--

# 工作流
## 如何更好地管理代码

--

![link]()

--

# 致谢

