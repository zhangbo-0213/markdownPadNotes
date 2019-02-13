### Unity3D 配合 SVN 版本控制流程###
**所需工具：**    

- SVN(Subversion)  
- SvnToolLite.unitypackage(Unity插件)    

**安装：**   

- 本机安装SVN客户端程序   
- Unity工程内导入SvnToolLite插件    

**Unity设置：**   
Edit-->Projects Seeting-->Version Control    
![](https://i.imgur.com/i8eS7xC.png)  
保证 **Editor Settings**中：     
**Version Control Mode**:Visible Meta Files   
**Asset Serialization**:Force Text  
第一次进入工程，安装完	SvnToolLite.unitypackage插件     
**Reposity** 会提示报错： Status 为 未能找到仓库        
关闭Unity工程后，将工程文件提交到SVN仓库后再打开工程文件，Status 为 连接到仓库状态   

**提交：**    


- 仅是对项目文件夹下Assets目录进行版本控制    
- 操作前先update本地的工作副本   
- 对于程序脚本等文本文件的更新操作，可以正常的使用SVN进行update和commit，可以对冲突进行比对合并，但**如果涉及到场景属性等设置上的修改**，这些文件上变化是无法进行版本合并的，建议先把要更新的对象锁起来，防止自己在做修改的同时别人也在向服务器commit    
- commit上去的文件会自动解锁，如果还有其他锁定的对象，勿忘手动解锁，尽量缩短占用的时间   


未完待续，遇到问题再做补充

 
