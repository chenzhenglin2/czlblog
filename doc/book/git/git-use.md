# git常见用法



## 正常提交流程

1，首先从GitHub/gitlab服务器拉下来
`git clone  仓库地址` 

如果使用ssh协议，还需要生成公私钥对，把公钥存储到仓库中

2，然后编辑和添加文件后
`git add . （所有文件）`
`git add filename(具体某个文件）`

3，然后提交
`git commit -m "message更新信息"`

若没有新增文件，只修改文件，上面两步可以合并为 git commit -am "message"
4，最后上传
`git push  [-u origin your_branch]`





## 把某个分支的文件copy到其他分支，如A分支文件copy到B分支

1,首先切换到B分支 

 `git checkout branchB `
2,在B分支下执行

` git checkout branchA（A分支） file1 file2 …`

3，剩下的步骤是提交和推送，和上面一样；



## 合并分支

把A分支所有的提交合并到B

首先同步A分支，使A分支本地和远程仓库保持一致

切换到B分支后

`git checkout B `

`git   merge   A `

然后A分支的内容就merge到B上了

如果有冲突，通过 git status 查看一下冲突文件，解决冲突，git add 冲突文件后，

剩下的提交和推送操作参照上面步骤





## 把A 分支某（几）次提交也提交到 B分支

在 A分支下执行
git log  查找到相关提交记录

切换到B分支，`git checkout B`

把A的某次提交，也提交到B：

`git cherry-pick 7f00fe9ebb（提交号）`

把A的某几次提交，也提交到B：

`git cherry-pick 7f00fe9ebb..7f00fe9ebb（提交范围）`

如果有冲突，通过 git status 查看一下冲突文件，解决冲突，git add 冲突文件后，剩下的提交和推送操作参照上面步骤

没有冲突直接执行 `git push  [-u origin your_branch]`





## 针对某次提交 打tag  

  1，使用git log查看提交日志，找出你需要的那个commit。假设提交的commit id

```
git checkout <commit id>
使用git tag进行打标签，例如：git tag -a v1.4 -m ‘xxxx’ 

git push origin --tags或者git push origin [tagname] 

git checkout -b  newbranch

git push origin newbranch
```



## git fetch  与 git pull

`   git fetch origin master:tmp `

 在本地新建一个temp分支，并将远程origin仓库的master分支代码下载到本地temp分支  

 `git push origin tmp`

 把tmp分支推送到远程仓库 



git pull  相当于 git fetch  + git merge



如    git  pull   tmp相当于 git  fetch origin/tmp + git merge origin/tmp 



把远程tmp merge到本地tmp分支中

## 本地仓库回退到某个版本　　

`git　　reset　　–hard　　bae168　`

   

## 创建空白新分支

```
git branch <new_branch>
git checkout <new_branch>

git rm --cached -r . 
git clean -f -d
创建空的commit
git commit --allow-empty -m "[empty] initial commit"
推送新分支
git push origin <new_branch> 
```

   

##  把修改存到缓存中

```
git stash 
git stash pop(这个会把缓存拿出来，删除缓存）
 或
git  stash 
git stash list(查看stash列表）
git  stash apply [列表名称]
```



## .gitignore 的设置：

很多时候，有些文件不需要提交，如开发人员本地环境配置，就用到了.gitignore   

只忽略dbg目录，不忽略dbg文件

dbg/

只忽略dbg文件，不忽略dbg目录

dbg

!dbg/        

   若把某些目录或文件加入忽略规则，按照上述方法定义后发现并未生效，原因是.gitignore只能忽略那些原来没有被追踪的文件，如果某些文件已经被纳入了版本管理中，则修改.gitignore是无效的。那么解决方法就是先把本地缓存删除（改变成未被追踪状态），然后再提交，这样就不会出现忽略的文件了。git清除本地缓存命令如下：

```
git rm -r --cached .
git add .
git commit -m 'message'
```



## 最后说一个 强制推送（操作前要慎之又慎）

```
git push -f  [-u origin your_branch]
```

这些功能 基本能覆盖到git日常90%使用，剩下的，可以网上搜索资料，或者在git bash 中使用帮助，如 git commit --help;