# 精简Unity Characters资源包，生成自己的第一人称预制体 #
在Unity3D的场景中需要漫游时，通常会使用Unity 3D的标准资源包中的Characters 资源，其中的 FirstPersonController 预制提供了完整的角色控制功能，使用起来十分方便，但是角色资源包的内容过多，比如：    

![](https://i.imgur.com/uQFDJNM.png)        

在只使用第一人称预制体的情况下，许多导入的资源和代码是使用不到的，因此可以通过适当的精简，生成第一人称预制，这样再次使用的时候就无需"臃肿"的Unity 官方Characters资源包，而是自己的精简资源包。    

在分析了第一人称预制体身上的核心代码 [FirstPersonController](https://blog.csdn.net/u013477973/article/details/80587704 "FirstPersonController") 后，可以看到代码中引用的其他类分别为：      

- **MouseLook** (获取鼠标位置输入，控制头部相机旋转)
- **FOVKick** (位于Standard Assets->Utility，提供角色加速和减速时，模拟视角微小变化功能)    
- **CurveControlledBob**（位于Standard Assets->Utility，提供角色走动时，模拟头部相机微小周期性的抖动）  
- **LerpControlledBob** （位于Standard Assets->Utility，提供角色在跳起落地后，模拟头部轻微抖动功能）    
- **CrossPlatfomInputManager** （位于Standard Assets->CrossPlatformInput->Scripts，静态类，处理跨平台输入）      

基于以上的分析，精简过程就很简单了，**保留 FirstPersonController脚本以及以上引用的类**，另外可以保留 跳跃，步行的音频文件 （位于Standard Assets->Characters->FirstPersonCharacter->Audio）,由于CrossPlatfomInputManager中引用了Standard Assets->CrossPlatformInput下的其他类，因此**保留CrossPlatformInput->Scripts整个文件夹**   

重新建立文件夹 FirstPersonPrefarb 将以上内容移到该文件夹内：      

![](https://i.imgur.com/9jkg0ZD.png)      

- **Audio**下为对应保留的音频文件
- **Prefarb**下为角色预制
- **Scripts**下包括 FirstPersonController.cs   MouseLook.cs CrossPlatformInput文件夹   Utility文件夹     

CrossPlatformInput文件夹与Standard Assets->CrossPlatformInput->Scripts文件夹下的内容一致         
Utility文件加下包括：     

- CurveControlledBob.cs
- LerpControlledBob.cs
- FOVKick.cs     

这里需要注意的是，以上**保留下来的脚本更改下命名空间**，区别于标准资源包的命名空间，最好是以当前脚本所在的文件夹路径作为对应命名空间，同时将预制体上的FirstPersonController.cs脚本更换成此时文件夹内的
FirstPersonController.cs 并将对应变量赋值。

这样完成后就可以将该文件夹作为资源包导出，方便以后的工程使用。比如，将刚才导出的资源包导入：        

![](https://i.imgur.com/CUa4C8a.png)        

资源包的大小为220K，相比于标准资源角色包23.3M，精简了许多
