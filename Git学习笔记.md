## Git常用命令       
[学习手册](https://git-scm.com/book/zh/v2)  
**分支**   
分支本质上是指向提交对象的可变指针，每次提交时会创建一个提交对象，该对象包含着文件结构树对象（记录着文件结构目录和索引）指针和提交信息（提交者信息，时间等，同时会产生一个指向上次提交的指针，从而形成链式结构。 而创建新的分支，实际上就是创建一个新指针指向当前分支所包含的上一个提交对象。   
Git 通过一个名为HEAD的特殊指针来确定当前本地所在的分支，可以将HEAD当作当前本地所在分支的别名   
### git新建本地分支并推送远程    
--- 
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

### git修改文件名     
```  
git mv oldname newname  #重命名文件
git commit -m "重命名文件"
//如果已经推送到远端，则需要
git push origin 分支名  
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

