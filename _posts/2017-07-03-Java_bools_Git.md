---
layout: post
title: Git常用命令
date: 2017-07-03 
tag: 工具
---

### Git add ###

	git add .                         # 将资料先暂存到staging area, add 之后再新增的资料,于此次commit 不会含在里面.
	git add filename                  # 增加文件到git库中
	git add modify-file               # 修改过的档案, 也要add. (不然commit 要加上-a 的参数)
	git add -u                        # 只加修改过的档案, 新增的档案不加入.
	git add -i 

### Git rm ###

	git rm filename                   # 将文件从git库中删除,这只是删除本地的文件,需要commit,还要push到远程git库中.

### Git mv ###

	git mv filename new-filename      # 将文件改名

### Git status ###

	git status                        # 看目前档案的状态

### Git Commit ###

	git commit
	git commit -m 'commit message'    # 修改过的文件进行提交,但必须先要执行 git-add.
	git commit -a -m 'commit -message'# 将所有修改过得档案都commit, 不需要执行 git-add命令.-a参数就是已经自动执行git-add命令
	git commit -a -v                  # -v 可以看到档案哪些内容有被更改, -a 把所有修改的档案都commit


### Git new-branch ###

	git branch                        # 列出目前有多少branch
	git branch new-branch             # 产生新的branch (名称: new-branch), 若没有特别指定, 会由目前所在的branch / master 直接复制一份.
	git branch new-branch master      # 由master 产生新的branch(new-branch)
	git branch new-branch tag v1      # 由tag(v1) 产生新的branch(new-branch)
	git branch -d new-branch          # 删除new-branch
	git branch -D new-branch          # 强制删除new-branch
	git checkout -b new-branch test   # 产生新的branch, 并同时切换过去new-branch与remote repository 有关
	git branch -r                     # 列出所有Repository branch
	git branch -a                     # 列出所有branch
	git branch -av                    # 列出所有分支,还有提交的哈希值.
	git branch -rD origin/name        # 删除远程分支或仓库分支.
	git push origin :heads/name       # 提交

### Git checkout-branch ###

	git checkout branch-name          # 切换到branch-name
	git checkout master               # 切换到master
	git checkout -b new-branch master # 从master 建立新的new-branch, 并同时切换过去new-branch
	git checkout -b new-branch        # 由现在的环境为基础, 建立新的branch
	git checkout -b new-branch tag v1 # 从tag v1 checkout文档,并新建一个branch.
	git checkout -b new-branch origin # 基于origin 的基础, 建立新的branch
	git checkout filename             # 还原档案到Repository 状态
	git checkout HEAD .               # 将所有档案都checkout 出来(最后一次commit 的版本),
	                                    #注意, 若有修改的档案都会被还原到上一版. (git checkout -f 亦可)
	git checkout xxxx .               # 将所有档案都checkout 出来(xxxx commit 的版本, xxxx是commit 的编号前四码),
	                                    #注意, 若有修改的档案都会被还原到上一版.
	git checkout -- *                 # 恢复到上一次Commit 的状态(* 改成档名, 就可以只恢复那个档案)


### Git diff ###

	git diff master                   # 与Master 有哪些资料不同
	git diff --cached                 # 比较staging area 跟本来的Repository
	git diff tag1 tag2                # tag1, 与tag2 的diff
	git diff tag1:file1 tag2:file2    # tag1, 与tag2 的file1, file2 的diff
	git diff                          # 比较目前位置与staging area
	git diff --cached                 # 比较staging area 与Repository 差异
	git diff HEAD                     # 比较目前位置与Repository 差别
	git diff new-branch               # 比较目前位置与branch(new-branch) 的差别
	git diff --stat

### Git tag ###

	git tag v1 ebff                   # log 是commit ebff810c461a 的内容, 设定简短好记得Tag: v1
	git tag -d v1                     # 把tag v1删掉.

### Git log ###

	git log                           # 将所有log 秀出
	git log --all                     # 秀出所有的log (含branch)
	git log -p                        # 将所有log 和修改过得档案内容列出
	git log -p filename               # 将此档案的commit log 和修改档案内容差异部份列出
	git log --name-only               # 列出此次log 有哪些档案被修改
	git log --stat --summary          # 查每个版本间的更动档案和行数
	git log filename                  # 这个档案的所有log
	git log directory                 # 这个目录的所有log
	git log -S'foo()'                 # log 里面有foo() 这字串的.
	git log --no-merges               # 不要秀出merge 的log
	git log --since="2 weeks ago"     # 最后这2周的log
	git log --pretty=oneline          # 秀log 的方式
	git log --pretty=short            # 秀log 的方式
	
	git log --pretty=format:'%h was %an, %ar, message: %s'
	git log --pretty=format:'%h : %s' --graph              # 会有简单的文字图形化, 分支等.
	git log --pretty=format:'%h : %s' --topo-order --graph # 依照主分支排序
	git log --pretty=format:'%h : %s' --date-order --graph # 依照时间排序

### Git show ###

	git show ebff                     # 查log 是commit ebff81 的内容
	git show v1                       # 查tag:v1 的修改内容
	git show v1:test.txt              # 查tag:v1 的test.txt 档案修改内容
	git show HEAD                     # 此版本修改的资料
	git show HEAD^                    # 前一版修改的资料
	git show HEAD^^                   # 前前一版修改的资料
	git show HEAD~4                   # 前前前前一版修改的资料

### Git reset ###

	git reset --hard HEAD             # 还原到最前面
	git reset --hard HEAD~3
	git reset --soft HEAD~3
	git reset HEAD filename           # 从staging area 状态回到unstaging 或untracked (档案内容并不会改变)

### Git grep ###

	git grep "te" v1                  # 查v1 是否有"te" 的字串
	git grep "te"                     # 查现在版本是否有"te" 的字串

### Git stash ###

	git stash                         # 丢进暂存区
	git stash list                    # 列出所有暂存区的资料
	git stash pop                     # 取出最新的一笔, 并移除.
	git stash apply                   # 取出最新的一笔stash 暂存资料. 但是stash 资料不移除
	git stash clear                   # 把stash 都清掉

