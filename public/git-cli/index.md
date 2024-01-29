# git command


<!--more-->

### [官网](https://git-scm.com/)

### 创建

```bash
#当前目录下创建git仓库
$ git init 

#? 文件名字
$ git init ? 

#? url 或者 ssh地址
$ git clone ? 
```

### 配置

```bash
#显示当前git 配置
$ git config --list 

#设置提交代码时名字
$ git config --global user.name ""

#设置提交代码邮箱
$ git config --global user.email ""
#设置编辑器
$ git config --global core.editor vim

#包拉取(尤其是go module)的时候用ssh方式代替http方式
[url "ssh://git@github.com/"]
	insteadOf = https://github.com/
```

### 增加和删除

```bash
#添加指定文件到缓存区
$ git add file1 file2 

#添加当前目录下所有的的文件
$ git add . 

#添加指定文件夹包括子文件到缓存区
$ git add dir 

# 添加每个变化前，都会要求确认
# 对于同一个文件的多处变化，可以实现分次提交
$ git add -p

# 删除工作区文件，并且将这次删除放入暂存区
$ git rm file1 file2 ...

# 暂存区删除指定文件，但该文件会保留在工作区
$ git rm --cached file

# 改名文件，并且将这个改名放入暂存区
$ git mv file-original file-renamed
```

### 代码提交

```bash
# 提交暂存区到仓库区,?提交原因
$ git commit -m "?"

# 提交暂存区的指定文件到仓库区
$ git commit file1 file2 ... -m "?"

# 对于那些已经存在的文件修改，可以一步到仓库区，不需要git add.
# 但是对于新增文件 还是要 git add file 再 git commit ""
$ git commit -a -m "?"

# 提交时显示所有diff信息
$ git commit -v

# 使用一次新的commit，替代上一次提交
# 如果代码没有任何新变化，则用来改写上一次commit的提交信息
# 场景，刚刚git commit后,发现又有东西要改
$ git commit --amend -m '?'

# 重做上一次commit，并包括指定文件的新变化
$ git commit --amend file1 file2 ...
```

### 分支

```bash
# 列出所有本地分支
$ git branch

# 列出所有远程分支
$ git branch -r

# 列出所有本地分支和远程分支
$ git branch -a

# ?分支名,新建一个分支，但依然停留在当前分支
$ git branch ?

# ?分支名,新建一个分支，并切换到该分支
$ git checkout -b ?

# ?1分支名,?2commit,新建一个分支，指向指定commit
$ git branch ?1 ?2

# 新建一个分支，与指定的远程分支建立追踪关系
$ git branch --track [branch-name] [remote-branch-name]

# ?分支名，切换到指定分支，并更新工作区
$ git checkout ?

# 切换到上一个分支
$ git checkout -

# 建立追踪关系，在现有分支与指定的远程分支之间
$ git branch --set-upstream [branch-name] [remote-branch-name]

# 合并指定分支到当前分支
$ git merge [branch]

# 选择一个commit，合并进当前分支
$ git cherry-pick [commit]

# 删除分支
$ git branch -d [branch-name]

# 删除远程分支
$ git push origin --delete [branch-name]
```

### 标签

```bash
# 列出所有tag
$ git tag

# 新建一个tag在当前commit
$ git tag [tag]

# 新建一个tag在指定commit
$ git tag [tag] [commit]

# 删除本地tag
$ git tag -d [tag]

# 删除远程tag
$ git push origin :refs/tags/[tagName]

# 查看tag信息
$ git show [tag]

# 提交指定tag
$ git push origin v1

# 提交所有tag
$ git push [remote] --tags

# 新建一个分支，指向某个tag
$ git checkout -b [branch] [tag]
```

### 查看信息

```bash
# 显示有变更的文件
$ git status

# 显示当前分支的版本历史
$ git log

# 显示commit历史，以及每次commit发生变更的文件
$ git log --stat

# 搜索提交历史，根据关键词
$ git log -S [keyword]

# 显示某个commit之后的所有变动，每个commit占据一行
$ git log [tag] HEAD --pretty=format:%s

# 显示某个commit之后的所有变动，其"提交说明"必须符合搜索条件
$ git log [tag] HEAD --grep feature

# 显示某个文件的版本历史，包括文件改名
$ git log --follow [file]
$ git whatchanged [file]

# 显示指定文件相关的每一次diff
$ git log -p [file]

# 显示过去5次提交
$ git log -5 --pretty --oneline

# 显示所有提交过的用户，按提交次数排序
$ git shortlog -sn

# 显示指定文件是什么人在什么时间修改过
$ git blame [file]

# 显示暂存区和工作区的差异
$ git diff

# 显示暂存区和上一个commit的差异
$ git diff --cached [file]

# 显示工作区与当前分支最新commit之间的差异
$ git diff HEAD

# 显示两次提交之间的差异
$ git diff [first-branch]...[second-branch]

# 显示今天你写了多少行代码
$ git diff --shortstat "@{0 day ago}"

# 显示某次提交的元数据和内容变化
$ git show [commit]

# 显示某次提交发生变化的文件
$ git show --name-only [commit]

# 显示某次提交时，某个文件的内容
$ git show [commit]:[filename]

# 显示所有的git 操作 包括切分支，pull push，reset 等。
# 可以找到丢失的commit号
$ git reflog
```

###  远程同步

````bash
# 下载远程仓库的所有变动
$ git fetch [remote]

# 显示所有远程仓库
$ git remote -v

# 显示某个远程仓库的信息
$ git remote show [remote]

# 增加一个新的远程仓库，并命名
$ git remote add [shortname] [url]

# 取回远程仓库的变化，并与本地分支合并
$ git pull [remote] [branch]

# 上传本地指定分支到远程仓库
$ git push [remote] [branch]

# 强行推送当前分支到远程仓库，即使有冲突
$ git push [remote] --force

# 推送所有分支到远程仓库
$ git push [remote] --all
````

### 撤销

```bash
# 恢复暂存区的指定文件到工作区
$ git checkout [file]

# 恢复某个commit的指定文件到暂存区和工作区
$ git checkout [commit] [file]

# 恢复暂存区的所有文件到工作区,⚠️使用前确定好
$ git checkout .

# 重置暂存区的指定文件，与上一次commit保持一致，但工作区不变
$ git reset [file]

# 重置暂存区与工作区，与上一次commit保持一致
$ git reset --hard

# 重置当前分支的指针为指定commit，同时重置暂存区，但工作区不变
$ git reset [commit]

# 重置当前分支的HEAD为指定commit，同时重置暂存区和工作区，与指定commit一致
$ git reset --hard [commit]

# 重置当前HEAD为指定commit，但保持暂存区和工作区不变
$ git reset --keep [commit]

# 新建一个commit，用来撤销指定commit
# 后者的所有变化都将被前者抵消，并且应用到当前分支
$ git revert [commit]

#暂时将未提交的变化移除，稍后再移入
$ git stash
# 将暂存的内容弹出
$ git stash pop
#列出暂存内容
$ git stash list
#清除暂存内容
$ git stash clear
```

### 命令以英文的方式展示

```bash
LANG=en_GB git
echo "alias git='LANG=en_GB git'" >> ~/.bash
```



### 流程图

{{% figure class="center" src="/img/git/1.png" alt="img" %}}

### fork后的仓库怎么与原仓库同步

1. 删掉仓库，重新fork

2. 设置上游代码库，进行merge

   ```bash
   $ cd ~/go/src/github.com/genDopamine
   
   ##查看远端链接
   $ git remote -v
   origin  https://github.com/genDopamine/gin (fetch)
   origin  https://github.com/genDopamine/gin (push)
   
   ##设置upstream
   $ git remote add upstream git@github.com:gin-gonic/gin.git
   
   ##再次查看git remote -v
   $ git remote -v
   origin  https://github.com/genDopamine/gin (fetch)
   origin  https://github.com/genDopamine/gin (push)
   upstream        git@github.com:gin-gonic/gin.git (fetch)
   upstream        git@github.com:gin-gonic/gin.git (push)
   
   ##更新原仓库的类容
   $ git fetch upstream
   
   #合并原仓库类容
   $ git merge upstream/master
   
   #push本仓库
   $ git push
   ```

   ### 社区书
   
   [git community book](http://shafiul.github.io/gitbook/index.html)

