# Git
## random
- git cherry-pick
- git rebase
- git delete remote branch
- git remote/upstream
- git branch -vv
- git push origin :remote-branch-name
- git commit -a -m "git commit without add"
- git rm --cached filename
- git rm filename (remove from staged and working)
- git mv (rename file on stageing level)
- commit object
    - pointer
    - email
    - name
    - message
- blob object
- object tree

## basic
### log
- git log -2
- git log -p
- git log --stat
- git log --pretty=oneline
- git log --pretty=format:""   [Git-基础-查看提交历史](https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E6%9F%A5%E7%9C%8B%E6%8F%90%E4%BA%A4%E5%8E%86%E5%8F%B2)
- git log --graph
- git log --since=2.weeks
- git log -Sstring
- git log --decorate...查看各个分支引用提交对象
### commit
- git commit --amend
    - 修改提交信息
    - 添加内容并修改提交信息
- git reset HEAD file
- git checkout -- file

### remote repo
- git fetch repo_url
- git remote show
- git remote rename
- git remote remove/rm

### tag
- git tag list tags
- git tag -l 'v1.2*'
- git tag -a v1.0 -m "message"...add annotated tag
- git show tag
- git tag -a v1.2 commit-sha-1
- git push origin master tagname
- git push origin master --tags...push all tags
- git checkout -b branch-name tagname

### alias
```
$ git config --global alias.co checkout
$ git config --global alias.br branch
$ git config --global alias.ci commit
$ git config --global alias.st status
$ git config --global alias.unstage 'reset HEAD --'
$ git config --global alias.last 'log -1 HEAD'...last commit
$ git config --global alias.visual '!gitk'
```

## branch
### random
```
git branch name -t upstream
OR
git branch name
git branch --set-upstream-to=upstream name
```

- 创建的分支会跟踪远程分支。
- 分支切换会改变工作目录中的文集
- 分支只是对象的校验和
- **三方合并与快进**
- 凭证存储
- git commit -a -m "msg"
- git branch --merged/--no-merged...查看哪些分支合并/未合并到当前分支
- master/develop/next/proposed/iss/hotfix
- git push origin local-branch:remote-branch-name
- git checkout -b serverfix origin/serverfix...以`origin/serverfix`为起点新建分支
- git checkout --track origin/serverfix | git checkout -b branch-name --track origin/serverfix
- git branch -u|--set-upstream-to origin/serverfix
- 上游快捷方式**@{upstream}/@{u}**
- git merge @{u}|git merge origin/master
- [ahead2,behind1]：本地2个commit没有推送到远程分支，远程分支有一个commit没有更新到本地（比较的是本地缓存的远程信息，所以最好`git fetch --all`）
- 删除远程分支：git push --delete remote-branch-name|git push :remote-branch-name

### rebase
整合来自不同分支的内容常用变基和合并（三方合并），将提交到某分支上的修改，都移动到另一个分支。
![](https://git-scm.com/book/en/v2/book/03-git-branching/images/basic-rebase-2.png)

## tools
- git rev-parse
- git reflog
- git show HEAD@{5}
- git show master@{yesterday}
- `HEAD^`，上一个提交
- `HEAD6`，第6父提交

## git internals
- 内容寻址文件系统
- 核心部分是一个简单的键值对数据库（key-value data store）
- hash-object
- cat-file
- update-index...更新文件到暂存区
- write-tree
- commit-tggree
- 将改写的信息保存为`数据对象`--更新暂存区--记录`树对象`--创`建提交对象`
![](https://git-scm.com/book/en/v2/book/10-git-internals/images/data-model-3.png)

### reference
- update-ref
- **这基本就是 Git 分支的本质：一个指向某一系列提交之首的指针或引用**
- git branch (branch),git会运行`update-ref`底层命令，获得当前分支最新提交的sha-1的值，并将它加入到新的引用中。
- `.git/HEAD`:符号链接，指向当前所在的分支。
- git symbolic-ref HEAD
- git symbolic-ref HEAD refs/heads/test

- tag object,标签对象，指向一个提交对象，永远指向同一个提交。给这个提交指定一个别名。
- 远程引用只读。

- git gc

### 引用规格
```
$ git log origin/master
$ git log remotes/origin/master
$ git log refs/remotes/origin/master
```
- 每次都拉取master分支
```
fetch = +refs/heads/master:refs/remotes/origin/master
OR
$ git fetch origin master:refs/remotes/origin/mymaster
```

- git fsck
- count-objects -v

### 环境变量
- GIT_EXEC_PATH；git --exec-path
