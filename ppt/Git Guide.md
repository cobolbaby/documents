title: Git Guide
author:
  name: Zhang.Xing-Long
  url: https://github.com/cobolbaby
  email: Zhang.Xing-long@inventec.com
output: _ppt/git-guide.html
theme: jdan/cleaver-retro

--

# Git入门指南
## 2018/08/30

--

### 目标

* 了解基本概念
* 玩转常用指令
* 明确工作流

--

# 基本概念
## 通过工作方式的变化谈Git的基本概念

--

### SVN vs Git - 离线操作

对于SVN，版本信息集中存放在中央服务器的，而干活的时候，用的都是自己的电脑，所以要先从中央服务器取得最新的版本，然后开始干活，干完活了，再把自己的活推送给中央服务器。而Git不同之处在于除了在中央服务器存储文件版本信息，每个人的电脑上也会有一个完整的版本库。这也意味着你不必依赖于网络就可以执行几乎所有的操作。举个例子，你回家后仍可以自由的拉分支修改代码。只有当你去拉取或更新远端仓库的时候，才会需要网络。

--

### SVN vs Git - 快速迭代

当前的软件产品都要求“快速迭代，持续交付”，开发人员会面临着太多的feature开发，此时就需要用分支去区分feature，以保证快速灵活的上线需求。所以随意的拉取分支以及切换分支是必要的一个条件，而Git就很好的做到了一点。*离线环境拉分支*解放了生产力，*切换分支无需切目录*也提高了生产效率。

--

### SVN vs Git - 文件状态的变更

讲Git文件状态之前先谈一个概念－－三棵“树”，即Git本地仓库的三个组成部分。<br>
1、工作区（working directory）<br>
工作目录是对项目的某个版本独立提取出来的内容。 这些从 Git 仓库的压缩数据库中提取出来的文件，放在磁盘上供你使用或修改。<br>
2、暂缓区（stage index）<br>
暂存区域保存了下次将提交的文件信息。有时候也被称作“索引”，不过一般说法还是叫暂存区域。<br>
3、Git仓库<br>
Git 仓库目录是 Git 用来保存项目的元数据和对象数据库的地方。

--

### SVN vs Git - 三棵树

<img src="https://note.youdao.com/yws/api/personal/file/WEB3f566bec96a38bc042d7dd767ff9b884?method=download&shareKey=1166885fdd960e911796cde9f6fad93f" style="width: 100%;">

--

### SVN vs Git - 文件状态的变更

上面讲了三棵“树”，相对应的文件也有三种状态：已修改(modified)、已暂存(staged)和 已提交(committed)。<br>
已修改表示修改了文件，但还没保存到数据库中。已暂存表示对一个已修改文件的当前版本做了标记，使之包含在下次提交的快照中。已提交表示数据已经安全的保存在本地数据库中。<br>
所以我们可以看到Git中修改了文件，我们需要先将其添加(add)到暂存区然后才能提交(commit)至本地版本库，而SVN直接通过commit即可。

--

### SVN vs Git - 提交变更

SVN里面的使用commit可以直接将修改提交到远端仓库，但是Git的commit仅是提交到本地仓库，如果需要推送至远端，还需要在commit之后执行push操作。

--

### SVN vs Git - 生态

Git庞大的生态圈就无需赘述了。

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

查看当前分支: git branch<br>
查看远程分支: git branch -r<br>
查看所有分支: git branch -a<br>
切换当前分支: git checkout <branch\><br>

--

### 新建分支

当你需要创建一个叫做 feature_x 新分支的时候，首先应该明确它基于哪个基础分支，是基于 master ，还是某一个 feature 分支，再或者是一个集成分支，必须先把这一点明确了。至于命令超简单：

git checkout -b <branch\>

--

### 修改及提交

当你需要将修改的内容提交的时候，与传统SVN的提交不同，此时需要经过两步操作，首先将已修改的内容添加至暂存区，当确认无误之后再将暂存区的内容进行提交，并且在提交过程中必须指定备注信息：

git status<br>
git add <filename\> <br>
git add .<br>
git status<br>
git commit -m "备注信息"<br>
git commit -am "备注信息"<br>

--

### 推送改动

上面提到的操作仅是将修改的内容保存至本地的 Git 库，并没有同步至远端，此时可以通过执行如下命令将其同步至远端仓库：

git push origin master

可以把 master 换成你想要推送的任何分支，但要明确的是Push代码的时候指定的分支参数应该与当前所在分支名一致

--

### 拉取更新

当我们 push 代码的时候，理想情况下一步到位没啥异常，但如果是多人协同开发的时候，代码冲突在所难免，即便没有冲突，那也需要在提交前先 pull 最新的代码,所以在提交之前，先习惯性的 git pull 最新的代码.

如果没有冲突，谢天谢地呀，那可以直接 push 了.<br>
如果存在冲突，就需要先解决冲突，解决冲突也很简单，只需要按照 git pull 时提示的异常信息操作即可，也可以依赖 IDE 解决冲突，这一点和 SVN 没啥本质区别。

--

### 查看Log

简单暴力 git log，不过更推荐在 Gitlab 中直接查看

--

### 合并分支

当开发的功能完成之后，需要将开发过程中创建的功能分支合并到主分支上。

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

### 参考指南

* [官方手册](https://git-scm.com/book/zh/v2)
* [GUI工具](https://git-scm.com/download/gui/win)

--

# 工作流
## 实际工作中如何更好地使用Git

--

### 分支管理

<img src="https://note.youdao.com/yws/api/personal/file/WEB7a96ac8f29901b359fb4e462f2a2f845?method=download&amp;shareKey=055bcb1fc16ce24e693e6d79687543a5" alt="分支管理" style="width: 100%;">

--

### 仓库管理

* [GitLab](http://itc-gitlab.itc.inventec/)
* [GitHub](https://github.com)

--

# 致谢

