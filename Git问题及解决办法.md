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
## Git常用命令  

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

---