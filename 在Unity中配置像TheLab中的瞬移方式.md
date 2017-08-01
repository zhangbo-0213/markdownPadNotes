## 在Unity中配置像The Lab中的瞬移方式##
使用HTC Vive玩过The Lab虚拟现实游戏的朋友们对于游戏中使用TouchPad触摸板进行位置移动的方法一定不会感到陌生  
就是这个样子：
>![](http://i.imgur.com/nKBsK80.jpg)  

使用瞬移的方式完成移动在小空间大场景中的虚拟现实应用开发中非常受用，同时在玩家移动过程中产生的眩晕感较小。那么如何在Unity中开发出这样的瞬移的移动效果呢？

----------
这里需要使用到一个叫做ViveTeleporter的插件，当然如果你能通过脚本自己实现这个功能，下面的就不用看了，如果方便的话，最好能发个教程教教我（感激.jpg）  
这里是插件的地址 [链接：http://pan.baidu.com/s/1o7SwCEA ](链接：http://pan.baidu.com/s/1o7SwCEA ) 密码：nzsc  
解压后直接拖到工程里就OK了  
除了需要这个插件外，还需要开发HTC Vive必不可少的插件SteamVR Plugin,目前SteamVR Plugin更新到1.2.1的版本，但是这个版本貌似不太稳定，导入到工程里后，测试运行会发现头盔里面看不见手柄控制器，层次面板中的左右控制器也是灰色的，所以使用1.2.0版本避免上述尴尬  
SteamVR Plugin1.2.0 地址 [链接：http://pan.baidu.com/s/1c3YHXG ](链接：http://pan.baidu.com/s/1c3YHXG ) 密码:o45k  
直接导入到工程中就OK了   

----------
###下面进行工程的配置：###

-  删除场景中的Main Camera
-  将SteamVR Plugin中的Prefabs中的CameraRig拖入场景中
-  搭建一个平面充当行走地面

###完成以上过程开始配置ViveTeleporter###

-  将ViveTelePorter中Prefabs中的NavMesh和Pointer预制体拖入场景中
-  在CameraRig下找到Camera（eye）并**添加脚本Teleport Vive**并对其中的参数赋值：
>  将刚才拖入场景中的Pointer赋值给**pointer**  
>  将CameraRig赋值给**Origin Transform**  
>  将Camera（head）赋值给**Head Transform**  


- Fade Material是指定显示的小圆圈是什么颜色，导入的包里面有一个默认的Fade材质可以选择，当然也可以创建自己喜欢的材质
- Controllers就是指定手柄控制器了，可以指定一个，也可以指定两个

![](http://i.imgur.com/0ozMQhh.png)   
Camera(eye)上TelePort Vive的配置完成，这个是主要部分

----------

Pointer的配置：

![](http://i.imgur.com/9GV7Xx8.png)  
Pointer配置比较简单，指定一下NavMesh为刚才拖入的NavMesh就好了 

----------
以上过程完成后，打开Navgation面板将地形Bake一下就可以了，完成地形的导航烘焙后，选择NavMesh面板，点击更新导航数据按钮就可以了  
![](http://i.imgur.com/bg9MfHh.png)  
需要注意的是，每次我们对地形重新烘焙后，都需要在NavMesh里点击更新  

----------

更新完成后，地图上会出现方格点，表示可以到达的区域
![](http://i.imgur.com/1J3mUzX.png)

----------
到目前为止，配置已经完成（很简单不是吗），点击运行就可以使用控制器完成瞬移，就像The Lab中的那样  
![](http://i.imgur.com/eCLkyFl.png)

----------
END  ：）