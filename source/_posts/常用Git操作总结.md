---
title: 常用Git操作总结
date: 2019-03-12 22:35:05
tags:
- Git
categories:
- 工具
---
### 合并远程分支
1. git fetch origin branch
2. git checkout [your-branch]
3. git merge FETCH_HEAD，merge 前可以先git dff [your-branch] [remote-branch]，查看本地分支和远程分支的区别

### 使用 rebase 合并分支
1. git rebase [origin-branch]
2. 遇到冲突，解决完冲突后 git add
3. git rebase --continue

rebase 和 merge 的区别
![](http://mares.oss-cn-qingdao.aliyuncs.com/image2018-4-11%2019_43_26.png)


### 合并历史commit
git rebase -i [commit id] commit id是合并的提交的前一个提交节点的commitID，最上的是最早的提交
修改需要合并的commit，将pick修改成 squash
退出保存 wq
git push origin [branch-name] -f
rebase 的主要用途一是替代merge，保持分支历史的线性提交；二是方便合并历史提交。



### 撤销一次merge
1. git reflog 确定回滚的commit id
2. git reset --hard [commit id]

git revert 也可以回退到指定版本，和reset的区别在于，revert会产生一次新的提交,用一次新的提交来消除历史修改。而reset是直接删除历史commit。

### 合并时遇到冲突，想取消merge
git merge --abort

### 修改最后一次提交的commit message
git commit --amend

### 清空工作区的修改
git checkout .

### 恢复误删分支
1. git log -g 找到之前分支提交的 commit id
2. git branch recover_branch [commit id]，这时切换到 recover_branch，可以看到原来的文件了。

### 修改历史提交信息
1. git rebase -i HEAD~10（查看前10次提交信息，也可以直接输入想要修改的commit id）
2. 将想要的修改的commit前的 pick 修改为 reword，保存并退出。
3. 在弹出的窗口中，修改commit message，保存。

### 删除历史提交
1. 用 git rebase -i 928582641a 指定 base 为你需要删除的提交的前一个提交。
2. 删除指定的commit, 保存退出. 之后可能 git 会提示出现 conflict, 根据提示完成处理。

### 创建空白分支
1. git checkout --orphan new_branch
2. git rm -rf .

### 查看本地分支和远程分支的差异
1. git fetch origin
2. git diff master origin/master --minimal

### 建立远程追踪分支
git branch -u [remote_branch]

### 删除远程分支
1. git branch -r -d origin/branch-name
2. git push origin :branch-name


