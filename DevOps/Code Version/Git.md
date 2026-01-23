## Git 详解

> 参考：[官方网站](https://git-scm.com/)，[Git详解](https://www.jianshu.com/p/382abb427ca9)，[微软官方文档](https://docs.microsoft.com/zh-cn/learn/paths/github-administration-products/)

Git 是一个分布式版本控制系统，CVS 及 SVN 都是集中式的版本控制系统。

## TODO

有必要整理一下Git的工作流程

## 命令

* `git init`：把当前文件夹初始化为一个仓库	
* `rm -rf .git`：删除仓库，本质就是删除 .git 文件夹，ls -a 能看到全部文件夹
* `git status`：查看 git 状态	



git不同的项目有不用的用户，应该有方便的设置来进行吧，不用每次都切换一下吧？

* `git config --global user.name "Jinzhao Tian" `：设置 git 自己的名字
* `git config --global user.email "jinzhao.tian.cs@gmail.com"`：和电子邮件
* `git config -l`：查看用户信息
* `git config --local  --list`：查看当前仓库配置信息



* `git branch`：查看所有分支	
* `git branch [name]`：创建分支
* `git branch -d [name]`：删除分支
* `git checkout [name]`：切换分支
* `git checkout -b [name]`：创建`[name]`分支，并切换到`[name]`分支
* `git merge [name]`：把`[name]`分支合并到`main`分支上
* `git pull --rebase`：就是你在你的分支上开发的时候，主分支又merge了一部分，那你当前的分支对应主分支上的起点就不是最新的origin master HEAD的，所以pull request的时候就要rebase一下，将你的分支的起点重新定位到最新的origin master HEAD。
* `git stash`：将修改的内容保存至堆栈区，等待另一个修改完成，再用`git stash pop`将你做的修改恢复一下，比如说直接pull origin master会冲突，就先放在堆栈区，合并了再弹出。
* `git branch -M main`：快捷修改当前项目的分支为main
* `git config --global init.defaultBranch main`：修改默认分支为main分支



* `git log`：列出当前分支的历史提交信息。
* `git reflog`：列出当前分支的所有commit，reset的相关命令列表，这是repo的undo历史。可以用``git reset --hard [历史号]``来回退到先前的状态。



* `git add [filename]`：添加文件到缓存区，filename为 `.` 的时候表示所有文件。
* `git reset HEAD [filename]`：撤销已经提交的缓存区的修改后文件，HEAD表示最新的版本。一般常用的有`git reset --soft [sha256]`，用来撤回提交的commit，把commit放回staged区，已经做的更改不会丢失。
* `git commit -m “balabala”`：提交	
* `git reset --hard [commit_id]`：版本回退，已经做的更改会丢失。
* `git rm [filename]`：已用`rm [filename]`命令删除文件后，如果确实要从版本库删除该文件，则使用该命令
* `git checkout -- [filename]`：如果删错了，用这个命令还原。`git checkout`：其实是用版本库里的版本替换工作区的版本，无论工作区是修改还是删除，都可以“一键还原”


- `git checkout -b [本地分支名] [origin/远程分支名]` ：拉取远程分支到本地

- `git cherry-pick [hash]` : 

* `git remote -v`：查看远程仓库的状态

* `git remote add origin https://github.com/***/***.git`：为本仓库添加远程仓库，远程仓库要自己建立好	

* `git remote rm origin`：删除远程仓库	



* `git push -u origin main`：推送到远程仓库，第一次要加-u，以后不用	

* `git pull origin main`：把远程仓库的内容拉进来	

* `git clone https://github.com/***/***.git `: 克隆别人的代码仓库


  **注意**：

  GitHub进行了更新，`git clone https://<username>:<githubtoken>@github.com/<username>/<repositoryname>.git`，使用了personal Token来代替密码，每个账户生成一个Token，然后具体的权限在生成Token时进行选择。[参考](https://stackoverflow.com/questions/68775869/support-for-password-authentication-was-removed-please-use-a-personal-access-to)



个人令牌拉取：`git clone https://x-access-token:your-token@github.com/your-username/your-repo.git
`



## 技巧

### 合并多个提交

```bash
git rebase -i HEAD~3      # 从HEAD版本开始往过去数3个版本

git rebase -i [commitid]  # 从此提交开始到当前的提交
```

1. 使用`git rebase -i`选择要合并的 commit
2. 编辑要合并的版本信息，保存提交，多条合并会出现多次（可能会出现冲突）
	- p, pick = use commit
	- r, reword = use commit, but edit the commit message
	- e, edit = use commit, but stop for amending
	- s, squash = use commit, but meld into previous commit
	- f, fixup = like "squash", but discard this commit's log message
	- x, exec = run command (the rest of the line) using shell
	- d, drop = remove commit
3. 修改注释信息后，保存提交，多条合并会出现多次
4. 推送远程仓库或合并到主干分支



### 调整提交顺序

```bash
git rebase -i HEAD~3      # 从HEAD版本开始往过去数3个版本

git rebase -i [commitid]  # 从此提交开始到当前的提交
```

使用快捷键 dd 剪切要调整的 commit 行，光标移动到指定行，按 p 把内容粘贴到当前行的下方。



## Worktree

传统上，一个 Git 仓库只有一个工作目录（`.git` 文件夹所在的目录），Git Worktree 允许你在同一个仓库中同时维护多个工作目录，连接到同一个 Git 仓库。

Worktree 让你可以：
1. **同时创建多个工作目录**
2. **每个目录可以 checkout 不同的分支**
3. **共享同一个 .git 仓库数据**


### 使用

```bash
# 创建新的工作树
git worktree add <路径> <分支名>

# 示例
git worktree add ../feature-login feature/login
git worktree add ../hotfix-bugfix hotfix/123

# 列出所有工作树
git worktree list

# 删除工作树
git worktree remove <路径>
```








## Commit Message

```
<type>(<scope>): <subject>
```

字段解释：

1. **type**：用于说明git commit的类别，只允许使用下面的标识

   * feat：新功能（feature）。
   * fix/to：修复bug，可以是QA发现的BUG，也可以是研发自己发现的BUG。
   * docs：文档（documentation）。
   * style：格式（不影响代码运行的变动）。
   * refactor：重构（即不是新增功能，也不是修改bug的代码变动）。
   * perf：优化相关，比如提升性能、体验。
   * test：增加测试。
   * chore：构建过程或辅助工具的变动。
   * revert：回滚到上一个版本。
   * merge：代码合并。
   * sync：同步主线或分支的Bug。

2. **scope**：用于说明 commit 影响的范围，视项目不同而不同。

3. **subject**：是commit目的的简短描述，不超过50个字符。





## 常见问题

1. [解决 Github port 443 : Timed out - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/636418854)
```
git config --global http.proxy http://127.0.0.1:7890 
git config --global https.proxy http://127.0.0.1:7890
```
