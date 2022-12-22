## Git常用命令       
[学习手册](https://git-scm.com/book/zh/v2)  
**分支**   
分支本质上是指向提交对象的可变指针，每次提交时会创建一个提交对象，该对象包含着文件结构树对象（记录着文件结构目录和索引）指针和提交信息（提交者信息，时间等，同时会产生一个指向上次提交的指针，从而形成链式结构。 而创建新的分支，实际上就是创建一个新指针指向当前分支所包含的上一个提交对象。   
Git 通过一个名为HEAD的特殊指针来确定当前本地所在的分支，可以将HEAD当作当前本地所在分支的别名   
### git新建本地分支并推送远程    
---   
[Git 分支 - 分支的新建与合并案例详解](https://git-scm.com/book/zh/v2/Git-分支-分支的新建与合并#_basic_branching)
1. 在我们原有分支中新建分支。
```
git branch name //创建新分支
git checkout name //切换新分支
``` 
或者
```
git checkout -b name //新建分支并切换
``` 

2. 在远程是没有对应的分支的，我们需要对远程进行推送。

```
命令: git push origin local_name:remote_name
```
```
例子: git push --set-upstream origin feature/pay:feature/pay 
``` 
这样一来远程就有对应的分支了。tips：远程和本地分支名不需要强制一致。

总结: 流程为 新建分支->切换到对应分支->推送对应分支到远程->实现远程分支和本地分支想关联。

对远程分支和本地分支进行关联以后我们使用很多命令就不需要指定对应的分支参数，默认会使用你远程分支，这有利于我们节省时间去敲命令，但是不利于进行git命令的纠错。    

3. 查看各个分支当前所指的对象  
```
git log --oneline --decorate 
```   
4. git改变当前分支的HEAD  
```
git checkout tags/v1.21.0  
```   
5. git分支后续提交，拉取  
``` 
git push origin branchName   
git pull origin branchName 
```  
6. git 分支删除   
``` 
git branch -d name   //本地分支删除    
git push origin --delete name  //远程分支删除
```  
7. git 查看所有远程分支   
``` 
git branch -r
``` 
---        

8. git 拉取远程分支并创建本地关联分支   
```  
git fetch origin branchPathName:branchName  
git chechkout branchName
```  
9. git 合并产生冲突，放弃合并
```
git merge --abort
```

## 子模块  
父项目的子模块独立于项目，在做更新时与父项目的更新拉取独立



---
### Git fetch和git pull的区别:

都可以从远程获取最新版本到本地

1. Git fetch:只是从远程获取最新版本到本地,不会merge(合并)
``` 
$:git fetch origin master     
//从远程的origin的master主分支上获取最新版本到origin/master分支上     
$:git log -p master..origin/master   
//比较本地的master分支和origin/master分支的区别
$:git merge origin/master           
//合并
 ```
2. Git pull:从远程获取最新版本并merge(合并)到本地   
```
$:git pull origin master    
//相当于进行了 git fetch 和 git merge两步操作
实际工作中,可能git fetch更好一些, 因为在merge前,可以根据实际情况决定是否merge   
```  

### git 提交   
```  
提交前拉取，保证是最新的状态： 
git pull   
如果落后于当前版本，同步到最新，拉取也视为一次提交，添加提交信息： //merge to master [wq]
添加当前目录下所有的修改：   
git add *  
查看已经添加的内容：  
git status  
确认提交   
git commit -m"add info"  
推送   
git push    
```  

如果在提交前 pull,出现冲突：   
```  
Please, commit your changes or stash them before you can merge   
```  
说明已经有人提交到分支的内容与当前修改有冲突，解决：   
```  
git stash   
git pull
git stash pop 
```    
通过git stash将工作区恢复到上次提交的内容，同时备份本地所做的修改，之后就可以正常git pull了，git pull完成后，执行git stash pop将之前本地做的修改应用到当前工作区。然后正常提交。


### git修改文件名     
```  
git mv oldname newname  #重命名文件
git commit -m "重命名文件"
//如果需要推送到远端
git push origin 分支名  
```  

### git 取消 add   
add 之后取消    
``` 
git reset HEAD //整体回到上次一次操作(Add之前的状态)  
```   
### git add commit 后取消 push 
``` 
git reset --soft HEAD~1 (整体回到上一次commimt前 add 操作保留) 
git reset --hard HEAD~1 (整体回到上一次操作，commit，add 操作均不再保留)
``` 
### git 放弃本地修改 (add 前)    
``` 
git checkout .  //在add之前放弃所有的本地修改   
```

### git 本地回滚      
```  
git reset --hard HEAD~1   //回滚到上一次提交  
$ git reset --hard HEAD^ //回退到上个版本
$ git reset --hard HEAD~3 //回退到前3次提交之前，以此类推，回退到n次提交之前
$ git reset --hard commit_id //退到/进到 指定commit的sha码
```    
### git 忽略     
本地仓库根目录新建 .gitignore 文件    
一定要将 .gitignore 文件放到 git 工程的根目录，否则配置是不会生效的。
```  
touch .gitignore    
```     
写入需要忽略的文件/文件夹     
如果需要忽略的文件/文件夹 在之前已经提交过，则需要在暂存区中予以清除：   
```   
git rm -r --cached obj/       
git commit -m -a "删除不需要的文件"    
git push   
```   

## Git问题解决办法 ###
---

1.If no other git process is currently running, this probably means a
git process crashed in this repository earlier. Make sure no other git
process is running and remove the file manually to continue.  

**解决办法**  
```	
rm -f ./.git/index.lock   
```    
---

