## Unity Shader入门学习##
**学习教材：《UnityShader入门精要》——冯乐乐**  
**部分计算图例为《UnityShader入门精要》书中截图**  
**代码和实例截图均为个人实际操作得到**

----------

### 渲染流水线 ###
渲染流水线从概念部分分为三个部分：      

- **应用阶段**  
应用阶段为开发者完全控制部分，主要提供渲染所需要的渲染数据，输出为渲染图元，该阶段可以细分为： 
- 加载渲染数据（HDD-->RAM-->VRAM）    
- 设置渲染状态（决定场景中的网格（图元）以怎样的方式渲染，使用什么着色器，光照，材质）  
- 调用DrawCall命令（指定需要渲染的图元列表，发起方为CPU，接收方为GPU）   

- **几何阶段**  
几何阶段的部分过程可以由开发者控制和配置，几何阶段主要将接收到的图元信息进行逐顶点，逐多边形操作，将顶点坐标变换到屏幕空间，同时记录顶点的光照，深度，着色信息。该阶段可以细分为：   
- 顶点着色器（该阶段可编程阶段，将顶点坐标从模型空间转换到齐次裁剪空间，顶点着色，纹理坐标输出也在该过程完成）  
- 曲面细分着色器（可选着色器，细分图元）  
- 几何着色器（可选着色器，进行逐图元操作，或者被用于产生更多图元）  
- 裁剪（将不在摄像机视野中的顶点裁剪掉） 
- 屏幕映射（将图元的顶点转换到屏幕空间坐标系，该过程不可编程或配置）

- **光栅化阶段**  
该阶段接收上一阶段的顶点信息，并对顶点所围成的网格覆盖的像素进行逐像素的处理，输出最终的渲染图像，该过程可以细分为：
- 三角形设置（该阶段将得到的顶点进行计算，得到顶点围成的三角网格的边上的像素坐标）
- 三角形遍历（根据上一步的计算结果，判断哪些像素点在网格内，并对覆盖的像素点进行插值，这里由于每个像素点上除了颜色信息，还包括光照，深度，纹理坐标等信息，将带有信息的像素点成为片元，这一过程的输出为片元序列）
- 片元着色器（该阶段为可编程着色阶段，输入为上一阶段的顶点信息的插值结果，许多较为重要的渲染技术在该阶段完成，如纹理采样，该阶段若要进行纹理采样，那么在顶点着色器阶段输出每个顶点对应的纹理坐标）  
- 逐片元操作（该过程为渲染流水线的最后阶段，也被称作合并输出阶段，主要是对每个片元进行模板，深度测试，混合操作等，通过测试的可以选择是否与颜色缓冲区的颜色进行混合，从而决定片元的可见度以及最终的颜色）

对应流程图：    
![](https://i.imgur.com/5ZiCsJ5.png)       

**shader**是渲染流水线中的一部分可高度编程的阶段，**Unity**中的**Shader**主要对  几何阶段的**顶点着色器**和光栅化阶段的**片元着色器**进行编程操作。 

### Shader Lab ###
在Unity中，所有的Unity Shader都是使用ShaderLab来编写的。Shader Lab是Unity提供编写Unity Shader的一种说明性语言，使用嵌套在花括号内部的语义来描述Unity Shader文件的结构。   
一个Unity Shader的基础结构：
		
		Shader "ShaderName"{
			properties{
				//属性
			}
			SubShader{
				//显卡A使用的子着色器
			}
			SubShader{
				//显卡B使用的子着色器
			}
			FallBack "VertexLit"
		}  
Unity Shader和通用的Shader不太一样，Unity在背后根据使用的平台将这些结构编译成正真意义上的Shader代码和文件，Unity开发者不必太关心底层的渲染，只用使用Unity Shader Lab即可。   

### Unity Shader结构及语义 ###
**Properties**  
*Properties*语义块中包括一系列的属性（Property）, **这些属性会出现在材质面板中**，*Properties*语义块定义：

		Properties{
			_name("display name",PropertyType)=DefaultValue
			_name("display name",PropertyType)=DefaultValue
			//更多属性
		}

属性的声明可以使我们很方便的在材质面板中看到这些属性，并对这些属性进行调节，**display name**是该属性在材质面板中的显示的名字，若要在后续的CG代码中使用这些属性，则是通过**_name**进行访问，每种属性在声明时，需要指定属性的类型，并给附上默认值。完整的属性及类型和默认值赋值方式为：

		Sahder "ShaderLabProperties"{
			Properties{
		_Int("Int",Int)=2
		_Float("Float",Float)=1.5
		_Range("Range",Range(0.0,5.0))=3.0
		//
		_Color("Color",Color)=(1,1,1,1)
		_Vector("Vector",Vector)=(2,3,6,1)
		//
		_2D("2D",2D)=""{}
		_Cube("Cube",Cube)="white"{}
		_3D("3D",3D)="black"{}
		}
	}
**值得注意的是**，在Shader Lab的语义块中，每行语句结尾是没有；的，对于2D，3D，Cube这3种纹理类型，默认值的定义通过一个字符串后跟一对{}来完成，其中，字符串要么为空，要么为内置纹理名称，如"gray","red","bump"等。Properties语义块的作用只是将定义的属性显示到材质面板中， **后续的Shader代码中若要访问这些属性，需要在CG代码片段中定义和这些属性相匹配的变量。**

**SubShader**  
每一个Unity Shader文件可以包含1个或多个*SubShader* 语义块，这是由于不同的显卡具有不同的渲染能力，多个 *SubShader* 对应着多个显卡，这样在不同能力的显卡上进行不同复杂度的渲染计算。     
*SubShader*语义块通常结构如下：    
			
		SubShader{
		//可选  
		[Tags]

		//可选
		[RenderSetup] 

		Pass{
		}
		//Other Pass
	}
需要注意的地方：   

- SubShader包括一系列Pass,可选的Tags,可选的RenderSetup，每一个Pass为一次完整的渲染流程
- Tags为可选的，可以在SubShader和Pass内声明，SubShader内声明的Tags是特定的
- RenderSetup为可选的，可以在SubShader和Pass内声明，非特定的，即在SubShader和Pass内可通用
- Tags和RenderSetup若在SubShader中进行了设置，会应用到所有的Pass中去    

*Tags*是一个键值对，其结构为：
	
	Tags{"TagsName1"="Value1" "TagsName2"="Value2"}

**Pass语义块**  
Pass语义块为*SubShader*的一部分，语义结构如下：

		Pass{
		[Name]
		[Tags]
		[RenderSetup]
		//Other Code
		}  
在Pass语义块中，可以定义该段Pass的名字，如：

		[FirstPass]    
定义了Pass的名字后，可以通过  

		UsePass "FirstShader/FIRSTPASS"  
来使用其他Unity Shader的Pass，提高复用性。由于Unity Shader的内部会将所有的Pass的名称转换成大写形式，因此在使用UsePass时，使用大写的Pass名称。   
Pass语义块中，可以设置Tags，与SubShader中的不同，Pass中可用的Tags有LightMode,RequireOptions   
Pass语义块中，可以设置RenderSetup,与SubShader中的设置通用，应用于当前的Pass  
Unity Shader支持一些特殊的Pass，以实现代码复用和更为复杂的功能，如：  

- **UsePass:**使用该命令复用其他Unity Shader的Pass
- **GrabPass:**该Pass负责抓取屏幕将结果存储在一张纹理当中，用于后续Pass的处理

**Fallback**  
在最后一个SubShader的语义块后面，有一个Fallback指令，如果所有的SubShader都不能被当前显卡运行，那就使用Fallback指定的Shader，如：  

		Fallback "VertexLit"   

在Unity Shader中，真正意义上的Shader代码都会写在SubShader语义块中：    

		Shader "FirstShader"{
		Properties{
		//所需各种属性
		}
		SubShader{
		//真正意义上的Shader代码会写在这部分中
		//表面着色器(Surface Shader)或者
		//顶点/片元着色器（Vertex/Fragment Shader）或者
		//固定函数着色器(Fixed Function Shader)
		}
		SubShader{
		}
		}  

### 相关数学内容 ###
**变换**  
变换指将一些数据，例如，点，方向矢量甚至颜色，通过某种方式进行转换的过程。   
线性变换是非常常见的一种变化类型，满足矢量加和标量乘的变换即为线性变换，即：

**f(x)+f(y)=f(x+y)**     
k**f(x)**=**f(**k**x)**    

**缩放**和**旋转**是一种线性变换，**错切**，**镜像**，**正交投影**也是线性变化。  

**平移**不属于线性变化，比如**f**(x)=x+(1,2,3)，因为不满足矢量加和标量乘的运算。 

线性变换使用3X3的矩阵即可表示，但由于**平移**变换不属于线性变换，因此无法使用3X3的矩阵来表示平移，需要将矢量扩展到四维空间下，即**齐次坐标空间**  

三维坐标-->齐次坐标空间（*w*分量设为1）   
方向矢量-->齐次坐标空间 (*w*分量设为0)  
当使用4X4矩阵对一个点进行变换时，平移、旋转和缩放会施加于该点，而对于一个方向矢量，平移的效果则会被忽略。  

**基础变换矩阵**   
用于表示**纯平移**、**纯旋转**和**纯缩放**的的变换矩阵为**基础变换矩阵**。基础变换矩阵的分解：   
**M（3x3）**        **T(3x1)**     
**0 (1x3)**         **1**   
其中，**M（3x3）**用于表示旋转和缩放部分，**T（3x1）**用于表示平移部分  

**复合变换**   
平移、缩放和旋转可以组合起来，形成复杂的变化过程。由于矩阵的乘法不满足交换律，因此不同的变换顺序得到的结果是不一样的。绝大多数情况下约定变换顺序为：缩放，旋转，再平移即(Unity中矢量一般化为列矩阵，放在变换右侧运算)：  

**P（new）=M(translation) M(rotation) M(scale) P(old)**  

这样的运算顺序符合预期，例如位于原点的坐标，先向z平移5个单位,即(0,0,5)，再扩大两倍，得到（0,0,10），这样实际上位置已经不符合预期。因此先进行缩放，再旋转，最后平移。 

复合旋转同样需要注意旋转的顺序，当给定（θx,θy,θz）的旋转角度，得到的组合变换旋转矩阵为： 

**Mθz Mθx Mθy**  

**坐标空间的变换**  
定义一个坐标空间需要其原点位置和3个坐标轴的方向的表示。原点位置和坐标轴的表示实际上是相对于另一个坐标系而言。对坐标空间的变换实际上就是在父空间和子空间之间对点和矢量进行变换。即： 

**A**p=**Mc-p** **A**c (将子空间的坐标或矢量变换到父空间)  
**B**c=**Mp-c** **B**p (将父空间的坐标或矢量变化到子空间) 

**Mc-p**表示从子坐标空间变换到父坐标空间的变换矩阵，**Mp-c**是其逆矩阵。   

若已知子坐标空间**C**中的3个坐标轴在父坐标空间**P**下的表示 **Xc** **Yc** **Zc** 以及其原点位置 **Oc** ，给定一个子坐标空间中的一点Ac=(a,b,c)

坐标的变化过程：    
Ap=**Oc** +a **Xc** +b **Yc** +c **Zc**    
  =(Xoc,Yoc,Zoc)+a(Xxc,Yxc,Zxc)+b(Xyc,Yyc,Zyc)+c(Xzc,Yzc,Zzc)   
  =((Xoc,Yoc,Zoc)+  
  |Xxc Xyc Xzc| a  
  |Yxc Yyc Yzc| b   
  |Zxc Zyc Zzc| c   
原点的加法部分属于平移变换，因此转换到齐次坐标空间下：   
  |1 0 0 Xoc||Xxc Xyc Xzc 0| a  
  |0 1 0 Yoc||Yxc Yyc Yzc 0| b  
  |0 0 1 Zoc||Zxc Zyc Zzc 0| c  
  |0 0 0 __1|| _0  _0 _0 _1| 1  
左侧两个矩阵进一步合并：  
  |Xxc Xyc Xzc Xoc| a  
  |Yxc Yyc Yzc Yoc| b  
  |Zxc Zyc Zzc Zoc| c  
  | _0  _0 _0  ___1| 1  
由此，**Mc-p**即为：  
  |Xxc Xyc Xzc Xoc|   
  |Yxc Yyc Yzc Yoc|   
  |Zxc Zyc Zzc Zoc|   
  | _0  _0 _0  ___1|   

若是对**矢量**进行变换的话，平移的过程是不需要的，因为一个矢量平移没有意义，所以矢量的变换矩阵为前3X3：  
  |Xxc Xyc Xzc|   
  |Yxc Yyc Yzc|   
  |Zxc Zyc Zzc|   
因此在Shader中，经常会有截取变换矩阵的前3X3对法线方向、光照方向进行空间变换。  

这里值得注意的地方，假如**Mc-p**是一个正交矩阵，那么**Mp-c**即为**Mc-p**T （**Mc-p**-1 = **Mc-p**T）  
这里变换矩阵的 **Mc-p** 前3X3就是 **Xc** **Yc** **Zc** 分别以**列的形式** 填充得到的矩阵，若是正交矩阵， 以**行的形式** 填充矩阵即可得到 **Mp-c**  

**顶点的坐标空间变换过程**  


- **模型空间**    
每个模型都有自己独立的坐标空间，当模型移动或旋转的时候，模型空间也会跟着移动或旋转。    


- **世界空间**  
世界空间是关心的最外层的坐标空间，在Unity中，即为场景空间空间。这里有一点注意的是，在Unity中，游戏对象的Transform组件中显示的坐标信息是相对于其父对象的坐标空间的，如果该游戏对象没有父对象，则坐标信息为世界空间的坐标信息。  

- **观察空间**   
观察空间可以看做模型空间的一个特列，即摄像机，也可以被称为摄像机空间。Unity中，观察空间的坐标轴的方向为：+x轴指向右方，+y轴指向上方，+z轴指向摄像机的后方，注意**观察空间使用的是右手坐标系**，摄像机的正前方指向-z轴方向。   

- **裁剪空间**   
裁剪空间的目的是能够方便地对渲染图元进行裁剪。处于这个空间内的图元会被保留，位于这个空间外的就会被剔除，与这块相交的图元会被裁剪。裁剪空间由视锥体决定。 

- **屏幕空间**  
经过投影矩阵的变换后，可以进行裁剪操作。当裁剪完成后，进行真正的投影，将视锥体投影到屏幕空间，生成对应的2D坐标，得到真正的像素位置。

**顶点变换第一步：模型空间-->世界空间**   
根据游戏对象的Transform信息，得到游戏对象在世界空间中进行的缩放，旋转，平移信息，构建变换矩阵，从右往左，依次是缩放，旋转，平移，得到变换矩阵，顶点坐标根据该变换矩阵运算得到在世界空间下的表示。  


**顶点变换第二步：世界空间-->观察空间**  
取得从世界空间到观察空间的变换矩阵有两种方法。   
第一种是通过摄像机的Transform组件信息，得到从观察空间到世界空间的变换矩阵，再通过这个变换矩阵求出逆矩阵，即为世界空间到观察空间的变换矩阵。    
第二种是通过将摄像机回到世界空间的原点，坐标轴与世界空间坐标轴重合，得到一个变换矩阵，顶点通过与这个矩阵运算得到新的坐标位置，这个坐标的值即为在观察空间下的坐标值，**这里有一点需要注意**，由于观察空间是右手坐标系，因此得到的变换矩阵还是相当于左手坐标系下，因此还需要做一个变化，使Z轴反向。在该矩阵的基础上乘上：  
|1 0 0 0|  
|0 1 0 0|  
|0 0 -1 0|  
|0 0 0 1|  

**顶点变换的第三步：观察空间-->裁剪空间**   
观察空间到裁剪空间的转换过程中，不管是透视投影的是视锥体还是正交投影的视锥体，都有对应的透视矩阵进行运算。视锥体是摄像机能够看到的空间。分为两种类型，对应两种投影，分别是**透视投影** 和 **正交投影**。如果直接使用视锥体围成的空间对图元进行裁剪，不同的视锥体的处理方式就不一样，而且在透视投影的视锥体下判断一个图元是否在视锥体内计算非常麻烦。因此，通过一个投影矩阵将顶点转换到裁剪空间内能得到一个更加通用的处理过程。在经过投影矩阵的转换之后，齐次坐标空间的*w*分量会作为一个范围值，如果x,y,z的值在这个范围内，说明该顶点位于裁剪空间内。 

**顶点变换的第四步：裁剪空间-->屏幕空间**  
把顶点从裁剪空间投影到屏幕空间中，生成对应的2D坐标，有两个步骤。   
**步骤1：齐次除法（透视除法）**  
该过程使用齐次坐标系的*w*分量去除以x,y,z分量。这一步得到的坐标可以称为**归一化设备坐标（NDC）**。经过投影矩阵变换得到的裁剪空间，经过齐次除法后，变化到一个立方体内。  
**步骤2：屏幕映射**  
这时，变化后的坐标均在[-1,1]内，而在Unity中，屏幕空间的左下角坐标为（0,0），右上角坐标为（pixelWidth，pixelHeight）因此将坐标进行一个映射过程,即：   
Pixelx=（1/2 * Xndc+1/2） * pixelWidth    
Pixely=（1/2 * Yndc+1/2） * pixelHeight   

**顶点着色器** 的最基本任务是将顶点坐标从**模型空间** 变换到 **裁剪空间**，对应着3个变换矩阵，即**模型变换** ，**观察变换**，**投影变换**，在顶点着色器中通常将这三个矩阵串联成一个矩阵，即**MVP**，用于将顶点坐标从**模型空间** 变换到 **裁剪空间**。   

而从**裁剪空间** 到 **屏幕空间** 的转换过程是由Unity完成的，所以顶点着色器只完成顶点从模型空间到裁剪空间即可。  

完整的变换过程：             
![](https://i.imgur.com/nWnPA1S.jpg)

**法线变换**   
法线，即法矢量，变换矩阵可以变换一个顶点或一个方向矢量，但法线是需要特殊处理的一种矢量。   
切线，即切矢量，通常与纹理空间对齐，与法矢量垂直。由于切线是由两个顶点通过插值计算得到，因此针对于顶点的变换矩阵应用于切矢量的变换不会出现什么问题。但是法线的变换会面临变换后不垂直的问题。  

同一顶点的切线 **Ta** 与法线 **Na** 应保持垂直关系 ，即 **TaNa** =0。在给定变换**Ma-b**的情况下，变换后的顶点的切线 **Tb** 与法线 **Nb** 同样应保持垂直关系，**Tb** =**Ma-bTa** ，假定 **Na** 由**G**矩阵进行变换，变换后，**Nb**=**GNa**,变换后：  
**Ma-bTaGNa**=0  

**Ta(T)Ma-b(T)GNa**=0
**Ta(T)(Ma-b(T)G)Na**=0 

**(T)表示转置**
由于**TaNa**=0,**Ta**,**Na**均为方向矢量，即**Ma-b(T)G**=**I** 此时可以满足垂直要求，**G**=**Ma-b(T)** 的逆矩阵，即**G**是**Ma-b** 的转置逆矩阵。若**Ma-b**为正交矩阵，则**G**=**Ma-b**，也就是直接使用Ma-b进行法线的变换就可以保证变换后的垂直要求。  
如果变换只包含旋转变换，那么**Ma-b**是正交矩阵，或者这包含旋转和统一缩放，那么统一变换系数为k,则**G**=1/k(**Ma-b**),但如果变换中包含了非统一变换，则**G** 必须通过 **Ma-b** 的转置逆矩阵进行求解。   

### 顶点/片元着色器基本结构 ###
Unity Shader基本结构包含：Shader、Properties、SubShader、Fallback，即：  

	Shader "MySahder"{
		//属性部分，使材质面板可见
		Properties{
		}
		
		//SubShader A  
		SubShader{
			//通道Pass
			Pass{
			//设置渲染状态及标签

			//开始CG代码片段  
			CGPROGRAM  
			//该代码段的编译指令
			#pragma vertex vert   //指定顶点着色器函数
			#pragma fragment frag  //指定片元着色器函数

			//CG代码计算部分

			ENDCG
			//其他设置			
			}

			//其他通道Pass
		}

		//SubShader B  
		SubShader{
		
		}
		
		//回滚操作  
		Fallback"VertexLit"
	}
比较重要的是Pass块内的内容  

**第一个Shader**   

	Shader "Custom/Chapter5_SimpleShader" {
	SubShader{
		Pass{
		CGPROGRAM 
		#pragma vertex vert
		#pragma fragment frag  

		float4 vert(float4 v: POSITION) : SV_POSITION{
			return mul(UNITY_MATRIX_MVP,v);
		}

		fixed4 frag() : SV_Target{
			return fixed4(1.0,1.0,1.0,1.0);
		}

		ENDCG
	}
	}
	FallBack "Diffuse"
	}  


**vert** 函数的输入v包含顶点位置，通过**POSITION**语义指定  

**POSITION** 将模型的顶点坐标填充到输入参数v中   
**SV_POSITION** 顶点着色器的输出是裁剪空间中顶点坐标       
**SV_Target** 将用户的输出颜色存储到一个渲染目标中，这里会输出到默认的帧缓存

**POSITION** 和 **SV_POSITION** 均为CG/HLSL中的语义，语义告诉系统输入哪些值，输出哪些值，输出到哪里 

**通过语义获得更多模型数据**   
**TEXCOORD0**：使用模型第一套纹理坐标填充texcoord变量，纹理坐标可以用来访问纹理  
**NORMAL**：获得法线方向，用于计算光照   

			//通过一个结构体定义顶点着色器的输入
		struct a2v {
		    //使用POSITION语义，将模型空间的顶点坐标填充至vertex
			float4 vertex:POSITION;
			//使用NORMAL语义，将模型空间的顶点法线填充至normal（由于是矢量，这里使用float3）
			float3 normal:NORMAL;
			//使用TEXCOORD0语义，将模型的第一套纹理坐标填充texcoord变量
			float4 texcoord:TEXCOORD0;
		};  

创建自定义结构体语法：  

	struct StructName{
		Type Name:语义;
		Type Name:语义;
		Type Name:语义;
	};   

填充到这些语义中的数据来自于模型的**Mesh Render**组件，每帧调用DrawCall时，Mesh Render组件会将负责渲染的模型的数据发给Unity。  

使用更加丰富的语义，获得模型更多信息：  

	Shader "Custom/Chapter5_SimpleShader" {
	SubShader{
		Pass{
		CGPROGRAM 
		#pragma vertex vert
		#pragma fragment frag  

		//通过一个结构体定义顶点着色器的输入
		struct a2v {
		    //使用POSITION语义，将模型空间的顶点坐标填充至vertex
			float4 vertex:POSITION;
			//使用NORMAL语义，将模型空间的顶点法线填充至normal（由于是矢量，这里使用float3）
			float3 normal:NORMAL;
			//使用TEXCOORD0语义，将模型的第一套纹理坐标填充texcoord变量
			float4 texcoord:TEXCOORD0;
		};
		
		//使用一个结构体定义片元着色器的输出
		struct v2f {
			//SV_POSITION语义告诉Unity，pos中包含模型顶点在裁剪空间的坐标
			float4 pos:SV_POSITION;
			//COLOR0语义告诉Unity,color用于存储颜色信息
			fixed3 color : COLOR0;
		};

	      v2f vert(a2v v) {
			//声明输出结构
			v2f o;
			o.pos= mul(UNITY_MATRIX_MVP,v.vertex);
			//将法线方向映射到颜色中(法线矢量范围[-1,1],因此做一个映射计算)  
			o.color = v.normal*0.5 + fixed3(0.5,0.5,0.5);
			return o;
		}

		fixed4 frag(v2f i) : SV_Target{
			//将计算后的颜色显示出来
			return fixed4(i.color,1.0);
		}

		ENDCG
	}
	}
	FallBack "Diffuse"
}

顶点着色器的输入和输出都是通过定义结构体变量对数据进行传递，当需要传递的数据不止一个时，需要使用定义结构体变量对多个数据进行传递。当顶点着色器的返回数据为一个结构时，方法的返回值指定需要返回的结构体类型，方法的参数列表后无需SV：POSITION语义，因为此时输出的数据不仅仅只是模型裁剪空间的顶点坐标了。

绑定到新建材质上，得到的效果： 

![](http://i.imgur.com/hlnCYMn.png) 

**添加属性**  
Shader中添加属性，可以在材质面板上看到对应的变量，并可以通过材质面板为该属性赋值。在计算输出时，需要在CG代码中，定义一个与属性名称和类型的相同的变量，才能在计算时使用该变量。   

	Properties{
		_Color("Color Tint",Color) = (1.0,1.0,1.0,1.0)
	}  
CG代码片段中：  
	
	CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag 
	//在CG代码中，需要定义一个与属性名称和类型匹配的变量
			fixed4 _Color;  
	fixed4 frag(v2f i) : SV_Target{
			//将计算后的颜色显示出来
			fixed3 c= i.color;
			//使用_Color属性控制颜色属性
			c *= _Color.rgb;
			return fixed4(c,1.0);
		}
	ENDCG  

效果:  

![](http://i.imgur.com/wOinvWs.png)  

### Unity中的基础光照 ###
**着色**是根据材质属性（高光反射属性，漫反射属性等）、光源信息（如光源方向、辐照度等），使用**等式**去计算沿某个观察方向的出射度的过程。该等式即 **光照模型（Lighting Model）** 。   
 
光照模型中包含不同的部分来计算光线经过物体表面后不同的方向   
**高光反射部分**：表示光线在物体表面如何被反射   
**漫反射部分**：表示光线在物体表面如何被折射，吸收和散射出表面 

**标准光照模型**     
标准光照模型将进入到摄像机内的光线分为4部分，每部分使用一种方法来计算其贡献度。   
   
- **自发光（emissive）**  
该部分用于描述当给定一个方向时，表面本身会像该方向发射多少辐射量。（如果没有使用全局光照，自发光的表面不会照亮周围物体，只会使本身看起来更亮）  
标准光照模型中，自发光直接使用材质的自发光颜色：
C(emissive)=M(emissive)
- **高光反射（specular）**   
该部分用于描述光源照射到物体表面时，该表面在完全镜面反射的方向反射出多少能量。高光反射需要知道的信息比较多，有法线方向（ **n** ）、入射方向( **i** )，反射方向( **r** )，视角方向( **v** )等。反射方向（ **r** ）可由入射方向（ **i** ）和法线方向( **n** )计算出来：    
![](http://i.imgur.com/KjnHeFI.png) 
![](http://i.imgur.com/FoHSWg7.png)  

计算出反射方向后，由**Phong**模型计算高光反射部分：  
![](http://i.imgur.com/JqrjJKN.png)        
M（gloss）是材质的光泽度，也成为反光度  

**Blinn**模型提出一个简单的修改方法得到类似效果，避免计算反射方向，引入一个新的矢量，对入射方向和视角方向求和后归一化得到：  
![](http://i.imgur.com/DsgBsxt.png)
![](http://i.imgur.com/sOGtzHw.png)           
然后使用法线方向和新的矢量进行计算而不是反射方向和视角方向：  
![](http://i.imgur.com/aVso0PK.png)

- **漫反射（diffuse）**   
该部分用于描述当光线从光源照射到物体表面时，会向**每个方向**散射的能量。漫反射光照复合Lambert定律：反射光线强度与表面法线和光源之间夹角余弦成正比  
![](http://i.imgur.com/gcZoOC1.png)        
使用 **Max** 函数是防止法线和光源方向点乘的结果为负，使物体前面被从背面来的光源照亮。
- **环境光（ambient）**  
该部分用于描述其他的间接光照。标准光照模型中，使用一种被称为环境光的部分模拟间接光照。环境光使用全局变量，场景中所有物体都使用这个环境光。   
C(ambient)=g(ambient)  

光照计算可以在顶点着色器中计算，称为**逐顶点光照**，也可以在片元着色器中计算，称为**逐像素光照**。
标准光照模型仅仅是一个经验模型，并不完全符合真实世界中的光照现象。这种模型的局限性在于：     

- 重要的物理现象无法表现出来，如菲涅尔反射
- 该模型**各项同性**，固定视角和光源方向旋转表面时，反射不会发生任何改变。而有些表面是具有**各项异性**的，如拉丝金属等  

不过由于易用性、计算速度和得到效果都比较好，被广泛应用。

**漫反射光照模型实现** 

**逐顶点漫反射实现**   

	// Upgrade NOTE: replaced '_World2Object' with 'unity_WorldToObject'
	// Upgrade NOTE: replaced 'mul(UNITY_MATRIX_MVP,*)' with 'UnityObjectToClipPos(*)'

	Shader "Custom/Chapter6_DiffuseVertexLevel" {
	
	Properties{
		_Diffuse("DiffuseColor",Color) = (1.0,1.0,1.0,1.0)
	}

		SubShader{
		Pass{
		//定义光照模式，只有正确定光照模式，才能得到一些Unity内置光照变量
		Tags{"LightMode" = "ForwardBase"}

		CGPROGRAM
		#pragma vertex vert
		#pragma fragment frag
		//使用Unity的内置包含文件，使用其内置变量
		#include "Lighting.cginc"

		//定义与属性相同类型和相同名称的变量
		fixed4 _Diffuse;
		//定义顶点着色器输入结构体
	struct a2v {
		float4 vertex:POSITION;
		float3 normal:NORMAL;
	};
		//定义顶点着色器输出结构体（片元着色器输入结构体）
	struct v2f {
		float4 pos:SV_POSITION;
		fixed3 color : COLOR;
	};

	v2f vert(a2v v) {
		v2f o;
		o.pos = UnityObjectToClipPos(v.vertex);

		fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;
		//通过模型到世界的转置逆矩阵计算得到世界空间内的顶点法向方向（v.normal存储的是模型空间内的顶点法线方向）
		fixed3 worldNormal = normalize(mul(v.normal,(float3x3)unity_WorldToObject));
		//得到世界空间内的光线方向
		fixed3 worldLight = normalize(_WorldSpaceLightPos0.xyz);

		//根据Lambert定律计算漫反射 saturate函数将所得矢量或标量的值限定在[0,1]之间
		fixed3 diffuse = _LightColor0.rgb*_Diffuse.rgb*saturate(dot(worldNormal,worldLight));

		o.color = diffuse + ambient;
		return o;
	}

	fixed4 frag(v2f i) :SV_Target{
		return fixed4(i.color,1.0);
	}

		ENDCG
	}
		
	}
	Fallback"Diffuse"
	}  
实现效果：  
![](http://i.imgur.com/jVJrBi7.png)

**逐像素漫反射实现**    

	Shader "Custom/Chapter6_DiffusePixelLevel" {
	
	Properties{
		_Diffuse("DiffuseColor",Color) = (1.0,1.0,1.0,1.0)
	}

		SubShader{
		Pass{
		//定义光照模式，只有正确定光照模式，才能得到一些Unity内置光照变量
		Tags{ "LightMode" = "ForwardBase" }

		CGPROGRAM
		#pragma vertex vert
		#pragma fragment frag
		//使用Unity的内置包含文件，使用其内置变量
		#include "Lighting.cginc"

		//定义与属性相同类型和相同名称的变量
		fixed4 _Diffuse;
		//定义顶点着色器输入结构体
	struct a2v {
		float4 vertex:POSITION;
		float3 normal:NORMAL;
	};
		//定义顶点着色器输出结构体（片元着色器输入结构体）
	struct v2f {
		float4 pos:SV_POSITION;
		float3 worldNormal:TEXCOORD0;
	};

	v2f vert(a2v v) {
		v2f o;
		o.pos = UnityObjectToClipPos(v.vertex);
		//存储世界空间下的法线，传递给片元着色器，
		//只有顶点着色器才能接受来自模型空间的数据，因此需要先计算再传递给片元着色器
		o.worldNormal =mul(v.normal,_World2Object);
		return o;
	}

	fixed4 frag(v2f i) :SV_Target{
		fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;
		fixed3 worldNormal = normalize(i.worldNormal);
		fixed3 worldLight = normalize(_WorldSpaceLightPos0);

		fixed3 diffuse = _LightColor0.rgb*_Diffuse.rgb*saturate(dot(worldNormal,worldLight));

		fixed3 color = diffuse + ambient;
		return fixed4(color,1.0);
	}

		ENDCG
	}

	}
		Fallback"Diffuse"
	}  

实现效果：  
![](http://i.imgur.com/mB1EjhO.png)  
左侧为逐顶点实现效果，右侧为逐像素实现效果。可以看到，在逐顶点效果中，模型下部背光面与向光面的交界处会出现不平滑的锯齿过渡。而逐像素则交界处相对平滑。但以上两种均存在在光线无法到达的区域，模型外观通常全黑，没有明暗变化，使背光区域看起来像平面，为此提出一项改进技术——**Half Lambert**光照模型。 

**半兰伯特(Half Lambert)光照模型**   
广义半兰伯特光照模型计算公式：    
![](http://i.imgur.com/TwRonRY.png)   
半兰伯特没有使用max函数进行限制负值，对点积结果进行缩放后再做偏移。一般情况下，α和β的取值为0.5,0.5,这样将[-1,1]的结果映射到[0,1]。这样在原有兰伯特基础上，背光面取值为负的顶点在半兰伯特下，有了对应的到[0,0.5]的值，**半兰伯特**并没有物理基础，仅仅只是视觉加强技术而已。  

半兰伯特实现：  

	fixed halfLambert = dot(worldNormal, worldLight)*0.5 + 0.5;
	fixed3 diffuse = _LightColor0.rgb*_Diffuse.rgb*halfLambert;  

实现效果:  
![](http://i.imgur.com/nosxzYs.png)    

**高光反射光照模型实现**   

**逐顶点高光反射实现**  

	Shader "Custom/Chapter6_SpecularVertexLevel" {
	Properties{
		_Diffuse("DiffuseColor",Color)=(1,1,1,1)
		_Specular("SpecualrColor",Color)=(1,1,1,1)
		_Gloss("Gloss",Range(8.0,256))=20
	}

		SubShader{
		Pass{
		Tags{"LightMode"="ForwardBase"}
		
		CGPROGRAM

		#pragma vertex vert
		#pragma fragment frag

		#include "Lighting.cginc"

		fixed4 _Diffuse;
		fixed4 _Specular;
		float _Gloss;

		struct a2v {
			float4 vertex:POSITION;
			float3 normal:NORMAL;
		};

		struct v2f {
			float4 pos:SV_POSITION;
			fixed3 color : COLOR;

		};

		v2f vert(a2v v) {
			v2f o;
			o.pos = mul(UNITY_MATRIX_MVP,v.vertex);

			fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;

			fixed3 worldNormal = normalize(mul(v.normal,(float3x3)_World2Object));
			fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);  

			fixed3 diffuse = _LightColor0.rgb*_Diffuse.rgb*saturate(dot(worldNormal,worldLightDir));

			//通过CG提供的reflect(i,n)提供的函数计算反射方向
			fixed3 reflectDir = normalize(reflect(-worldLightDir,worldNormal));
			//计算世界空间下的观察方向
			fixed3 viewDir = normalize(_WorldSpaceCameraPos.xyz-mul(_Object2World,v.vertex).xyz);
			//计算高光反射部分
			fixed3 specular = _LightColor0.rgb*_Specular.rgb*pow(saturate(dot(viewDir, reflectDir)), _Gloss);

			o.color=ambient+diffuse+specular;		

			return o;
		}

		fixed4 frag(v2f i) :SV_Target{
			return fixed4(i.color,1.0);
		}

		ENDCG
		}
	}

	Fallback "Specular"
	}   
实现效果： 
![](http://i.imgur.com/5P4Z4vP.png)    
可以看出，逐顶点的高光反射在高光部分出现非常明显的锯齿不连续现象，由于高光计算部分pow（）是非线性，因此顶点插值后的结果不理想。   

**逐像素高光反射实现**  

	// Upgrade NOTE: replaced '_Object2World' with 'unity_ObjectToWorld'
	// Upgrade NOTE: replaced '_World2Object' with 'unity_WorldToObject'
	// Upgrade NOTE: replaced 'mul(UNITY_MATRIX_MVP,*)' with 'UnityObjectToClipPos(*)'

	Shader "Custom/Chapter6_SpecularPixelLevel" {
	Properties{
		_Diffuse("DiffuseColor",Color)=(1,1,1,1)
		_Specular("SpecualrColor",Color)=(1,1,1,1)
		_Gloss("Gloss",Range(8.0,256))=20
	}

		SubShader{
		Pass{
		Tags{"LightMode"="ForwardBase"}
		
		CGPROGRAM

		#pragma vertex vert
		#pragma fragment frag

		#include "Lighting.cginc"

		fixed4 _Diffuse;
		fixed4 _Specular;
		float _Gloss;

		struct a2v {
			float4 vertex:POSITION;
			float3 normal:NORMAL;
		};

		struct v2f {
			float4 pos:SV_POSITION;
			fixed3 worldNormal : TEXCOORD0;
			fixed3 viewDir : TEXCOORD1;

		};

		v2f vert(a2v v) {
			v2f o;
			o.pos = UnityObjectToClipPos(v.vertex);
			 o.worldNormal = normalize(mul(v.normal,(float3x3)unity_WorldToObject));
  		
			//计算世界空间下的观察方向
			o.viewDir = normalize(_WorldSpaceCameraPos.xyz-mul(unity_ObjectToWorld,v.vertex).xyz);
			
			return o;
		}

		fixed4 frag(v2f i) :SV_Target{
			fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;

			fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);
			fixed3 worldNormal = i.worldNormal;

			fixed3 diffuse = _LightColor0.rgb*_Diffuse.rgb*saturate(dot(worldNormal, worldLightDir));

			//通过CG提供的reflect(i,n)提供的函数计算反射方向
			fixed3 reflectDir = normalize(reflect(-worldLightDir, worldNormal));
			fixed3 viewDir = i.viewDir;
			//计算高光反射部分
			fixed3 specular = _LightColor0.rgb*_Specular.rgb*pow(saturate(dot(viewDir, reflectDir)), _Gloss);
			return fixed4(ambient+diffuse+specular,1.0);
		}

		ENDCG
		}
	}

	Fallback "Specular"
	} 
实现效果：
![](http://i.imgur.com/qItJc7S.png)  
右侧为逐像素高光效果，高光边缘连续无明显锯齿现象  

**Blinn-Phong光照模型**   
Blinnn模型没有使用反射方向，引入新的变量 **h** 通过对光照方向**i** 和 视角方向 **v** 求和后归一化得到，
![](http://i.imgur.com/nDyUh3G.png)  
Blinn模型高光计算：  
![](http://i.imgur.com/zB68zTP.png)
Blinn-Phong逐像素实现： 

	// Upgrade NOTE: replaced '_Object2World' with 'unity_ObjectToWorld'
	// Upgrade NOTE: replaced '_World2Object' with 'unity_WorldToObject'
	// Upgrade NOTE: replaced 'mul(UNITY_MATRIX_MVP,*)' with 'UnityObjectToClipPos(*)'

	Shader "Custom/Chapter6_SpecularBlinnPixelLevel" {
	Properties{
		_Diffuse("DiffuseColor",Color)=(1,1,1,1)
		_Specular("SpecualrColor",Color)=(1,1,1,1)
		_Gloss("Gloss",Range(8.0,256))=20
	}

		SubShader{
		Pass{
		Tags{"LightMode"="ForwardBase"}
		
		CGPROGRAM

		#pragma vertex vert
		#pragma fragment frag

		#include "Lighting.cginc"

		fixed4 _Diffuse;
		fixed4 _Specular;
		float _Gloss;

		struct a2v {
			float4 vertex:POSITION;
			float3 normal:NORMAL;
		};

		struct v2f {
			float4 pos:SV_POSITION;
			fixed3 worldNormal : TEXCOORD0;
			fixed3 viewDir : TEXCOORD1;

		};

		v2f vert(a2v v) {
			v2f o;
			o.pos = UnityObjectToClipPos(v.vertex);
			 o.worldNormal = normalize(mul(v.normal,(float3x3)unity_WorldToObject));
  		
			//计算世界空间下的观察方向
			o.viewDir = normalize(_WorldSpaceCameraPos.xyz-mul(unity_ObjectToWorld,v.vertex).xyz);
			
			return o;
		}

		fixed4 frag(v2f i) :SV_Target{
			fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;

			fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);
			fixed3 worldNormal = i.worldNormal;

			fixed3 diffuse = _LightColor0.rgb*_Diffuse.rgb*saturate(dot(worldNormal, worldLightDir));
			
			fixed3 viewDir = i.viewDir;

			fixed3 halfDir = normalize(worldLightDir+viewDir);
			//计算高光反射部分
			fixed3 specular = _LightColor0.rgb*_Specular.rgb*pow(saturate(dot(halfDir, worldNormal)), _Gloss);
			return fixed4(ambient+diffuse+specular,1.0);
		}

		ENDCG
		}
	}

	Fallback "Specular"
	}   
实现效果：  
![](http://i.imgur.com/unl0D0Z.png)    
最右侧为逐像素Blinn-Phong效果，可以看出高光的光晕面积相对比Phong的要大一些。实际渲染中，大多数情况下选择Blinn-Phong模型。

在光照模型中，需要得到光源方向、视角方向等基本方向信息。通过自行计算的方式得到，如：  

	fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);
	
视角方向  
	
	o.viewDir = normalize(_WorldSpaceCameraPos.xyz-mul(unity_ObjectToWorld,v.vertex).xyz);
如果处理更加复杂的光照（如点光源和聚光灯），计算方向就是错误的。Unity提供了一些内置函数来计算这些信息。  
![](http://i.imgur.com/LkqnRkd.png)  

### 基础纹理 ###
通常使用一张纹理来代替物体的漫反射颜色。使用纹理的Shader中，需要对纹理进行采样。
使用CG的tex2D(_MainTex,uv)函数进行纹理采样，第一个参数是需要被采样的纹理，第二是float2类型的纹理坐标，该坐标在顶点着色器中由_MainTex_ST对定点纹理坐标进行变换得到。  

	Shader "Custom/Chapter7_SingleTexture" {
		Properties{
			_Color("Color",color) = (1,1,1,1)
			_MainTex("Main Tex", 2D) = "white"{}
			_Specular("Specular", color) = (1, 1, 1, 1)
			_Gloss("Gloss", Range(8.0, 256)) = 20
		}
			SubShader{
				Pass{
				Tags{"LightMode" = "ForwardBase"}

				CGPROGRAM
				#pragma vertex vert
				#pragma fragment frag
				#include "Lighting.cginc"

				fixed4 _Color;
				sampler2D _MainTex;
				//纹理类型的属性需要再声明一个float4类型的变量，命名格式为 纹理名_ST 
				//其中，ST是缩放和偏移的缩写 纹理名_ST.xy存储的是缩放值，纹理名_ST.zw存储的是偏移值
				//对应材质面板的纹理属性的Tiling和Offset调节项
				float4 _MainTex_ST;
				fixed4 _Specular;
				float   _Gloss;  

				struct a2v {
					float4 vertex:POSITION;
					float3 normal:NORMAL;

					//使用texcoord变量存储模型的第一组纹理坐标
					float4 texcoord:TEXCOORD0;  
				};

				struct v2f {
					float4 pos:SV_POSITION;
					float3 worldNormal:TEXCOORD0;
					float3 worldPos:TEXCOORD1;
					float2 uv:TEXCOORD2;

				};

				v2f vert(a2v v) {
					v2f o;
					o.pos = mul(UNITY_MATRIX_MVP, v.vertex);
					o.worldNormal = UnityObjectToWorldNormal(v.normal);
					o.worldPos = mul(_Object2World, v.vertex).xyz;
					//对纹理坐标进行变换，对应材质面板的Tiling和Offset调节项
					o.uv = v.texcoord.xy*_MainTex_ST.xy + _MainTex_ST.zw;
					//也可以使用内置函数  o.uv=TRANSFORM_TEX(v.texcoord,_MainTex); 计算过程一样

					return o;
				}
				fixed4 frag(v2f i) :SV_Target{
					fixed3 worldNormal = normalize(i.worldNormal);
					fixed3 worldLightDir = normalize(UnityWorldSpaceLightDir(i.worldPos));

					//使用tex2D做纹理采样，将采样结果和颜色属性_Color相乘作为反射率
					fixed3 albedo = tex2D(_MainTex,i.uv).rgb*_Color.rgb;

					fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz*albedo;

					fixed3 diffuse = _LightColor0.rgb*albedo*max(0,dot(worldNormal,worldLightDir));

					fixed3 viewDir = normalize(UnityWorldSpaceViewDir(i.worldPos));
					fixed3 halfDir = normalize(worldLightDir+viewDir);
					fixed3 specular = _LightColor0.rgb*albedo*pow(saturate(dot(halfDir,worldNormal)), _Gloss);

					return fixed4(ambient+diffuse+specular,1.0);
				}
				ENDCG		
		}
		}

		Fallback "Specular"
	}
实现效果：  

![](http://i.imgur.com/y27fUwB.png)   
![](http://i.imgur.com/WQ6pMfe.png)      

**Warp Mode**属性  
每张纹理在导入到Unity后，在纹理的检视面板中有Warp Mode属性，该属性决定了当纹理坐标超过[0,1]范围后如何被平铺，有Repeat（重复平铺）,Clamp（截取平铺）截取平铺是当超过1后的纹理坐标的对应的顶点颜色值均为1处的值。

### 凹凸映射 ###
凹凸映射的目的是使用一张纹理来修改模型表面的法线。这种方法不会真正该改变模型顶点位置，使模型看起来具有凹凸的效果，这点从模型的轮廓可以看出来。   
凹凸映射的两种方法：  

- **高度纹理**   
使用一张高度纹理来模拟表面位移，得到一个修改后的法线值，该方法也叫“高度映射"。  
高度图中存储的是强度值，用于表示模型表面的海拔高度。颜色越浅表示越向外凸起，颜色越深越向里凹。实时计算中，不能直接得到表面法线，需要由像素的灰度值计算得到，需要消耗更多性能。
- **法线纹理**  
使用一张法线纹理直接存储表面法线，该方法也叫“法线映射”。   
法线纹理中存储的是表面的法线方向，法线矢量值范围[-1,1]，纹理中的颜色值范围[0,1]，因此将法线存储在一张纹理中，有一个映射过程：   
**pixel= ( normal+1)/2**    
因此在对法线纹理进行采样得到像素颜色后，为了得到对应的发现方向，需要进行反映射过程   
**normal=2(pixel)-1** 

**法线方向所在的坐标空间**    

- 模型空间的法线纹理  
模型顶点自带的法线，定义在模型空间中，若将修改后的模型空间中的表面法线存储在一张纹理中，则为模型空间中的法线纹理   
- 切线空间的法线纹理      
![](https://i.imgur.com/SMbT2IX.png)    
模型的每一个 顶点有自己对应的切线空间，z轴为顶点的法线方向，x轴为顶点的切线方向，y轴为顶点的副切线方向，由法线和切线的叉积得到，这种纹理被称为切线空间的法线纹理。    

![](https://i.imgur.com/mjraezn.png)     
模型空间下的法线纹理颜色比较丰富，这是因为模型空间下各顶点的的法线所在的坐标系为同一个坐标系，而各个顶点法线方向各异，因此映射过后颜色相对比较丰富。    
而切线空间下的法线纹理颜色集中在浅蓝色，这是由于各自顶点都有自己的坐标系，法线的方向尽管在同一个坐标下方向各异，但在自身的切线空间坐标系下，法线方向均指向z轴，因此映射到纹理上，像素颜色单一。   

使用哪种坐标空间只是第一步，得到法线信息是为了转化到相应的坐标系（如世界空间）后进行后续的光照计算。  

使用模型空间法线纹理**优势**：   

- 直观  
转化到其他坐标空间计算相对简单。    
- 在纹理坐标的边缘处，可见突变较少。   
模型空间下的法线纹理是在同一个坐标系下，边缘处可以通过插值得到平滑效果。   

使用切线空间法线纹理**优势**：  

- 重用性较高。  
模型空间下法线纹理存储的是绝对法线信息，只能作用于创建时对应的模型，应用到其他模型上无法得到正确的效果。而切线空间的法线纹理的坐标系为各自顶点的坐标系，是一个相对法线信息，应用到其他的模型或者一个砖块使用一张法线纹理应用到6个面上也能得到合理的结果。  
- 可进行UV动画    
切线空间纹理中的法线方向是根据对应纹理的纹理坐标方向得到。因此可以通过移动一个纹理的UV坐标实现凹凸移动的效果。  
- 可压缩    
切线空间下的法线纹理中，z轴方向总为正方向，因此可以可以仅存储x,y轴方向，通过计算再得到z方向。  

切线空间下的法线纹理在使用上更加灵活，因此基本上选用切线空间作为法线纹理的坐标系。  

**光照模型计算中坐标空间的选择**    
由于在计算光照过程中，需要在一个统一的空间下进行，而法线纹理所在的空间为切线空间。因此有两种方式选择。   
 
- 将光照方向和视角方向变换到切线空间进行，这个过程在顶点着色器中可以完成。  
- 将法线方向变换到世界空间中进行，这时需要先对发现纹理进行采样，所以该过程在片元着色器中进行，并在片元着色器中进行一次矩阵运算。  

**切线空间下的光照模型计算**  
由于法线纹理中使用的本身就是切线空间，因此需要将光照方向和观察方向先转换到切线空间，这个过程可以在顶点着色器中完成。模型-->切线空间下的转换矩阵计算，首先切线空间-->模型空间的变换矩阵为模型顶点的切线方向（x轴）、副切线方向（y轴）、法线方向（z轴）的**按列排列**形式，即模型-->切线空间变换矩阵的逆矩阵，而对于一个方向矢量而言，一个变换矩阵若只存在平移和旋转变换，则该矩阵为一个正交阵，即变换矩阵的逆矩阵与转置矩阵相等，因此模型-->切线空间的变换矩阵为逆矩阵的转置，即将模型顶点的切线方向（x轴）、副切线方向（y轴）、法线方向（z轴）的 **按行排列**。   
实例代码：  

		// Upgrade NOTE: replaced 'mul(UNITY_MATRIX_MVP,*)' with 'UnityObjectToClipPos(*)'
	Shader "Custom/Chapter7_NormalMapTangentSpace" {
	Properties{
		_Color("Color",Color)=(1,1,1,1)
		_MainTex("MainTex",2D) ="white" {}
		_BumpTex("Noraml Tex",2D) = "bump"{} //bump为Unity自带的法线纹理，当没有提供任何法线时，"bump"就对应模型自身的法线信息
		_BumpScale("BumpScal",Float) = 1.0  //BumpScale代表凹凸程度，值为0时，表示该法线纹理不会对光照产生任何影响
		_Specular("Specular",Color) = (1,1,1,1)
		_Gloss("Gloss",Range(8.0,256)) = 20
	}
		SubShader{
			Pass{
				Tags{"LightMode" = "ForwardBase"}

				CGPROGRAM

				#pragma vertex vert
				#pragma fragment frag
				#include "Lighting.cginc"

				fixed4 _Color;
				sampler2D _MainTex;
				float4 _MainTex_ST;
				sampler2D _BumpTex;
				float4 _BumpTex_ST;
				float _BumpScale;
				fixed4 _Specular;
				float _Gloss;  

				struct a2v {
					float4 vertex:POSITION;
					float3 normal:NORMAL;
					float4 tangent:TANGENT;   //tangent存储顶点的切线方向，float4类型，通过tangent.w分量决定副切线的方向性
					float4 texcoord:TEXCOORD0;
				};

				struct v2f {
					float4 pos:SV_POSITION;
					float4 uv:TEXCOORD0;
					float3 lightDir:TEXCOORD1;
					float3 viewDir:TEXCOORD2;
				};

				v2f vert(a2v v) {
					v2f o;
					o.pos = UnityObjectToClipPos(v.vertex);
					o.uv.xy = v.texcoord.xy*_MainTex_ST.xy + _MainTex_ST.zw;//(uv.xy存储主纹理坐标变换后的uv坐标)
					o.uv.zw = v.texcoord.xy*_BumpTex_ST.xy + _BumpTex_ST.zw;//(uv.zw存储法线纹理坐标变换后的uv坐标)
					//_MainTex和_BumpTex通常会使用同一组纹理坐标（法线纹理贴图由对应纹理贴图生成） 

					float3 binormal = cross(normalize(v.normal),normalize(v.tangent.xyz))*v.tangent.w;
					float3x3 rotation = float3x3(v.tangent.xyz, binormal,v.normal);
					//(按行填充，得到的矩阵实际上是模型到切线的逆矩阵的转置矩阵，也就是模型到切线的转换矩阵(正交矩阵))
					//也可以使用内建宏TANGENT_SPACE_ROTATION得到变换矩阵 

					o.lightDir = mul(rotation, ObjSpaceLightDir(v.vertex)).xyz;
					o.viewDir = mul(rotation,ObjSpaceViewDir(v.vertex)).xyz;

					return o;
				} 

				fixed4 frag(v2f i):SV_TARGET {
					fixed3  tangentLightDir = normalize(i.lightDir);
					fixed3 tangentViewDir = normalize(i.viewDir);  

					fixed4 packedNormal = tex2D(_BumpTex,i.uv.zw);  //对法线纹理进行采样
					fixed3 tangentNormal;
				    //若法线纹理类型没有被设置为bump类型，则进行手动反映射
					//tangentNormal=(packedNormal.xyz*2-1);
					//若已经设置为bump类型，可以使用内建函数
					tangentNormal = UnpackNormal(packedNormal);
					tangentNormal.xy *= _BumpScale;
					tangentNormal.z = sqrt(1.0-saturate(dot(tangentNormal.xy,tangentNormal.xy)));

					fixed3 albedo = _Color.rgb*tex2D(_MainTex,i.uv.xy);
					fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz*albedo;
					fixed3 diffuse = _LightColor0.rgb*albedo*max(0,dot(tangentNormal,tangentLightDir));
					fixed3 halfDir = normalize(tangentLightDir+tangentViewDir);
					fixed3 specular = _LightColor0.rgb*_Specular.rgb*pow(max(0,dot(tangentNormal,halfDir)),_Gloss);

					return fixed4(ambient+diffuse+specular,1.0);
				}
			ENDCG
		}
		}
			FallBack "Specular"
	}  

实例效果：     
![](https://i.imgur.com/GpVUUi2.png)  
![](https://i.imgur.com/qw5MWv5.png)  

**世界空间下的光照模型计算**     
世界空间下的光照模型计算需要将切线空间下的法线变换到世界空间中，因此需要先知道切线到世界的变换矩阵，又由于法线是通过在片元着色器中做纹理采样得到，因此需要通过在顶点着色器中得到转换矩阵再传递到片元着色器，在进行纹理采样后再做转换并计算光照效果。  

实例代码：  

		Shader "Custom/Chapter7_WorldNormal" {
		Properties{
		_Color("Color",Color)=(1,1,1,1)
		_MainTex("MainTex",2D) = "white"{}
		_BumpTex("BumpTex",2D) = "bump"{}
		_BumpScale("BumpScale",Float) = 1.0
		_Specular("Specular",Color) = (1,1,1,1)
		_Gloss("Gloss",Range(8.0,256)) = 20
	}

		SubShader{
			Pass{
				Tags{"LightMode" = "ForwardBase"}

				CGPROGRAM
				#pragma vertex vert
				#pragma fragment frag
				#include "Lighting.cginc"  

				fixed4 _Color;
				sampler2D _MainTex;
				float4 _MainTex_ST;
				sampler2D _BumpTex;
				float4 _BumpTex_ST;
				float _BumpScale;
				fixed4 _Specular;
				float _Gloss;

				struct a2v {
					float4 vertex:POSITION;
					float3 normal:NORMAL;
					float4 tangent:TANGENT;
					float4 texcoord:TEXCOORD0;
				};  

				struct v2f {
					float4 pos:SV_POSITION;
					float4 uv:TEXCOORD0;
					//定义用于存储变换矩阵的变量，并拆分成行存储在对应行的变量中，
					//对于矢量的变换矩阵只需要3X3即可,float4的最后一个值可以用来存储世界空间下顶点的位置
					float4 T2W0:TEXCOORD1;
					float4 T2W1:TEXCOORD2;
					float4 T2W2:TEXCOORD3;
				};

				v2f vert(a2v v) {
					v2f o;
					o.pos = UnityObjectToClipPos(v.vertex);
					o.uv.xy = v.texcoord.xy*_MainTex_ST.xy + _MainTex_ST.zw;
					o.uv.zw = v.texcoord.xy*_BumpTex_ST.xy + _BumpTex_ST.zw;  

					float3 worldPos = mul(_Object2World, v.vertex).xyz;
					fixed3 worldNormal = UnityObjectToWorldNormal(v.normal);
					fixed3 worldTangent = UnityObjectToWorldDir(v.tangent.xyz);
					fixed3 worldBinormal = cross(worldNormal, worldTangent)*v.tangent.w;

					o.T2W0 = float4(worldTangent.x, worldBinormal.x, worldNormal.x,worldPos.x);
					o.T2W1 = float4(worldTangent.y, worldBinormal.y, worldNormal.y, worldPos.y);
					o.T2W2 = float4(worldTangent.z, worldBinormal.z, worldNormal.z, worldPos.z);

					return o;
				}

				fixed4 frag(v2f i) :SV_Target{
					float3 worldPos = float3(i.T2W0.w,i.T2W1.w,i.T2W2.w);
					fixed3 worldLightDir = normalize(UnityWorldSpaceLightDir(worldPos));
					fixed3 worldViewDir = normalize(UnityWorldSpaceViewDir(worldPos));

					//法线纹理采样
					
					fixed3 tangentNormal = UnpackNormal(tex2D(_BumpTex, i.uv.zw));
					tangentNormal.xy *= _BumpScale;
					tangentNormal.z = sqrt(1.0-saturate(dot(tangentNormal.xy,tangentNormal.xy)));
			
					fixed3 worldNormal =normalize( half3(dot(i.T2W0.xyz,tangentNormal),dot(i.T2W1.xyz,tangentNormal),dot(i.T2W2.xyz,tangentNormal)));

					fixed3 albedo = _Color.rgb*tex2D(_MainTex,i.uv).rgb; 
					fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz*albedo;
					fixed3 diffuse = _LightColor0.rgb*albedo*max(0,dot(worldLightDir,worldNormal));
					fixed3 halfDir = normalize(worldLightDir+worldViewDir);
					fixed3 specular = _LightColor0.rgb*_Specular.rgb*pow(max(0,dot(halfDir,worldNormal)),_Gloss);

					return fixed4(ambient+diffuse+specular,1.0);
				}
				ENDCG	
			}
		}
		FallBack "Specular"
	}  

这里需要注意的是，当定义一个float3,float4类型时，在赋值的右边一定要加上float3，float4关键字，否则可能会得到错误的效果。  

		float3 worldPos = float3(i.T2W0.w,i.T2W1.w,i.T2W2.w);
		o.T2W0 = float4(worldTangent.x, worldBinormal.x, worldNormal.x,worldPos.x);    
实例效果：  
![](https://i.imgur.com/S2IxJkN.png)    
![](https://i.imgur.com/GQ2ibaD.png)    
中间为切线空间下计算的结果，右边为世界空间下的计算的结果，表现效果上并没有区别。  

### 渐变纹理 ###
纹理的最初使用，是为了给一个模型表面上色。实际上，纹理可以用来存储表面属性，如之前的法线纹理将法线信息存储在一张纹理中。通过纹理也可以控制漫反射光照结果。   
渐变纹理控制漫反射实例代码：    
		
		Shader "Custom/Chapter7_RampTexture" {
		Properties{
			_Color("Color",Color)=(1,1,1,1)
			_RampTex("RampTex",2D) = "white"{}
			_Specular("Specular",Color) = (1,1,1,1)
			_Gloss("Gloss",Range(8.0,256)) = 20
		}
			SubShader{
				Pass{
				Tags{"LightMode" = "ForwardBase"}

				CGPROGRAM
				#pragma vertex vert
				#pragma fragment frag
				#include  "Lighting.cginc"

					fixed4 _Color;
					sampler2D _RampTex;
					float4 _RampTex_ST;
					fixed4 _Specular;
					float _Gloss;

					struct a2v {
						float4 vertex:POSITION;
						float3 normal:NORMAL;
						fixed4 texcoord : TEXCOORD0;
					}; 

					struct v2f {
						float4 pos:SV_POSITION;
						float3 worldNormal:TEXCOORD0;
						float3 worldPos:TEXCOORD1;
						float2 uv:TEXCOORD2;
					};

					v2f vert(a2v v) {
						v2f o;
						o.pos = UnityObjectToClipPos(v.vertex);
						o.worldNormal = UnityObjectToWorldNormal(v.normal);
						o.worldPos = mul(unity_ObjectToWorld,v.vertex).xyz;
						o.uv = v.texcoord.xy*_RampTex_ST.xy + _RampTex_ST.zw;

						return o;
					}

					fixed4 frag(v2f i) :SV_Target{
						fixed3 worldNormal = normalize(i.worldNormal);
						fixed3 worldLightDir = normalize(UnityWorldSpaceLightDir(i.worldPos));
						fixed3 worldViewDir = normalize(UnityWorldSpaceViewDir(i.worldPos));
				
						fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;
						fixed halfLambert = 0.5*dot(worldNormal, worldLightDir) + 0.5;
						fixed3 diffuseColor = tex2D(_RampTex, fixed2(halfLambert, halfLambert))*_Color.rgb;
						fixed3 diffuse = diffuseColor*_LightColor0.rgb;  

						fixed3 halfDir = normalize(worldLightDir+worldViewDir);
						fixed3 specular = _LightColor0.rgb*_Specular.rgb*pow(max(0,dot(halfDir,worldNormal)),_Gloss);

						return fixed4(ambient+diffuse+specular,1.0);
					}
			ENDCG
			}
		}
	FallBack "Specular"
	}

实际效果：  
![](https://i.imgur.com/PpXQqvy.png)		  
分别对应的渐变纹理：  
![](https://i.imgur.com/e8av7AN.png)   

Shader代码中值得注意的地方：
	
		fixed halfLambert = 0.5*dot(worldNormal, worldLightDir) + 0.5;
		fixed3 diffuseColor = tex2D(_RampTex, fixed2(halfLambert, halfLambert))*_Color.rgb;  



- 这里进行纹理采样的uv坐标为半兰伯特值，将法线与光照方向的点积映射到[0,1]也就是说原本光照不到的地方会取到渐变纹理中靠左下部分的颜色。  
- 由于采样时uv坐标都是相等的，因此取到的颜色应该是对应纹理坐标[0,1]内的对角线上的颜色。  
- 还有一点值得注意的是，当采用突变性的渐变纹理时（如第一张渐变纹理），漫反射的结果是阴影之间更加分明，类似于卡通效果。  

### 遮罩纹理 ###
遮罩纹理应用于很多商业游戏中，用来保护某些区域，免于某些修改。两个常见的应用：     

- 使模型某些区域的高光强烈，某些区域较弱。而不是将高光反射应用到模型的所有地方，使用遮罩纹理可以更加细腻的控制高光的光照效果。 
- 制作地形材质时需要混合多张图片，例如表现草地，石子，裸露土地的纹理。使用遮罩纹理可以控制如何混合这些纹理。  

使用者遮罩纹理的流程：通过采样得到遮罩纹理的纹素值，然后使用其中某个通道的值与某种表面属性进行相乘，当该通道值为0时，可以保护表面不受属性影响。  

实例代码：   

		Shader "Custom/Chapter7_MaskTexture" {
		Properties{
			_Color("Color",Color)=(1,1,1,1)
			_MainTex("MainTex",2D) = "white"{}
			_BumpTex("BumpTex",2D) = "bump"{}
			_BumpScale("BumpScale",Float)=1.0
			_SpecularMask("SpecularMask",2D) = "white"{}
			_SpecularMaskScale("SpecularMaskScale",Float) = 1.0
			_Specular("Specular",Color) = (1,1,1,1)
			_Gloss("Gloss",Range(8.0,256)) = 20.0
		}
			SubShader{
				Pass{
					Tags{"LightMode" = "ForwardBase"}

					CGPROGRAM
					#pragma vertex vert
					#pragma fragment frag
					#include  "Lighting.cginc"

					fixed4 _Color;
					sampler2D _MainTex;
					float4 _MainTex_ST;     //主纹理，法线纹理，遮罩纹理的纹理坐标用一个变量来存储
					sampler2D _BumpTex;
					float _BumpScale;
					sampler2D _SpecularMask;
					float _SpecularMaskScale;
					fixed4 _Specular;
					float _Gloss;

					struct a2v {
						float4 vertex:POSITION;
						float3 normal:NORMAL;
						float4 tangent:TANGENT;
						float4 texcoord:TEXCOORD0;
					};

					struct v2f {
						float4 pos:SV_POSITION;
						float2 uv:TEXCOORD0;
						float3 lightDir:TEXCOORD1;
						float3 viewDir:TEXCOORD2;

					};

					v2f vert(a2v v) {
						v2f o;
						o.pos = UnityObjectToClipPos(v.vertex);
						o.uv = v.texcoord*_MainTex_ST.xy + _MainTex_ST.zw;  

						float3 binormal = cross(normalize(v.normal),normalize(v.tangent))*v.tangent.w;
						float3x3 rotation = float3x3(v.tangent.xyz,binormal,v.normal);

						o.lightDir = mul(rotation,ObjSpaceLightDir(v.vertex)).xyz;
						o.viewDir = mul(rotation,ObjSpaceViewDir(v.vertex)).xyz;

						return o;
					}

					fixed4 frag(v2f i) :SV_Target{
						fixed3 tangentLightDir = normalize(i.lightDir);
						fixed3 tangentViewDir = normalize(i.viewDir);

						fixed3 tangentNormal = UnpackNormal(tex2D(_BumpTex,i.uv));
						tangentNormal.xy *= _BumpScale;
						tangentNormal.z = sqrt(1.0-saturate(dot(tangentNormal.xy,tangentNormal.xy)));

						fixed3 albedo = tex2D(_MainTex,i.uv)*_Color.rgb;
						fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz*albedo;
						fixed3 diffuse = _LightColor0.rgb*albedo*max(0,dot(tangentLightDir,tangentNormal));

						fixed3 halfDir = normalize(tangentLightDir+tangentViewDir);
						fixed3 specularMask = tex2D(_SpecularMask, i.uv).r*_SpecularMaskScale;//使用其中一个通道r来影响高光的光照效果
						fixed3 specular=_LightColor0.rgb*_Specular.rgb*pow(max(0,dot(halfDir,tangentNormal)),_Gloss)*specularMask;

						return fixed4(ambient+diffuse+specular,1.0);
					}
				ENDCG
			}
		}
	FallBack "Specular"
	}  
这里有两点需要注意的地方：    



- 主纹理，法线纹理和遮罩纹理的纹理坐标都来自主纹理的uv坐标，也就是说当修改材质面板的主纹理的缩放和偏移值时，法线纹理和遮罩纹理都会相应变化，而修改法线纹理和遮罩纹理的缩放偏移值是不会对计算结果产生任何影响的，事实上测试的结果也是这样。   
- 遮罩纹理的计算过程中只用到了纹理的r通道，其他3个通道其实可以用来存储更多的设置值。  

实例效果：  
![](https://i.imgur.com/QtJQCYq.png)
![](https://i.imgur.com/www5pFO.png)  

通过遮罩处理后，高光的效果不是全部反映到整个区域，而是由r通道进行选择，遮罩纹理中全黑色的地方，即r=0处是不会受到高光影响的，这也是为什么高光部分的裂缝处的凹槽更加清晰。   

### 透明效果 ###
Unity中使用两种方法实现透明效果：    

- **透明度测试(Alpha Test)**   
- **透明度混合(Alpha  Blending)**      

开启透明混合后，当一个物体被渲染到屏幕上时，每个片元除了颜色和深度值之外，还具有透明度的属性。透明度为1时，完全不透明，透明度为0时，完全透明。 

**关于渲染顺序**   
对于不透明物体的渲染由于有深度缓冲(depth buffer，z-buffer)，不考虑渲染顺序也能得到正确的渲染结果。使用深度缓冲在渲染不透明物体时，可以不用考虑其渲染顺序，在进行深度测试时会判断物体距离摄像机的远近，远的物体是不会写入到颜色缓冲中的。  
而在处理透明物体的渲染时，需要注意渲染顺序，因为在进行透明渲染时**关闭了深度写入**   

**透明度测试(Alpha Test)**     
透明度测试会根据片元的透明度进行对应操作。  
如果透明度小于某一个值，对应的片元就被舍弃，被舍弃的片元不会再进行任何处理，不会对颜色缓冲产生影响，即完全透明，没有颜色一样。   
否则，按照不透明物体进行处理，进行深度测试和深度写入等。   
透明度测试不需要关闭深度写入，得到的效果**要么完全透明**， **要么完全不透明**。     

**透明度混合(Alpha Blending)**    
透明度混合使用片元的透明度作为混合因子，与在颜色缓冲区内的颜色进行混合，得到新的颜色。透明度混合在渲染具有透明度的物体时需要**关闭深度写入**，但**没有关闭深度测试**。当使用透明度混合渲染片元时，会比较深度值与当前深度缓冲中的深度值，如果距离摄像机更远，就不会进行混合操作。因此，先渲染不透明物体，渲染时会写入深度缓冲，再进行透明物体的渲染，进行深度测试。
透明度混合是可以得到真正的半透明效果的。

**为何关闭深度写入？**      
在对透明物体进行渲染需要关闭深度写入，这是因为假如现在有一个半透明物体离摄像机更近，后面有一个不透明物体，正常的渲染结果应该是透过半透明物体能够看到后面的不透明物体，而开启深度写入后，在对后面物体进行深度测试时，半透明物体的深度值已经写入深度缓冲中，通过比较会将后面不透明的物体剔除掉，因为 深度值比深度缓冲中的要大，因此不会被渲染出来。所以在对透明物体进行渲染需要关闭深度写入，而深度写入的关闭**需要对渲染顺序做出调整，即先进行不透明物体渲染，在进行透明物体渲染，以得到正确的渲染结果**。   

**渲染引擎中的渲染排序**：     

- 先渲染所有的不透明物体，并**开启深度写入和深度测试**
- 再把半透明物体按照距离摄像机的远近进行排序，按照从后往前的顺序进行渲染半透明物体，**关闭深度写入，开启深度测试**    

**UnityShader的渲染顺序**   
Unity使用**渲染队列(render queue)**解决渲染顺序问题。通过使用SubShader的**Queue**标签决定渲染的模型属于哪一个渲染队列。Unity内部使用一系列索引号表示渲染顺序，索引号越小渲染越早。  
![](https://i.imgur.com/YvMlZVa.png)  

若是使用透明度测试实现透明效果：   

	SubShader{
		Tags{"Queue"="AlphaTest"}
		Pass{
		}
	}  
若是使用透明度混合实现透明效果：    

	SubShader{
		Tags{"Queue"="Transparent"}
		Pass{
			Zwrite Off
		}
	}
使用透明度混合是需要**关闭深度写入**的。
		
**透明度测试(Alpha Test)实例**   
使用透明度测试的方式实现透明是对一个片元的透明度进行判断，如果小于某一个阈值，则舍弃该片元，作为完全透明处理。通常在片元着色器中使用**clip函数**进行透明度测试。函数定义：  

	函数：void clip(float4 x) ; void clip(float3 x) ; void clip(float2 x) ; void clip(float x)
	参数 ：裁剪时使用的标量或矢量条件
	描述：如果给定参数的任何一个分量是负数，就舍弃当前像素的输出颜色，即
	
	void clip(float x){
		if(any(x<0))
			discard;
	} 
实例代码：  

	Shader "Custom/Chapter8_AlphaTest" {
	Properties{
		_Color("Color",Color)=(1,1,1,1)
		_MainTex("MainTex",2D)="white"{}
		_Cutoff("Alpha Cutoff",Range(0,1))=0.5  //在材质面板显示和调节透明度测试的控制阈值
	}
	SubShader{
		Tags{"Queue"="AlphaTest" "IgnoreProjector"="True" "RenderType"="TransparentCutout"}
		//通常，使用透明度测试的Shader都应该在SubShader中设置这三个标签
		Pass{
			Tags{"LightMode"="ForwardBase"}
			CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag
			#include "Lighting.cginc"

			fixed4 _Color;
			sampler2D _MainTex;
			float4 _MainTex_ST;
			fixed _Cutoff;

			struct a2v{
				float4 vertex:POSITION;
				float3 normal:NORMAL;
				float4 texcoord:TEXCOORD0;
			};

			struct v2f{
				float4 pos:SV_POSITION;
				float3 worldNormal:TEXCOORD0;
				float3 worldPos:TEXCOORD1;
				float2 uv:TEXCOORD2;
			};

			v2f vert(a2v v){
				v2f o;
				o.pos=UnityObjectToClipPos(v.vertex);
				o.worldNormal=UnityObjectToWorldNormal(v.normal);
				o.worldPos=mul(unity_ObjectToWorld,v.vertex).xyz;
				o.uv=TRANSFORM_TEX(v.texcoord,_MainTex);

				return o;
			}

			fixed4 frag(v2f i):SV_Target{
				fixed3 worldNormal=normalize(i.worldNormal);
				fixed3 worldLightDir=normalize(UnityWorldSpaceLightDir(i.worldPos));

				fixed4 texColor=tex2D(_MainTex,i.uv);

				clip(texColor.a-_Cutoff);
				//clip函数做透明度的比较后进行裁剪操作
				fixed3 albedo=texColor.rgb*_Color;
				fixed3 ambient=UNITY_LIGHTMODEL_AMBIENT.xyz*albedo;
				fixed3 diffuse=_LightColor0.rgb*albedo*max(0,dot(worldNormal,worldLightDir));

				return fixed4(ambient+diffuse,1.0);

			}
			ENDCG
		}
	}
	FallBack "Diffuse"
	}  
实例效果：  
![](https://i.imgur.com/MNRm0Pz.png)  
![](https://i.imgur.com/vDKKHli.png)   
从左往右阈值依次是0.55,0.65,0.75，漫反射贴图自带透明通道a，使用clip函数根据传入的透明度与阈值的插值判断片元舍弃还是按照不透明处理，可以看出透明度测试效果要么为全透明，要么为完全不透明。

**透明度混合**    
透明度混合可以得到真正的半透明效果，使用指定的混合因子对当前片元颜色和颜色缓冲区中的颜色进形混合，得到新的颜色。透明度混合需要关闭深度写入，因此需要注意渲染的顺序。    
混合需要使用混合命令Blend。Blend是Unity提供的设置混合模式的命令。Blend的语义：   
![](https://i.imgur.com/rcNSghf.png)  

实例代码： 

	Shader "Custom/Chapter8_AlphaBlend" {
	Properties{
		_Color("Color",Color)=(1,1,1,1)
		_MainTex("MainTex",2D)="white"{}
		_AlphaScale("AlphaScale",Range(0,1))=1
	}
	SubShader{
		Tags{"Queue"="Transparent" "IgnoreProjector"="True" "RenderType"="Transparent"}
		//"RenderType"="Transparent"指明该shader为使用了透明度混合的shader
		Pass{
			Tags{"LightMode"="ForwardBase"}

			Zwrite Off  //关闭深度写入
			Blend SrcAlpha OneMinusSrcAlpha  //设置混合因子为源的透明度 

			CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag
			#include "Lighting.cginc"

			fixed4 _Color;
			sampler2D _MainTex;
			float4 _MainTex_ST;
			fixed   _AlphaScale;

			struct a2v{
				float4 vertex:POSITION;
				float3 normal:NORMAL;
				float4 texcoord:TEXCOORD0;
			};

			struct v2f{
				float4 pos:SV_POSITION;
				float3 worldPos:TEXCOORD0;
				float3 worldNormal:TEXCOORD1;
				float2 uv:TEXCOORD2;
			};

			v2f vert(a2v v){
				v2f o;
				o.pos=mul(UNITY_MATRIX_MVP,v.vertex);
				o.worldPos=mul(unity_ObjectToWorld,v.vertex).xyz;
				o.worldNormal=UnityObjectToWorldNormal(v.normal);
				o.uv=TRANSFORM_TEX(v.texcoord,_MainTex);

				return o;
			}

			fixed4 frag(v2f i):SV_Target{
				fixed3 worldNormal=normalize(i.worldNormal);
				fixed3 worldLightDir=normalize(UnityWorldSpaceLightDir(i.worldPos));

				fixed4 texColor=tex2D(_MainTex,i.uv);
				
				fixed3 albedo=texColor.rgb*_Color.rgb;
				fixed3 ambient=UNITY_LIGHTMODEL_AMBIENT.xyz*albedo;
				fixed3 diffuse=_LightColor0.rgb*albedo*max(0,dot(worldNormal,worldLightDir));

				return fixed4(ambient+diffuse,texColor.a*_AlphaScale);
			}
			ENDCG
		}
	}
	FallBack "Transparent/VertexLit"
	}  

实例效果：  
![](https://i.imgur.com/yNDpIfA.png)    
![](https://i.imgur.com/HF3yWbv.png)    
从左至右AlphaScale值依次是 1 0.5 0.2   

**开启深度写入的半透明效果**   
当模型本身具有复杂的遮挡关系或者包含复杂非凸网格时，未开启深度写入，会有因为排序错误产生的错误透明效果。一种解决办法是**使用两个Pass**来对模型进行渲染，第一个Pass开启深度写入，不输出颜色，第二个进行正常透明度混合。  

实例代码：   
	
	Shader "Custom/Chapter8_AlphaBlendZwrite" {
	Properties{
		_Color("Color",Color)=(1,1,1,1)
		_MainTex("MainTex",2D)="white"{}
		_AlphaScale("AlphaScale",Range(0,1))=1
	}
	SubShader{
		Tags{"Queue"="Transparent" "IgnoreProjector"="True" "RenderType"="Transparent"}
		//"RenderType"="Transparent"指明该shader为使用了透明度混合的shader
		
		//开启深度写入的Pass是为了将模型的深度信息写入深度缓冲中，从而剔除模型中被自身遮挡的片元
		Pass{
			ZWrite On
			ColorMask 0
		}
		Pass{
			Tags{"LightMode"="ForwardBase"}

			Zwrite Off  //关闭深度写入
			Blend SrcAlpha OneMinusSrcAlpha  //设置混合因子为源的透明度 

			CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag
			#include "Lighting.cginc"

			fixed4 _Color;
			sampler2D _MainTex;
			float4 _MainTex_ST;
			fixed   _AlphaScale;

			struct a2v{
				float4 vertex:POSITION;
				float3 normal:NORMAL;
				float4 texcoord:TEXCOORD0;
			};

			struct v2f{
				float4 pos:SV_POSITION;
				float3 worldPos:TEXCOORD0;
				float3 worldNormal:TEXCOORD1;
				float2 uv:TEXCOORD2;
			};

			v2f vert(a2v v){
				v2f o;
				o.pos=UnityObjectToClipPos(v.vertex);
				o.worldPos=mul(unity_ObjectToWorld,v.vertex).xyz;
				o.worldNormal=UnityObjectToWorldNormal(v.normal);
				o.uv=TRANSFORM_TEX(v.texcoord,_MainTex);

				return o;
			}

			fixed4 frag(v2f i):SV_Target{
				fixed3 worldNormal=normalize(i.worldNormal);
				fixed3 worldLightDir=normalize(UnityWorldSpaceLightDir(i.worldPos));

				fixed4 texColor=tex2D(_MainTex,i.uv);
				
				fixed3 albedo=texColor.rgb*_Color.rgb;
				fixed3 ambient=UNITY_LIGHTMODEL_AMBIENT.xyz*albedo;
				fixed3 diffuse=_LightColor0.rgb*albedo*max(0,dot(worldNormal,worldLightDir));

				return fixed4(ambient+diffuse,texColor.a*_AlphaScale);
			}
			ENDCG
		}
	}
	FallBack "Transparent/VertexLit"
	}   
在第一个Pass中  
	
	Pass{
			ZWrite On
			ColorMask 0
		}   
ColorMask用于设置颜色通道的写掩码，语义：  

	ColorMask  RGB | A | 0  
设置为0时，表示该Pass不写入任何颜色通道，不会输出任何颜色。  

实例效果：    
![](https://i.imgur.com/9dVufuv.png)   
左侧为一个Pass的透明度混合效果，可以看到模型内部的透明效果显示关系出现混乱，右侧为添加深度写入的Pass后，模型内部的遮挡效果正常。  

**双面渲染的透明效果**  
前面实现的透明效果中，透明度测试和透明度混合均无法看到物体内部结构。这是由于默认情况下渲染引擎剔除了物体背面的（相对于摄像机的方向）渲染图元，通过使用**Cull**命令控制需要剔除的面或得到双面渲染效果。 **Cull**语义：  

	Cull | Back | Front | Off     
设置为Back 背对摄像机的图元不会被渲染，设置为Front朝向摄像机的图元不会被渲染，设置为Off，双面渲染。  


- **透明度测试双面渲染**   
实例代码：   
	
		Pass{
			Tags{"LightMode"="ForwardBase"}

			Cull Off   //开启双面渲染效果     
			}   
透明度测试的双面渲染只需在原来基础上添加**Cull Off**的命令  
实例效果：     
![](https://i.imgur.com/NpIyU18.png)    
左侧为双面渲染效果，可以看到内部结构，右边为默认剔除背面渲染图元效果。  

- **透明度混合双面渲染**    
由于透明度混合是关闭了深度写入的，因此如果直接关闭剔除，并不能保证一个物体的正面和背面按照正确定的渲染顺序进行渲染，可能得到错误效果。因此实现双面透明度混合可以通过两个Pass，先剔除正面，渲染背面，再剔除背面渲染正面。  
实例代码：  

		Shader "Custom/Chapter8_AlphaBlendBothSide" {
		Properties{
			_Color("Color",Color)=(1,1,1,1)
			_MainTex("MainTex",2D)="white"{}
			_AlphaScale("AlphaScale",Range(0,1))=1
		}
		SubShader{
			Tags{"Queue"="Transparent" "IgnoreProjector"="True" "RenderType"="Transparent"}
		//"RenderType"="Transparent"指明该shader为使用了透明度混合的shader
		Pass{
			Tags{"LightMode"="ForwardBase"}

			Cull Front  //先渲染背面

			Zwrite Off  //关闭深度写入
			Blend SrcAlpha OneMinusSrcAlpha  //设置混合因子为源的透明度 

			CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag
			#include "Lighting.cginc"

			fixed4 _Color;
			sampler2D _MainTex;
			float4 _MainTex_ST;
			fixed   _AlphaScale;

			struct a2v{
				float4 vertex:POSITION;
				float3 normal:NORMAL;
				float4 texcoord:TEXCOORD0;
			};

			struct v2f{
				float4 pos:SV_POSITION;
				float3 worldPos:TEXCOORD0;
				float3 worldNormal:TEXCOORD1;
				float2 uv:TEXCOORD2;
			};

			v2f vert(a2v v){
				v2f o;
				o.pos=UnityObjectToClipPos(v.vertex);
				o.worldPos=mul(unity_ObjectToWorld,v.vertex).xyz;
				o.worldNormal=UnityObjectToWorldNormal(v.normal);
				o.uv=TRANSFORM_TEX(v.texcoord,_MainTex);

				return o;
			}

			fixed4 frag(v2f i):SV_Target{
				fixed3 worldNormal=normalize(i.worldNormal);
				fixed3 worldLightDir=normalize(UnityWorldSpaceLightDir(i.worldPos));

				fixed4 texColor=tex2D(_MainTex,i.uv);
				
				fixed3 albedo=texColor.rgb*_Color.rgb;
				fixed3 ambient=UNITY_LIGHTMODEL_AMBIENT.xyz*albedo;
				fixed3 diffuse=_LightColor0.rgb*albedo*max(0,dot(worldNormal,worldLightDir));

				return fixed4(ambient+diffuse,texColor.a*_AlphaScale);
			}
			ENDCG
		}
		Pass{
			Tags{"LightMode"="ForwardBase"}

			Cull Back //再渲染正面

			Zwrite Off  //关闭深度写入
			Blend SrcAlpha OneMinusSrcAlpha  //设置混合因子为源的透明度 

			CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag
			#include "Lighting.cginc"

			fixed4 _Color;
			sampler2D _MainTex;
			float4 _MainTex_ST;
			fixed   _AlphaScale;

			struct a2v{
				float4 vertex:POSITION;
				float3 normal:NORMAL;
				float4 texcoord:TEXCOORD0;
			};

			struct v2f{
				float4 pos:SV_POSITION;
				float3 worldPos:TEXCOORD0;
				float3 worldNormal:TEXCOORD1;
				float2 uv:TEXCOORD2;
			};

			v2f vert(a2v v){
				v2f o;
				o.pos=UnityObjectToClipPos(v.vertex);
				o.worldPos=mul(unity_ObjectToWorld,v.vertex).xyz;
				o.worldNormal=UnityObjectToWorldNormal(v.normal);
				o.uv=TRANSFORM_TEX(v.texcoord,_MainTex);

				return o;
			}

			fixed4 frag(v2f i):SV_Target{
				fixed3 worldNormal=normalize(i.worldNormal);
				fixed3 worldLightDir=normalize(UnityWorldSpaceLightDir(i.worldPos));

				fixed4 texColor=tex2D(_MainTex,i.uv);
				
				fixed3 albedo=texColor.rgb*_Color.rgb;
				fixed3 ambient=UNITY_LIGHTMODEL_AMBIENT.xyz*albedo;
				fixed3 diffuse=_LightColor0.rgb*albedo*max(0,dot(worldNormal,worldLightDir));

				return fixed4(ambient+diffuse,texColor.a*_AlphaScale);
			}
			ENDCG
		}
		}
		FallBack "Transparent/VertexLit"
		}   
透明度混合若想看到物体内部结构，需要两个Pass，先对背面完成渲染，在对正面进行渲染。  
实例效果：  
![](https://i.imgur.com/JWY0ORA.png)       

**应用正面剔除的描边效果**   
使用两个Pass可以实现模型描边效果，一个Pass正常渲染图元，另一个Pass中剔除正面，并在裁剪空间对定点进行移动，移动的方向为顶点对应的法线方向，这样在经过逐片元操作后背面会生成面片，由于Pass按照顺序渲染，因此只有超出模型边缘的面片会显示出来，然后指定面片颜色，即可实现类描边效果。    
实例代码：  

	Shader "Custom/GeometryEdge" {
	Properties{
		_Color("Color",Color)=(1,1,1,1)
		_EdgeColor("EdgeColor",Color)=(1,1,1,1)
		_EdgeFactor("EdgeFactor",Range(0,6))=3
		_MainTex("MainTex",2D)="white"{}
	}
	SubShader{

		Pass{
			Tags{"LightMode"="ForwardBase"}		
			CGPROGRAM	
			#pragma vertex vert
			#pragma fragment frag
			#include "Lighting.cginc"  

			fixed4 _Color;
			sampler2D _MainTex;
			float4 _MainTex_ST;

			struct a2v{
				float4 vertex:POSITION;
				float3 normal:NORMAL;
				float4 texcoord:TEXCOORD0;
			};
			struct v2f{
				float4 pos:SV_POSITION;
				float2 uv:TEXCOORD0;
				float3 worldNormal:TEXCOORD1;
				float4 worldPos:TEXCOORD2;
			};

			v2f vert(a2v v){
				v2f o;
				o.pos=UnityObjectToClipPos(v.vertex);
				o.uv=TRANSFORM_TEX(v.texcoord,_MainTex);
				o.worldPos=mul(unity_ObjectToWorld,v.vertex);
				o.worldNormal=UnityObjectToWorldNormal(v.normal);

				return o;
			}

			fixed4 frag(v2f i):SV_Target{
				fixed3 worldNormal=normalize(i.worldNormal);
				fixed3 worldLightDir=normalize(UnityWorldSpaceLightDir(i.pos));

				fixed3 albedo=tex2D(_MainTex,i.uv).rgb*_Color.rgb;
				fixed3 ambient=UNITY_LIGHTMODEL_AMBIENT.xyz*albedo;
				fixed3 diffuse=(_LightColor0.rgb)*albedo*max(0,dot(worldNormal,worldLightDir));

				return fixed4(ambient+diffuse,1.0);
			}
			ENDCG
		}

		Pass{
			Tags{"LightMode"="ForwardBase"}		
			Cull Front   //剔除正面

			CGPROGRAM	
			#pragma vertex vert
			#pragma fragment frag
			#include "UnityCG.cginc"  

			fixed4 _EdgeColor;
			float   _EdgeFactor;

			float4 vert(appdata_base v):SV_POSITION{
		
				float4 pos=UnityObjectToClipPos(v.vertex);
				//将法线转换到裁剪空间
				float3 clipNormal=mul((float3x3)UNITY_MATRIX_MVP,v.normal);
				//裁剪空间定点朝法线方向进行移动
				pos.xy+=_EdgeFactor*clipNormal.xy;

				return pos;
			}

			fixed4 frag():SV_Target{	
				return _EdgeColor;
			}
			ENDCG
		}
	}
	FallBack "Diffuse"
	}

实例效果：       
![](https://i.imgur.com/dQlUjDT.png)         
![](https://i.imgur.com/2pjCvqs.png)      
这里值得注意的是，描边效果实际上在第二个Pass过后生成的是面片而非线条，这一点在模型表面顶点法线变化剧烈的地方可以看出来，例如CUBE的顶点处。 

**渲染路径&复杂光照**       
渲染路径(Rendering Path)决定光照如何应用到UnityShader中。为了获取场景中的光源数据，需要为每个Pass指定其渲染路径。5.0以后的版本中，Unity支持**前向渲染(Forward Renddering Path)**和 **延迟渲染(Deferred Rendering Path)**。系统默认情况下选择的是**前向渲染路径**，可以更改，同时若想使用多个渲染路径，可以在不同的摄像机中调节 **Rendering Path**选项。     
通过在每个Pass中使用标签来指定Pass使用的渲染路径，即使用 **LightMode**标签实现。不同类型的渲染路径可能包含多种标签设置。  

	Pass{
		Tags{"LightMode"="ForwardBase"}
	}
前向渲染路径除了有**ForwardBase**，还有 **ForwardAdd**。Pass中的 **LightMode**支持的渲染路径设置选项：      
![](https://i.imgur.com/xXrJjDW.png)   
当我们为一个Pass设置了渲染路径的标签，就可以通过Unity提供的内置光照变量来访问这些属性。  

**前向渲染路径**   
每进行一次完整的前向渲染，都需要渲染该对象的渲染图元，并计算颜色和深度缓冲区的值。**对于每一个逐像素光源，都需要进行一次该过程。**也就是说，如果一个物体在多个逐像素光源的影响区域内，那么就需要执行对应数量的Pass，每个Pass对应一个逐像素的光源，然后在帧缓冲内将这些 **光照结果**混合起来得到最终的颜色值。      

**Unity中的前向渲染**  
Unity中，前向渲染路径有3种处理光照的方式：**逐顶点处理，逐像素处理，球谐函数** 光源类型和渲染重要度决定了使用哪一种处理方式。    
光源类型指该光源是平行光还是其他类型的光源。    
渲染重要度是指该光源是否为**重要的(IMPORTANT)**，如果将一个光源设置为重要的，那么Unity将其作为逐像素光源来处理。     
在前向渲染中，Unity根据场景中的光源对物体的影响程度对光源进行重要度排序。   
一定数目光源按照 **逐像素**处理    
最多 **4个**光源按照 **逐顶点**方式处理    
剩下的光源按照 **球谐函数**方式处理      
Unity使用的判定规则：      

- 最亮的平行光总是按照逐像素处理     
- 渲染模式设置为 **Not Important**的光源，按照逐顶点或球谐函数的方式处理
- 渲染模式被设置成**Important**的光源，按逐像素处理    
- 根据以上规则得到的逐像素光源数量小于**QualitySetting**中的逐像素光源数量，将有更多的光源以逐像素方式进行渲染       

光照计算在Pass中，前向渲染有两种：**ForwardBase**和 **ForwardAdd**，使用如下：    
![](https://i.imgur.com/gCTkHpW.png)       
 

- 在对应的Pass中使用 #pragma multi-compile-fwdbase和 #pragma multi-compile-fwdadd编译命令才能正确的访问光照衰减等光照变量。   
- Base Pass中渲染的平行光默认支持阴影，Additional Pass中渲染的光源默认情况没有阴影效果，需要使用 #pragma multi-compile-fwdadd-fullshadows代替 #pragma multi-compile-fwdadd编译指令。   
- 环境光和自发光只需计算一次，因此放到BasePass中    
- Additional Pass的渲染设置中，开启和设置设置混合模式，因为希望每个Additional Pass可以与上一次的光照结果在帧缓存中进行叠加。如果没有设置，那么就会将上一次的计算结果进行覆盖。通常设置的混合模式为**Blend One One**       
- 针对前向渲染，一个Shader通常会定义一个Base Pass（Base Pass也可以定义多个，像之前的双面渲染）和一个Additional Pass。一个Base Pass仅执行一次，而Additional Pass会根据影响该物体的其他逐像素光源的数目被多次调用。  

**内置的光照变量&函数**      
前向渲染中可以使用的内置光照变量       
![](https://i.imgur.com/wi50EzO.png)      
前向渲染中可以使用的内置光照函数           
![](https://i.imgur.com/BLt5F0f.png)           

**延迟渲染路径**        
Pass中的渲染路径的设置是根据场景中的光源数据来进行选择。当场景中包含大量实时光源时，前向渲染的性能会下降，因为需要多次Pass来计算不同光源对该物体的光照结果，在颜色缓冲区将结果混合起来得到最终光照效果。实际上每执行一个Pass都需要重新再渲染一遍物体，很多计算实际上是重复的。**延迟渲染**除了使用颜色和深度缓冲区，还会使用额外的缓冲区，称为G-缓冲，存储表面（通常离摄像机最近的表面）的法线，位置，用于光照计算的材质属性等。   
**延迟渲染原理**     
延迟渲染主要包含两个Pass。第一个Pass中不做任何光照计算，只计算片元的可见性，通过深度缓冲来实现，当片元可见，将该片元相关信息存储到G缓冲中。第二个Pass中，利用G缓冲的各个片元信息，进行光照计算。延迟渲染使用的数目通常就2个，和场景中的光源数量并没有关系。  
**Unity中的延迟渲染**     
延迟渲染路径适合场景中**光源数量较多、使用前向渲染路径会造成性能瓶颈**的情况下使用，延迟渲染中的每个光源都可以按逐像素的方式进行处理。不过，延迟渲染也存在**缺点**：     

- 不支持真正的抗锯齿(anti-aliasing)功能    
- 不能处理半透明物体（延迟渲染需要深度写入）    
- 对显卡有一定要求     

Unity中使用延迟渲染路径，需要提供两个Pass。    
第一个Pass用于渲染G缓冲，在该Pass中将物体漫反射、高光反射、颜色、平滑度、法线、自发光和深度等信息渲染到**屏幕空间**的G缓冲区，因此，延迟渲染路径的效率不依赖与场景是否复杂，而是屏幕的分辨率高低。对于每个物体来说，**这个Pass仅会执行一次**。     
第二个Pass用于计算真正的光照模型。这个Pass使用上一个Pass中渲染的数据来计算最终的光照颜色，再存储到帧缓冲中。默认的G缓冲区包括以下渲染纹理(Render Texture):    

- RT0：ARGB，RGB通道存储漫反射颜色，A通道未使用    
- RT1：ARGB，RGB通道存储高光反射颜色，A通道存储高光反射的指数部分（Gloss）    
- RT2：ARGB，RGB通道存储法线，A通道未使用     
- RT3：ARGB，存储自发光+lightmap+反射探针     
- 深度缓冲和模板缓冲   

延迟渲染路径中可以使用的内置变量   
![](https://i.imgur.com/fkq4lFr.png)       

**Unity光源类型**     
Unity支持4种光源类型：平行光、点光源、聚光灯和面光源。面光源只有在烘焙后才能起作用，不属于实时光源。Shader中常用的光源属性包括：光源的位置、方向、颜色、强度和衰减。   

- **平行光**    
平行光对照亮的范围没有限制，平行光的几何属性只有方向。平行光没有衰减的概念。      
- **点光源**     
点光源的照亮空间是有限的，由空间中的一个球体定义。点光源可以表示由一个点发出、向所有方向延伸的光。点光源具有位置、方向、颜色、强度和衰减的属性。     
- **聚光灯**    
聚光灯的照亮空间是有限的，由空间中的一个锥形区域定义。聚光灯同样具有位置、方向、颜色、强度和衰减的属性。聚光灯的衰减比点光源的衰减计算更为复杂，因为要判断是否在一个锥形区域内。    

**前向渲染中处理不同的光源类型**     
使用一个平行光和一个点光源共同照亮物体    
实例代码：  

	Shader "Custom/Chapter9_ForwardRendering" {
	Properties{
		_Color("Color",Color)=(1,1,1,1)
		_SpecularColor("SpecularColor",Color)=(1,1,1,1)
		_Gloss("Gloss",Range(8.0,256))=20
	}
	SubShader{
		Pass{
			Tags{"LightMode"="ForwardBase"}
			CGPROGRAM
				#pragma multi_compile_fwdbase
				//#pragma multi_compile_fwdbase指令保证Shader中使用光照衰减变量可以被正确赋值
				#pragma vertex vert
				#pragma fragment frag 
				
				#include "Lighting.cginc"
				
				fixed4 _Color;
				fixed4 _SpecularColor;
				float   _Gloss;

				struct a2v{
					float4 vertex:POSITION;
					float3 normal:NORMAL;
				};
				struct v2f{
					float4 pos:SV_POSITION;
					float3 worldPos:TEXCOORD0;
					float3 worldNormal:TEXCOORD1;
				};

				v2f vert(a2v v){
					v2f o;
					o.pos=UnityObjectToClipPos(v.vertex);
					o.worldPos=mul(unity_ObjectToWorld,v.vertex).xyz;
					o.worldNormal=UnityObjectToWorldNormal(v.normal);

					return o;
				}

				fixed4 frag(v2f i):SV_Target{
					fixed3 worldLightDir=normalize(UnityWorldSpaceLightDir(i.worldPos));
					fixed3 worldNormal=normalize(i.worldNormal);
					fixed3 worldViewDir=normalize(UnityWorldSpaceViewDir(i.worldPos));

					fixed3 ambient=UNITY_LIGHTMODEL_AMBIENT.xyz;
					fixed3 diffuse=_LightColor0.rgb*_Color.rgb*max(0,dot(worldNormal,worldLightDir));
					fixed3 halfDir=normalize(worldViewDir+worldLightDir);
					fixed3 specularColor=_LightColor0.rgb*_SpecularColor.rgb*pow(saturate(dot(worldNormal,halfDir)),_Gloss);
					
					fixed atten=1.0;
					//Unity选择最亮的平行光在BasePass中处理，平行光的衰减值设为1，认为没有衰减
					return fixed4(ambient+(diffuse+specularColor)*atten,1.0);
				}
			ENDCG
		}

		Pass{
			Tags{"LightMode"="ForwardAdd"}
			Blend One One
			CGPROGRAM
				#pragma multi_compile_fwdadd
				//#pragma multi_compile_fwdadd指令保证Shader中访问正确的光照变量
				#pragma vertex vert
				#pragma fragment frag 
				
				#include "Lighting.cginc"
				//需要引入改文件，才能正确的访问到_LightMatrix0光照矩阵，对坐标进行转换
				//以便对光照衰减纹理进行采样
				#include "AutoLight.cginc"
				
				fixed4 _Color;
				fixed4 _SpecularColor;
				float   _Gloss;

				struct a2v{
					float4 vertex:POSITION;
					float3 normal:NORMAL;
				};
				struct v2f{
					float4 pos:SV_POSITION;
					float3 worldPos:TEXCOORD0;
					float3 worldNormal:TEXCOORD1;
				};

				v2f vert(a2v v){
					v2f o;
					o.pos=UnityObjectToClipPos(v.vertex);
					o.worldPos=mul(unity_ObjectToWorld,v.vertex).xyz;
					o.worldNormal=UnityObjectToWorldNormal(v.normal);

					return o;
				}

				fixed4 frag(v2f i):SV_Target{
					//Additional中去掉BasePass中的环境光、自发光、逐顶点光照和SH光照部分
					//AddPass中处理的光源可能是非最亮平行光、点光源、聚光灯，因此计算光源的
					//位置、方向、颜色、强度和衰减时，需要根据光源类型分别进行计算
					#ifdef USING_DIRECTIONAL_LIGHT
						fixed3 worldLightDir=normalize(_WorldSpaceLightPos0.xyz);
					#else
						fixed3 worldLightDir=normalize(_WorldSpaceLightPos0.xyz-i.worldPos.xyz);
					#endif
					//上述过程先判断处理的光源是否为平行光，如果是平行光，渲染引擎会定义USING_DIRECTIONAL_LIGHT
					//若没有定义则说明不是平行光，光源位置通过运算得到
					fixed3 worldNormal=normalize(i.worldNormal);
					fixed3 worldViewDir=normalize(UnityWorldSpaceViewDir(i.worldPos));
					
					fixed3 diffuse=_LightColor0.rgb*_Color.rgb*max(0,dot(worldNormal,worldLightDir));
					fixed3 halfDir=normalize(worldViewDir+worldLightDir);
					fixed3 specularColor=_LightColor0.rgb*_SpecularColor.rgb*pow(saturate(dot(worldNormal,halfDir)),_Gloss);
					
					//处理衰减过程
					//针对其他光源类型，Unity使用一张纹理作为查找表，来得到片元着色器中的光照衰减值
					//先得到光源空间下的坐标，使用该坐标进行采样得到衰减值
					#ifdef USING_DIRECTIONAL_LIGHT
						fixed  atten=1.0;
					#else 
						float3 lightCoord=mul(_LightMatrix0,float4(i.worldPos,1)).xyz;
						fixed atten = tex2D(_LightTexture0, dot(lightCoord, lightCoord).rr).UNITY_ATTEN_CHANNEL;
					#endif		
					return fixed4((diffuse+specularColor)*atten,1.0);
				}
			ENDCG
		}
	}
	FallBack "Specular"
	}  
实例效果：    
![](https://i.imgur.com/8SycLR1.png)     
左侧没有AddPass的BlinnPong光照效果，右侧为增加上述AddPass的BlinnPhong光照效果，可以看出右侧可以接收场景中的点光源的影响。 

**Unity中的光照衰减**    
Unity在内部使用一张名为_LightTexture0的纹理来计算光源衰减，好处在于计算衰减不依赖于复杂的数学公式。

	fixed atten = tex2D(_LightTexture0, dot(lightCoord, lightCoord).rr).UNITY_ATTEN_CHANNEL;   

通常只会关心_LightTexture0对角线上的纹理颜色值，这些值表明了在光源空间中不同位置点的衰减值。(0,0)点表明了与光源位置重合的点的衰减值，(1,1)表明了在光源空间中所关心的距离最远点的衰减。   
在对光照纹理进行采样得到光照衰减值之前，先得到光照空间的坐标信息，是通过将世界空间中的坐标信息转换到光照空间实现的    

	float3 lightCoord=mul(_LightMatrix0,float4(i.worldPos,1)).xyz;  

**Unity中的阴影**     
Unity中可以使一个物体向其他物体投射阴影，以及让一个物体接收其他物体的阴影，从而使场景看起来更加真实。Unity的实时渲染中，使用一种**Shadow Map**技术，这种技术将摄像机放在光源位置，场景中的阴影区域即为摄像机看不到的区域。阴影映射纹理本质上是一张深度图，记录从光源位置出发，能看到的场景中距离距离它最近的表面位置的深度信息。Unity使用一个额外的Pass专门用于更新光源的阴影映射纹理，该Pass为 **LightMode**标签设置为 **ShadowCaster**的Pass。当使用屏幕空间阴影映射时，Unity先调用  **LightMode**为 **ShadowCaster**的Pass来得到可投射阴影的光源的阴影映射纹理以及摄像机的深度纹理。根据光源的阴影映射纹理和深度纹理得到屏幕空间的阴影图，一个物体若想接收其他物体的阴影，则需要在Shader中对阴影进行采样。阴影图是屏幕空间下的，先对表面坐标从模型空间变换到屏幕空间，再对阴影图进行采样。完整的过程为：    
![](https://i.imgur.com/RjORQ02.png)     

**不透明物体的阴影**     
Unity中通过设置物体**MeshRenderer**中的 **Cast Shadows**和  **Receive Shadows**可以使物体投射或接收阴影。   

- **让物体投射阴影**      
设置Mesh Renderer的Cast Shadows为On，Unity会将该物体加入到光源的阴影映射纹理的计算中，使其他物体在对阴影映射纹理采样时得到该物体的信息。例如该场景中：      
![](https://i.imgur.com/j5FLhcG.png)       
设置Cube的Cast Shadows为On，勾选Plane的Receive Shadows      
![](https://i.imgur.com/23rAAUD.png)      
Cube所应用的材质挂载的Shader为上述"Custom/Chapter9_ForwardRendering"，该Shader中并没有一个专门的Pass为 “LightMode”="ShadowCaster"来实现阴影的处理，而能投射的阴影的原因在于 **FallBack:"Specular"**语义，Specular本身也没有这样的Pass，而Specular的 **Fallback:"VertexLit"**包含对应的Pass。在开启Cast Shadows后Unity会在Shader和回调的Shader中一直寻找对应Pass并处理阴影映射纹理计算。  
物体的Mesh Renderer中的Cast Shadows还可以设置为 **Two Sided**,允许对物体所有的面都加入到阴影映射纹理的计算中。   

- **让物体接收阴影**    
物体接受阴影，需要对阴影纹理进行采样和相应的计算，需要用到     
		
		#include  "AutoLight.cginc"
		SHADOW_COORDS()
		TRTANSFER_SHADOW
		SHADOW_ATTENUATION()    
对应的包含文件和相应的宏指令，完整代码：   

		Shader "Custom/Chapter9_Shadow" {
		Properties{
		_Color("Color",Color)=(1,1,1,1)
		_SpecularColor("Specular",Color)=(1,1,1,1)
		_Gloss("Gloss",Range(8.0,256))=20
		}

		SubShader{
		Pass{
			Tags{"LightMode"="ForwardBase"}

			CGPROGRAM
			#pragma multi_compile_fwdbase
			#pragma vertex vert
			#pragma fragment frag

			#include "Lighting.cginc"
			#include "AutoLight.cginc" 
			//计算阴影所用的宏包含在AutoLight.cginc文件中

			fixed4 _Color;
			fixed4 _SpecularColor;
			float   _Gloss;

			struct a2v{
				float4  vertex:POSITION;
				float3  normal:NORMAL; 
			};

			struct v2f{
				float4 pos:SV_POSITION;
				float3 worldPos:TEXCOORD0;
				float3 worldNormal:TEXCOORD1;
				SHADOW_COORDS(2) 
				//该宏的作用是声明一个用于对阴影纹理采样的坐标
				//这个宏的参数是下一个可用的插值寄存器的索引值，上述中为2
			};

			v2f vert(a2v v){
				v2f o;
				o.pos=UnityObjectToClipPos(v.vertex);
				o.worldPos=mul(unity_ObjectToWorld,v.vertex).xyz;
				o.worldNormal=UnityObjectToWorldNormal(v.normal);

				TRANSFER_SHADOW(o);
				//该宏用于计算上一步声明的阴影纹理采样坐标

				return o;
			}

			fixed4 frag(v2f i):SV_Target{
				float3 worldLightDir=normalize(UnityWorldSpaceLightDir(i.worldPos));
				float3 worldNormal=normalize(i.worldNormal);
				float3 worldViewDir=normalize(UnityWorldSpaceViewDir(i.worldPos));

				fixed3 ambient=UNITY_LIGHTMODEL_AMBIENT.xyz;
				fixed3 diffuse=_LightColor0.rgb*_Color.rgb*max(0,dot(worldLightDir,worldNormal));
				fixed3 halfDir=normalize(worldLightDir+worldViewDir);
				fixed3 specularColor=_LightColor0.rgb*_SpecularColor.rgb*pow(saturate(dot(halfDir,worldNormal)),_Gloss);

				fixed shadow=SHADOW_ATTENUATION(i);
				//片元着色器中计算阴影值

				fixed atten=1.0;
				return fixed4(ambient+(diffuse+specularColor)*atten*shadow,1.0);
			}
			ENDCG
		}

		Pass{
			Tags{"LightMode"="ForwardAdd"}
			Blend One One
			CGPROGRAM
			#pragma multi_compile_fwdadd
			#pragma vertex vert
			#pragma fragment frag

			#include "Lighting.cginc"
			#include "AutoLight.cginc"

			fixed4 _Color;
			fixed4 _SpecularColor;
			float   _Gloss;

			struct a2v{
				float4  vertex:POSITION;
				float3  normal:NORMAL; 
			};

			struct v2f{
				float4 pos:SV_POSITION;
				float3 worldPos:TEXCOORD0;
				float3 worldNormal:TEXCOORD1;
			};

			v2f vert(a2v v){
				v2f o;
				o.pos=UnityObjectToClipPos(v.vertex);
				o.worldPos=mul(unity_ObjectToWorld,v.vertex).xyz;
				o.worldNormal=UnityObjectToWorldNormal(v.normal);

				return o;
			}

			fixed4 frag(v2f i):SV_Target{
				#ifdef USING_DIRECTIONAL_LIGHT
					fixed3 worldLightDir=normalize(_WorldSpaceLightPos0.xyz);
					fixed atten=1.0;
				#else
					fixed3 worldLightDir=normalize(_WorldSpaceLightPos0.xyz-i.worldPos.xyz);
					float3 lightCoord=mul(unity_WorldToLight,float4(i.worldPos,1.0)).xyz;
					fixed atten=tex2D(_LightTexture0,dot(lightCoord,lightCoord).rr).UNITY_ATTEN_CHANNEL;
				#endif
				float3 worldNormal=normalize(i.worldNormal);
				float3 worldViewDir=normalize(UnityWorldSpaceViewDir(i.worldPos));

				fixed3 diffuse=_LightColor0.rgb*_Color.rgb*max(0,dot(worldLightDir,worldNormal));
				fixed3 halfDir=normalize(worldLightDir+worldViewDir);
				fixed3 specularColor=_LightColor0.rgb*_SpecularColor.rgb*pow(saturate(dot(halfDir,worldNormal)),_Gloss);

				
				return fixed4((diffuse+specularColor)*atten,1.0);
			}
			ENDCG
		}
		}
		FallBack "Specular"
		}  
第二个Pass是为了计算场景中其他光源对物体的影响。阴影的计算主要在第一个Pass中，也就是"LightMode"="ForwardBase" 。
在前向渲染中，**SHADOW_COORDS** 声明一个阴影纹理坐标， **TRANSFER_SHADOW** 根据平台不同使用屏幕空间阴影纹理映射技术，将顶点坐标从模型空间变换到光源空间后存储到之前声明的阴影纹理坐标中， **SHADOW_ATTENUATION** 负责使用阴影映射纹理坐标对相关纹理进行采样，得到阴影信息。   
**这里需要注意的是** 这些宏会使用上下文变量来进行相关计算，为了确保宏正确工作，要保证自定义的变量名与宏使用的变量名相匹配，a2f结构体中顶点坐标变量名必须是 **vertex** ,顶点着色器的输入结构体a2v 必须命名为 **v** v2f中的顶点位置变量名必须是 **pos**。   
实例效果：  
![](https://i.imgur.com/bY2Nt1g.png)      
![](https://i.imgur.com/rM0DCcZ.png)   
第一个没有添加阴影处理效果，后一个添加阴影处理效果，可以看到后一个接受到了侧面的阴影       
Unity中使用 **UNITY_LIGHT_ATTENUATION**同时计算阴影和衰减    

		UNITY_LIGHT_ATTENUATION(atten,i,i.worldPos);
		//通过UNITY_LIGHT_ATTENUATION计算衰减和阴影
		return fixed4(ambient+(diffuse+specularColor)*atten,1.0);  
得到的效果和之前相同， **UNITY_LIGHT_ATTENUATION**接收三个参数，将光照衰减和阴影值相乘后的结果存储到第一个参数中，atten不需要在代码中提前声明， **UNITY_LIGHT_ATTENUATION**宏会声明这个变量，第二个参数是结构体v2f，该参数传递阴影坐标，计算阴影值，第三个参数是世界空间坐标，用于计算光源下的坐标。    
Addtional Pass中添加阴影，可以使用      

		#pragma multi_compile_fwdadd_fullshadows   
编译命令代替     

		#pragma multi_compile_fwdadd  
Unity会为额外的逐像素光源计算阴影。  

**透明度物体的阴影**     
Unity中物体想要投射阴影，必须在其Shader中提供**ShadowCaster**的Pass。对于大部分不透明的物体来说， **FallBack**设置为 **VertexLit**就可以得到正确的阴影。对于透明物体来说，其透明效果的实现使用透明度测试和透明度混合得到，需要小心设置这些物体的FallBack。  

- **透明度测试的处理**    
透明度测试的Shader使用之前的Shader，只过不过需要添加    

		#include "Lighting.cginc"
		#include "AutoLight.cginc"  
		SHADOW_COORDS()
		TRANSFER_SHADOW;
		UNITY_LIGHT_ATTENUATION();   
对应的包含文件和阴影相关的宏指令，如果FallBack使用 **"VertexLit"**,得到的阴影效果：     
![](https://i.imgur.com/FBpVEaV.png)       
可以看到，透明镂空部分的阴影仍然完整，看起来CUBE和普通CUBE一样。这是由于 **VertexLit**中提供的 **ShadowCaster**Pass中并没有处理透明度测试的计算。    
Unity中提供了对应的Pass，具有透明度测试功能的 **ShadowCaster** ,若将FallBack改为 **"Transparent/Cutout/VertexLit"** ，得到效果   
![](https://i.imgur.com/4RBw8Xi.png)    
可以看到，这时镂空部分并没有投射出阴影，**需要注意的是**  这个Pass中使用到了 **_Cutoff** 的属性，因此在自定义的Pass中需要提供对应名称的属性。  
实例完整代码：   

		Shader "Custom/Chapter9_AlphaTestWithShadow" {
		Properties{
		_Color("Color",Color)=(1,1,1,1)
		_MainTex("MainTex",2D)="white"{}
		_Cutoff("Alpha Cutoff",Range(0,1))=0.5  //在材质面板显示和调节透明度测试的控制阈值
		}
		SubShader{
		Tags{"Queue"="AlphaTest" "IgnoreProjector"="True" "RenderType"="TransparentCutout"}
		//通常，使用透明度测试的Shader都应该在SubShader中设置这三个标签
		//"RenderType"="TransparentCutout"指明该shader为使用了透明度测试的shader
		Pass{
			Tags{"LightMode"="ForwardBase"}
			CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag
			#include "Lighting.cginc"

			#include "Autolight.cginc"

			fixed4 _Color;
			sampler2D _MainTex;
			float4 _MainTex_ST;
			fixed _Cutoff;

			struct a2v{
				float4 vertex:POSITION;
				float3 normal:NORMAL;
				float4 texcoord:TEXCOORD0;
			};

			struct v2f{
				float4 pos:SV_POSITION;
				float3 worldNormal:TEXCOORD0;
				float3 worldPos:TEXCOORD1;
				float2 uv:TEXCOORD2;

				SHADOW_COORDS(3)
			};

			v2f vert(a2v v){
				v2f o;
				o.pos=UnityObjectToClipPos(v.vertex);
				o.worldNormal=UnityObjectToWorldNormal(v.normal);
				o.worldPos=mul(unity_ObjectToWorld,v.vertex).xyz;
				o.uv=TRANSFORM_TEX(v.texcoord,_MainTex);

				TRANSFER_SHADOW(o);
				return o;
			}

			fixed4 frag(v2f i):SV_Target{
				fixed3 worldNormal=normalize(i.worldNormal);
				fixed3 worldLightDir=normalize(UnityWorldSpaceLightDir(i.worldPos));

				fixed4 texColor=tex2D(_MainTex,i.uv);

				clip(texColor.a-_Cutoff);
				//clip函数做透明度的比较后进行裁剪剔除操作
				fixed3 albedo=texColor.rgb*_Color;
				fixed3 ambient=UNITY_LIGHTMODEL_AMBIENT.xyz*albedo;
				fixed3 diffuse=_LightColor0.rgb*albedo*max(0,dot(worldNormal,worldLightDir));

				UNITY_LIGHT_ATTENUATION(atten,i,i.worldPos);
				return fixed4(ambient+diffuse*atten,1.0);

			}
			ENDCG
			}
		}
		//FallBack "VertexLit"
		FallBack "Transparent/Cutout/VertexLit"
		}
				 


- **透明度混合的处理**    
由于透明度混合需要关闭深度写入，对阴影纹理会产生影响，半透明物体不会参与深度图和阴影映射纹理的计算，他们不会向其他物体投射阴影，也不会接受来自其他物体的阴影。若要强制产生阴影，可以将其Fallback设置为 **VertexLit**并开启阴影投射和接收阴影选项。

### 高级纹理 ###
**立方体纹理**    
**立方体纹理（CubeMap）**是 **环境映射(Enviroment Mapping)**的一种实现方式。立方体纹理包含6张图像，每个面表示沿着世界空间下的轴向（上、下、左、右、前、后） 观察所得的图像。对立方体纹理的采样需要提供三维纹理坐标，表示在世界空间下的一个3D方向，方向矢量从立方体中心出发，向外部延伸就会和立方体的6个纹理之一发生相交，采样结果由交点计算而来。    
立方体纹理在实时渲染中最常见的应用是天空盒子以及环境映射。

天空盒子的创建通过Unity自带的Skybox/6 Sided材质创建，需要对应6张纹理，即        
![](https://i.imgur.com/JVgrjwG.png)

立方体纹理用于环境映射，模拟类似金属反射周围环境的效果，这个过程首先需要获取到指定位置的立方体纹理，再应用到反射和折射计算。  
**Step1 获取指定位置的立方体纹理**     
主要通过脚本实现，添加编辑器工具，利用Unity提供的**Camera.RenderToCubemap**方法生成立方体纹理，完整代码：  

	public class RenderCubeMapWizard : ScriptableWizard
	{

    public Transform renderFromPosition;
    public Cubemap cubemap;

    void OnWizardUpdate()
    {
        helpString = "Select transform to render from and cubemap to render into";
        isValid=(renderFromPosition!=null)&&(cubemap!=null);
    }

    void OnWizardCreate()
    {
        GameObject go=new GameObject("CubemapCamera");
        go.AddComponent<Camera>();
        go.transform.position = renderFromPosition.position;
        go.GetComponent<Camera>().RenderToCubemap(cubemap);

        DestroyImmediate(go);
    }

    [MenuItem("GameObject/Render into Cubemap")]
    static void RenderCubemap()
    {
        ScriptableWizard.DisplayWizard<RenderCubeMapWizard>("Render cubemap", "Render");
    }
    }	
需要添加 “using UnityEditor”命名空间，点击“Render”脚本执行后会将对应点的立方体纹理渲染到指定的立方体纹理中。 

**Step2 反射计算**      
反射效果通过入射光线方向和表面法线得到反射方向，再利用反射方向对立方体纹理进行采样，得到反射效果。入射光线为观察方向的反方向，主要使用CG函数中的reflect函数。完整代码：   
	
	Shader "Custom/Chapter10_Reflection" {
	Properties{
		_Color("Color",Color)=(1,1,1,1)
		_ReflectionColor("ReflectionColor",Color)=(1,1,1,1)
		_ReflectionAmount("ReflectionAmount",Range(0,1))=1
		_Cubemap("Cubemap",Cube)="_Skybox"{}
	}
	SubShader{
		Pass{
			Tags{"LightMode"="ForwardBase"}

			CGPROGRAM
				#pragma vertex vert
				#pragma fragment frag
				#pragma multi_compile_fwdbase

				#include "Lighting.cginc"
				#include "AutoLight.cginc"

				fixed4 _Color;
				fixed4 _ReflectionColor;
				float   _ReflectionAmount;
				samplerCUBE  _Cubemap;

				struct a2v{
					float4 vertex:POSITION;
					float3 normal:NORMAL;
				};

				struct v2f{
					float4 pos:SV_POSITION;
					float3 worldPos:TEXCOORD0;
					float3 worldNormal:TEXCOORD1;
					float3 worldViewDir:TEXCOORD2;
					float3 worldRefl:TEXCOORD3;
					SHADOW_COORDS(4)
				};

				v2f vert(a2v v){
					v2f o;
					o.pos=UnityObjectToClipPos(v.vertex);
					o.worldPos=mul(_Object2World,v.vertex).xyz;
					o.worldNormal=UnityObjectToWorldNormal(v.normal);
					o.worldViewDir=UnityWorldSpaceViewDir(o.worldPos);

					//将观察方向的反方向作为入射方向去计算反射方向
					o.worldRefl=reflect(-o.worldViewDir,o.worldNormal);

					TRANSFER_SHADOW	(o);
					return o;
				}
				fixed4 frag(v2f i):SV_Target{
					fixed3 worldNormal=normalize(i.worldNormal);
					fixed3 worldViewDir=normalize(i.worldViewDir);
					fixed3 worldLightDir=normalize(UnityWorldSpaceLightDir(i.worldPos));

					fixed3 ambient=UNITY_LIGHTMODEL_AMBIENT.xyz;
					fixed3 diffuse=_LightColor0.rgb*_Color.rgb*max(dot(worldNormal,worldLightDir),0);
					fixed3 reflection=texCUBE(_Cubemap,i.worldRefl).rgb*_ReflectionColor.rgb;
					
					UNITY_LIGHT_ATTENUATION(atten,i,i.worldPos);

					fixed3 color=ambient+lerp(diffuse,reflection,_ReflectionAmount)*atten;

					return fixed4(color,1.0);
				}
			ENDCG
		}
	}
	FallBack "Diffuse"
	} 

代码效果：  
	![](https://i.imgur.com/Xu5X3Ey.png)     

**Step3 折射计算**
折射计算和反射计算类似，先计算出折射方向，再向生成立方体纹理进行采样，主要用到CG函数中的refract函数，传入三个参数，入射方向，法线方向，折射率比值（入射到反射方向）**值得注意的是** 传入的方向参数必须是归一化之后的方向。完整代码为：  
	
	Shader "Custom/Chapter10_Refraction" {
	Properties{
		_Color("Color",Color)=(1,1,1,1)
		_RefractionColor("Reflection Color",Color)=(1,1,1,1)
		_RefractionAmount("Reflection Amount",Range(0,1))=1
		_RefractionRatio("Refraction Ratio",Range(0.1,1))=0.5
		_Cubemap("Cubemap",Cube)="_Skybox"{}
	}
	SubShader{
		Pass{
			Tags{"LightMode"="ForwardBase"}

			CGPROGRAM
				#pragma vertex vert
				#pragma fragment frag
				#pragma multi_compile_fwdbase

				#include "Lighting.cginc"
				#include "AutoLight.cginc"

				fixed4 _Color;
				fixed4 _RefractionColor;
				float   _RefractionAmount;
				float   _RefractionRatio;
				samplerCUBE  _Cubemap;

				struct a2v{
					float4 vertex:POSITION;
					float3 normal:NORMAL;
				};

				struct v2f{
					float4 pos:SV_POSITION;
					float3 worldPos:TEXCOORD0;
					float3 worldNormal:TEXCOORD1;
					float3 worldViewDir:TEXCOORD2;
					float3 worldRefr:TEXCOORD3;
					SHADOW_COORDS(4)
				};

				v2f vert(a2v v){
					v2f o;
					o.pos=UnityObjectToClipPos(v.vertex);
					o.worldPos=mul(unity_ObjectToWorld,v.vertex).xyz;
					o.worldNormal=UnityObjectToWorldNormal(v.normal);
					o.worldViewDir=UnityWorldSpaceViewDir(o.worldPos);

					//将观察方向的反方向作为入射方向去计算折射方向
					o.worldRefr=refract(-normalize(o.worldViewDir),normalize(o.worldNormal),_RefractionRatio);

					TRANSFER_SHADOW	(o);
					return o;
				}
				fixed4 frag(v2f i):SV_Target{
					fixed3 worldNormal=normalize(i.worldNormal);
					fixed3 worldViewDir=normalize(i.worldViewDir);
					fixed3 worldLightDir=normalize(UnityWorldSpaceLightDir(i.worldPos));

					fixed3 ambient=UNITY_LIGHTMODEL_AMBIENT.xyz;
					fixed3 diffuse=_LightColor0.rgb*_Color.rgb*max(dot(worldNormal,worldLightDir),0);
					fixed3 refraction=texCUBE(_Cubemap,i.worldRefr).rgb*_RefractionColor.rgb;
					
					UNITY_LIGHT_ATTENUATION(atten,i,i.worldPos);

					fixed3 color=ambient+lerp(diffuse,refraction,_RefractionAmount)*atten;

					return fixed4(color,1.0);
				}
			ENDCG
		}
	}
	FallBack "Diffuse"
	}   
实例效果：   
![](https://i.imgur.com/upqTXat.png)      

**菲涅尔反射**    
菲涅尔反射描述一种光学现象，当光线照射到物体表面时，一部分发生反射，一部分发生折射，一部分进入物体内部发生反射或折射。实时渲染中，经常使用**菲涅尔反射**，通过视角方向控制反射程度。  
菲涅尔反射通过菲涅尔等式计算，真实的菲尼尔反射非常复杂，实时渲染中使用近似公式计算，两个用的比较多的近似公式：  

- **Schlick菲涅尔近似等式**        
![](https://i.imgur.com/dcdSACr.png)    
F为反射系数，控制菲涅尔反射强度，v为视角方向，n为法线方向   
- **Empricial菲涅尔近似等式**     
![](https://i.imgur.com/JS93z0E.png)      
bias、scale和power是控制项     

许多车漆、水面等材质的渲染经常使用菲涅尔反射模拟更加真实的反射效果。  
使用Schlick菲涅尔近似等式模拟，完整代码：

	Shader "Custom/Chapter10_Fresnel" {
	Properties{
		_Color("Color",Color)=(1,1,1,1)
		_FresnelFactor("FresnelFactor",Range(0,1))=0.5
		_FreractionRatio("FreractionRatio",Range(0.1,1))=0.5
		_Cubemap("Cubemap",Cube)="_Skybox"{}
	}
	SubShader{
		Pass{
			Tags{"LightMode"="ForwardBase"}
			CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag
			#pragma multi_compile_fwdbase

			#include "Lighting.cginc"
			#include "AutoLight.cginc"

			fixed4 _Color;
			float   _FresnelFactor;
			float   _FreractionRatio;
			samplerCUBE _Cubemap;

			struct a2v{
				float4 vertex:POSITION;
				float3 normal:NORMAL;
			};

			struct v2f{
				float4 pos:SV_POSITION;
				float3 worldPos	:TEXCOORD0;
				float3 worldNormal:TEXCOORD1;
				float3 worldViewDir:TEXCOORD2;
				float3 worldRefl:TEXCOORD3;
				float3 worldRefr:TEXCOORD4;
				SHADOW_COORDS(5)
			};

			v2f vert(a2v v){
				v2f o;
				o.pos=UnityObjectToClipPos(v.vertex);
				o.worldPos=mul(unity_ObjectToWorld,v.vertex).xyz;
				o.worldNormal=UnityObjectToWorldNormal(v.normal);
				o.worldViewDir=UnityWorldSpaceViewDir(o.worldPos);
				o.worldRefl=reflect(-o.worldViewDir,o.worldNormal);
				o.worldRefr=refract(-normalize(o.worldViewDir),normalize(o.worldNormal),_FreractionRatio);

				TRANSFER_SHADOW(o);
				return o;
			}

			fixed4 frag(v2f i):SV_Target{
				fixed3 worldNormal=normalize(i.worldNormal);
				fixed3 worldViewDir=normalize(i.worldViewDir);
				fixed3 worldLightDir=normalize(UnityWorldSpaceLightDir(i.worldPos));

				fixed3 ambient=UNITY_LIGHTMODEL_AMBIENT.xyz;
				fixed3 diffuse=_LightColor0.rgb*_Color.rgb*max(dot(worldNormal,worldLightDir),0);
				fixed3 reflect=texCUBE(_Cubemap,i.worldRefl).rgb;

				fixed3 refract=texCUBE(_Cubemap,i.worldRefr).rgb;

				fixed fresnel=_FresnelFactor+(1-_FresnelFactor)*pow(1-dot(worldViewDir,worldNormal),5);
				
				UNITY_LIGHT_ATTENUATION(atten,i,i.worldPos);

				//fixed3 color=ambient+lerp(diffuse,reflect,saturate(fresnel))*atten;
				fixed3 color=ambient+lerp(refract,reflect,saturate(fresnel))*atten;

				return fixed4(color,1.0);
			}
			ENDCG
		}
	}
	FallBack "Diffuse"
	}
最后的颜色混合部分，可以将漫反射与反射通过菲涅尔反射系数进行插值，也可以将折射与反射通过菲涅尔反射系数进行插值混合，实例效果：      
![](https://i.imgur.com/QGaecNB.png)  

关于反射和折射部分：      

- step1: 得到环境立方体纹理  
- step2: 使用函数计算反射或折射方向
- step3: 利用 texCUBE 对立方体纹理采样 
- step4: 计算影响系数，对 漫反射与反射或折射/折射与反射 进行插值混合，得到最终颜色 

**渲染纹理**    
现代GPU允许把整个三维场景渲染到中间缓冲中，而不是帧缓冲当中，这个中间缓冲叫做**渲染目标纹理(Render Target Texture RTT)** ,与之对应的是**多重渲染目标(Mutil-Render Target) MRT** 将场景渲染到多个渲染目标纹理中。为此，Unity专门定义了一种纹理类型——渲染纹理(Render Texture)。其使用通常有两种方式：   

- 创建渲染纹理，将某个摄像机的渲染目标设置成该渲染纹理，摄像机的渲染结果就会实时渲染到该纹理中  
- 通过后期处理抓取当前屏幕图像，Unity将屏幕图像放到一张同等分辨率的渲染纹理中    

**使用渲染纹理实现镜子效果**      
通过一个额外的摄像机，调整到对应位置，设置渲染目标为一张渲染纹理，将该渲染纹理作为一张2D纹理，在采样是，将UV坐标的进行翻转即可，完整代码：   
		
	Shader "Custom/Chapter10_Mirror" {
	Properties{
		_MainTex("MainTex",2D)="white"{}
	}
	SubShader{
		
		Pass{
			Tags{"LightMode"="ForwardBase"}
			CGPROGRAM
				#pragma vertex vert
				#pragma fragment frag
				#include "Lighting.cginc"

				sampler2D _MainTex;
				float4        _MainTex_ST;

				struct a2v{
					float4 vertex:POSITION;
					float4 texcoord:TEXCOORD0;
				};
				struct v2f{
					float4 pos:SV_POSITION;
					float4 uv:TEXCOORD0;
				};

				v2f vert(a2v v){
					v2f o;
					o.pos=UnityObjectToClipPos(v.vertex);
					o.uv=v.texcoord;
					o.uv.x=1-o.uv.x;   //将uv.x分量进行翻转，实现镜子效果

					return o;
				}
				fixed4 frag(v2f i):SV_Target{
					return tex2D(_MainTex,i.uv);
				}

			ENDCG
		}
	}
	FallBack "Diffuse"
	}  

实例效果：  
![](https://i.imgur.com/zvPv802.png)     

**玻璃效果**     
Unity Shader中可以使用GrabPass完成对屏幕图像的抓取。定义GrabPass后，Unity将当前屏幕图像绘制在一张纹理中，使用GrabPass模拟玻璃透明效果，可以对物体后面的图像做更复杂的处理（使用法线模拟折射效果），而不是像使用透明度混合，只是颜色上的混合。     
在使用GrabPass进行透明效果模拟时，要注意**渲染顺序的设置** ，先保证场景中所有不透明物体已经绘制在屏幕上，再对屏幕进行抓取图像，因此一般设置成  "Queue"="Transparent"

实现玻璃效果：   

- Step1 获取指定位置的立方体纹理，通过反射方向采样得到反射颜色
- Step2 获取屏幕抓取图像，通过法线纹理得到法线方向作为影响值与影响因子相乘，调整获取的屏幕图像扭曲程度来模拟折射   
- Step3 将两者颜色进行混合，通过混合值调整反射和折射的混合程度  

完整代码：    

	Shader "Custom/Chapter10_GlassRefraction" {
	Properties{
		_MainTex("Main Tex",2D)="white"{}
		_BumpTex("Bump Tex",2D)="bump"{}
		_CubeMap("Cube Map",Cube)="_Skybox"{}
		_Distortion("Distortion",Range(0,100))=10
		_RefractAmount("Refract Amount",Range(0.0,1.0))=1.0
	}
	SubShader{
		Tags{"Queue"="Transparent" "RenderType"="Opaque"}
		//指定渲染队列为"Transparent"，确保所有不透明物体先渲染完成
		GrabPass {"_RefractionTex"}
		//声明GrabPass 该Pass会将屏幕抓取图像存储到名为"_RefractionTex"的纹理中
		Pass{
			CGPROGRAM
				#pragma vertex vert
				#pragma fragment frag 

				#include "UnityCG.cginc"

				sampler2D _MainTex;
				float4 _MainTex_ST;
				sampler2D _BumpTex;
				float4 _BumpTex_ST;
				samplerCUBE _CubeMap;
				float _Distortion;
				float _RefractAmount;
				sampler2D _RefractionTex;   //存储GrabPass抓取的屏幕图像
				float4 _RefractionTex_TexelSize;  //得到屏幕图像的纹素值，在做偏移计算时使用

				struct a2v{
					float4 vertex:POSITION;
					float3 normal:NORMAL;
					float4 tangent:TANGENT;
					float4 texcoord:TEXCOORD0;
				};

				struct v2f{
					float4 pos:SV_POSITION;
					float4 uv:TEXCOORD0;
					float4 scrPos:TEXCOORD1;
					float4 TtoW0:TEXCOORD2;
					float4 TtoW1:TEXCOORD3;
					float4 TtoW2:TEXCOORD4;
				};

				v2f vert(a2v v){
					v2f o;
					o.pos=UnityObjectToClipPos(v.vertex);
					o.scrPos=ComputeGrabScreenPos(o.pos);
					o.uv.xy=TRANSFORM_TEX(v.vertex,_MainTex);
					o.uv.zw=TRANSFORM_TEX(v.vertex,_BumpTex);

					float3 worldPos=mul(unity_ObjectToWorld,v.vertex).xyz;

					fixed3 worldNormal=UnityObjectToWorldNormal(v.normal);
					fixed3 worldTangent=UnityObjectToWorldDir(v.tangent.xyz);
					fixed3 worldBinormal=cross(worldNormal,worldTangent)*v.tangent.w;

					o.TtoW0=(worldTangent.x,worldBinormal.x,worldNormal.x,worldPos.x);
					o.TtoW1=(worldTangent.y,worldBinormal.y,worldNormal.y,worldPos.y);
					o.TtoW2=(worldTangent.z,worldBinormal.z,worldNormal.z,worldPos.z);

					return o;
				}

				fixed4 frag(v2f i):SV_Target{
					float3 worldPos=(i.TtoW0.w,i.TtoW1.w,i.TtoW2.w);
					fixed3 worldViewDir=normalize(UnityWorldSpaceViewDir(worldPos));

					fixed3 bump=UnpackNormal(tex2D(_BumpTex,i.uv.zw));

					float2 offset=bump*_Distortion*_RefractionTex_TexelSize.xy;
					i.scrPos.xy=offset+i.scrPos.xy;
					fixed3 refrColor=tex2D(_RefractionTex,i.scrPos.xy/i.scrPos.w).rgb;

					bump=normalize(half3(dot(i.TtoW0.xyz,bump),dot(i.TtoW1.xyz,bump),dot(i.TtoW2.xyz,bump)));
					fixed3 reflectDir=reflect(-worldViewDir,bump);
					fixed4 texColor=tex2D(_MainTex,i.uv.xy);
					fixed3 reflColor=texCUBE(_CubeMap,reflectDir).rgb*texColor.rgb;

					fixed3 finalColor=reflColor*(1-_RefractAmount)+refrColor*_RefractAmount;

					return fixed4(finalColor,1.0);

				}
			ENDCG
		}
	}
	FallBack "Tranparent/VertexLit"
	}  

这里需要解释一下 **ComputeGrabScreenPos**函数，与** ComputeScreenPos**类似，输入为顶点在裁剪空间下的坐标，得到屏幕坐标，但是这里需要注意的是， **此时得到的坐标并没有进行归一化，也就是还没有除w分量**，这一点可以从ComputeScreenPos的定义中看出来：      
![](https://i.imgur.com/TpiPGkO.png)      
pos是顶点在裁剪空间的坐标，_ProjectionParams.X默认情况下为1，针对不同平台可能会做翻转处理，实际上得到的结果：     
![](https://i.imgur.com/RV9eAZk.png)     
也就是说，**此时得到的坐标并不是最终屏幕图像上的坐标**，因此在片元着色器中在对抓取的屏幕图像（实际上是对这一张纹理进行取样）取样时，会有 除以w分量的操作（得到[0-1]的UV纹理坐标）
		
	tex2D(_RefractionTex,i.scrPos.xy/i.scrPos.w)

而这样做的原因是，在顶点着色器内直接除w分量会影响插值的结果，因而将该操作保留到片元着色器中进行逐像素处理。  
最终效果：      
![](https://i.imgur.com/Djeuiad.jpg)     
扭曲因子的值设为100，混合值设为1（全为折射效果）      
![](https://i.imgur.com/5PdERdE.jpg)    
扭曲因子的值设为0，混合值设为1（全为折射效果）     
可以看到这个时候，几乎看不到物体，这是由于物体表面由物体后面渲染的屏幕图像区域进行了着色，所以这种看似透明的效果是通过设置正确的渲染顺序，抓取屏幕图像实现的，而不是给物体本身设置透明材质。

**程序纹理**     
程序纹理是通过计算机计算生成的图像，使用特定的算法创建个性化图案或非常真实的自然元素。    
创建一个波点纹理  
完整代码：   
	
	[ExecuteInEditMode]
	public class ProceduralTextureGeneration : MonoBehaviour {

    public Material material = null;
    #region Material properties
    [SerializeField,SetProperty("textureWidth")]
    private int m_textureWidth = 512;
    public int textureWidth {
        get {
            return m_textureWidth;
        }
        set {
            m_textureWidth = value;
            _UpdateMaterial();
        }
    }

    [SerializeField, SetProperty("backgroundColor")]
    private Color m_backgroundColor = Color.white;
    public Color backgroundColor {
        get {
            return m_backgroundColor;
        }
        set {
            m_backgroundColor = value;
            _UpdateMaterial();
        }
    }

    [SerializeField, SetProperty("circleColor")]
    private Color m_circleColor = Color.yellow;
    public Color circleColor {
        get {
            return m_circleColor;
        }
        set {
            m_circleColor = value;
            _UpdateMaterial();
        }
    }

    [SerializeField, SetProperty("blurFactor")]
    private float m_blurFactor = 2.0f;
    public float blurFactor {
        get {
            return m_blurFactor;
        }
        set {
            m_blurFactor = value;
            _UpdateMaterial();
        }
    }
    #endregion

    private Texture2D m_generateTexture = null;


    // Use this for initialization
    void Start () {
        if (material == null)
        {
            Renderer renderers = gameObject.GetComponent<Renderer>();
            if (renderers == null)
            {
                Debug.LogWarning("Cannot find a renderer.");
                return;
            }
            material = GetComponent<Renderer>().sharedMaterial;
        }
        _UpdateMaterial();
    }
	
	// Update is called once per frame
    private void _UpdateMaterial() {
        if (material != null)
        {
            m_generateTexture = _GenerateProceduralTexture();
            material.SetTexture("_MainTex",m_generateTexture);
        }
    }

    private Texture2D _GenerateProceduralTexture()
    {
        Texture2D proceduralTexture=new Texture2D(textureWidth,textureWidth);

        //定义圆与圆之间的距离
        float circleInterval = textureWidth/4.0f;
        //定义圆的半径
        float radius = textureWidth/10.0f;
        //定义模糊系数
        float edgeBlur = 1.0f/blurFactor;

        for (int w = 0; w < textureWidth; w++)
        {
            for (int h = 0; h < textureWidth; h++)
            {
                Color pixel = backgroundColor;

                //绘制9个圆
                for (int i = 0; i < 3; i++)
                {
                    for (int j = 0; j < 3; j++)
                    {
                        //计算当前所绘制圆的位置
                        Vector2 circleCenter=new Vector2(circleInterval*(i+1),circleInterval*(j+1));
                        //计算当前像素与圆边界的距离
                        float dist = Vector2.Distance(new Vector2(w, h), circleCenter)-radius;

                        //模糊圆的边界
                        Color color = _MixColor(circleColor, new Color(pixel.r,pixel.g,pixel.b,0.0f),Mathf.SmoothStep(0f,1f,dist*edgeBlur));

                        //与之前得到的颜色混合
                        pixel = _MixColor(pixel, color, color.a);
                    }
                }
                proceduralTexture.SetPixel(w,h,pixel);
            }
        }
        proceduralTexture.Apply();
        return proceduralTexture;
    }


    private Color _MixColor(Color color0, Color color1, float mixFactor)
    {
        Color mixColor = Color.white;
        mixColor.r = Mathf.Lerp(color0.r, color1.r, mixFactor);
        mixColor.g = Mathf.Lerp(color0.g, color1.g, mixFactor);
        mixColor.b = Mathf.Lerp(color0.b, color1.b, mixFactor);
        mixColor.a = Mathf.Lerp(color0.a, color1.a, mixFactor);
        return mixColor;
    }
	}
实例效果：    
![](https://i.imgur.com/0GAm0qP.png)     

### 动画效果 ###
动画效果需要引入时间变量，Unity内置的时间变量：    
![](https://i.imgur.com/TeWEngz.png)       
这些时间变量可以用来实现纹理动画和顶点动画   

**纹理动画**     

- **序列帧动画**    
序列帧动画是通过依次播放一系列关键帧图像，当播放速度达到一定数值时，看起来就像是一个连续的动画。灵活性较强，通过一张包含关键帧的图像可以得到比较细腻的动画效果，缺点在于需要大量时间制作包含关键帧的图像。    
关键帧实现的关键在于，**每个时刻计算该时刻下应该播放的关键帧的位置**        

完整代码   

		Shader "Custom/Chapter11_ImageSequenceAnimation" {
	Properties{
		_Color("Main Clolr",Color)=(1,1,1,1)
		_MainTex("MainTex",2D)="white"{}
		_HorizantalAmount("HorizantalAmount",Float)=4
		_VerticalAmount("VerticalAmount",Float)=4
		_Speed("Speed",Range(1,100))=30
	}

	SubShader{
		Tags{"Queue"="Transparent" "IgnoreProjector"="True" "RenderType"="Transparent"}
			//关键帧动画中使用的关键帧图像一般包含透明通道，因此当成半透明来处理
		Pass{
			Tags{"LightMode"="ForwardBase"}
			ZWrite Off
			Blend  SrcAlpha OneMinusSrcAlpha

			CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag
			#include "Lighting.cginc"

			fixed4 _Color;
			sampler2D _MainTex;
			float4 _MainTex_ST;
			float _HorizantalAmount;
			float _VerticalAmount;
			float _Speed;  

			struct a2v{
				float4 vertex:POSITION;
				float4 texcoord:TEXCOORD0;
			};
			struct v2f{
				float4 pos:SV_POSITION;
				float2 uv:TEXCOORD0;
			};

			v2f vert(a2v v){
				v2f o;
				o.pos=UnityObjectToClipPos(v.vertex);
				o.uv=TRANSFORM_TEX(v.texcoord,_MainTex);

				return o;
			}

			fixed4 frag(v2f i):SV_Target{
				float time=floor(_Time.y*_Speed);   //内置时间变量 _Time(t/20,t,2t,3t)
				float row=floor(time/_HorizantalAmount);
				float column=time-row*_VerticalAmount;  
				//根据播放的速度计算对应的序列帧行和列

				//纹理中包含许多关键帧图像，将采样坐标映射到每一个关键帧的坐标范围内
				//纹理中的采样方向与关键帧的播放顺序在Y方向是反的，因此Y坐标做减法
				half2 uv=float2(i.uv.x/_HorizantalAmount,i.uv.y/_VerticalAmount);			
				uv.x+=column/_HorizantalAmount;
				uv.y-=row/_VerticalAmount;

				fixed4 c=tex2D(_MainTex,uv);
				c.rgb*=_Color;

				return c;

			}
			ENDCG
		}
	}
	FallBack "Transparent/VertexLit"
	}

- **滚动背景**    
滚动背景一般使用多个层以不同的速度进行滚动，滚动的关键在于使滚动方向上的纹理坐标的增量与时间变量相乘，这样随着时间的变化，滚动方向上的采样坐标不断变换，画面也能连续变化。   

完整代码： 
   
		Shader "Custom/Chapter11_ScrollingBackground" {
	Properties{
		_MainTex("Base Layer",2D)="white"{}
		_DetialTex("Second Layer",2D)="white"{}
		_ScrollX("Base Layer Scroll Speed",Float)=1.0
		_Scroll2X("Second Layer Scroll Speed",Float)=1.0
		_Multiplier("Layer Multiplier",Float)=1
	}
	SubShader{
		Pass{
			Tags{"LightMode"="ForwardBase"}
			CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag
			#include "Lighting.cginc"

			sampler2D _MainTex;
			float4         _MainTex_ST;
			sampler2D _DetialTex;
			float4        _DetialTex_ST;
			float			_ScrollX;
			float			_Scroll2X;
			float			_Multiplier;

			struct a2v{
				float4 vertex:POSITION;
				float4 texcoord:TEXCOORD0;
			};

			struct v2f {
				float4 pos:SV_POSITION;
				float4 uv:TEXCOORD0;
			};

			v2f vert(a2v v){
				v2f o;
				o.pos=UnityObjectToClipPos(v.vertex);
				o.uv.xy=TRANSFORM_TEX(v.texcoord,_MainTex)+frac(float2(_ScrollX,0)*_Time.y);
				o.uv.zw=TRANSFORM_TEX(v.texcoord,_DetialTex)+frac(float2(_Scroll2X,0)*_Time.y);
				//frac取小数函数，使取样坐标在[0,1]范围内，背景连续重复滚动

				return o;
			}

			fixed4 frag(v2f i):SV_Target{
				fixed4 baseLayer=tex2D(_MainTex,i.uv.xy);
				fixed4 secondLayer=tex2D(_DetialTex,i.uv.zw);

				fixed4 c=lerp(baseLayer,secondLayer,secondLayer.a);

				c.rgb*=_Multiplier;
				//_Multiplier用来控制整体亮度
				return c;
			}
			ENDCG
		}
	}
	FallBack "VertexLit"
	}   
实例效果：   
![](https://i.imgur.com/q1dHLu3.png)            
![](https://i.imgur.com/rLdoCEf.png)        

- **顶点动画**    
游戏中通常使用顶点动画模拟飘动旗帜、湍流小溪的效果，其主要方式是使模型顶点随着时间变化而变化。     

完整代码：       

		Shader "Custom/Chapter11_Water" {
	Properties{
		_Color("Main Color",Color)=(1,1,1,1)
		_MainTex("MainTex",2D)="white"{}
		_Magnitude("Magnitude",Float)=1
		_Frequency("Frequency",Float)=1
		_InvWaveLength("InvWaveLength",Float)=10
		_Speed("Speed",Float)=0.5
	}
	SubShader{
	Tags{"Queue"="Transparent" "IgnoreProjector"="True" "RenderType"="Transparent" "DisableBatching"="True"}
			//为透明效果设置对应标签，这里“DisableBatching”的标签是关闭批处理，
			//包含模型空间顶点动画的Shader是需要特殊处理的Shader，
			//而批处理会合并所有相关的模型，这些模型各自的模型空间会丢失
			//而顶点动画需要在模型空间对顶点进行偏移
		Pass{
			Tags{"LightMode"="ForwardBase"}
			ZWrite Off
			Blend SrcAlpha OneMinusDstAlpha
			Cull Off
			CGPROGRAM
				#pragma vertex vert
				#pragma fragment frag
				#include "Lighting.cginc"
				#include "UnityCG.cginc"

				fixed4 _Color;
				sampler2D _MainTex;
				float4 _MainTex_ST;
				float _Magnitude;
				float _Frequency;
				float _InvWaveLength;
				float _Speed;

				struct a2v{
					float4 vertex:POSITION;
					float4 texcoord:TEXCOORD0;
				};

				struct v2f{
					float4 pos:SV_POSITION;
					float2 uv:TEXCOORD0;
				};

				v2f vert(a2v v){
					v2f o;
					float4 offset;
					offset.yzw=float3(0.0,0.0,0.0);
					offset.x=sin(_Frequency*_Time.y+v.vertex.x*_InvWaveLength+v.vertex.y*_InvWaveLength+v.vertex.z*_InvWaveLength)*_Magnitude;
				    //在顶点进行空间变换前，对x分量进行正弦操作
					o.pos=UnityObjectToClipPos(v.vertex+offset);  

					o.uv=TRANSFORM_TEX(v.texcoord,_MainTex);
					o.uv+=float2(0.0,_Time.y*_Speed);
					//进行纹理动画

					return o;
				}

				fixed4 frag(v2f i):SV_Target{
					fixed4 c=tex2D(_MainTex,i.uv);
					c.rgb*=_Color.rgb;

					return c;
				}
			ENDCG
		}
	}
	FallBack "Transparent/VertexLit"
	}

- **广告牌技术**     
广告牌技术是另一种常见的顶点动画。广告牌会根据视角方向来旋转一个被纹理着色的多边形，使多边形看起来始终朝着摄像机。      
广告牌技术的本质是构建旋转矩阵。计算过程中先根据初始计算得到目标的表面法线（例如视角方向）和指向上的方向，这两者的方向往往不是垂直的，根据这两者的方向做叉乘得到垂直于两者的方向，再根据这个得到的方向，假定之前的两个方向某一个不变，计算另一个垂直的方向，这样得到三个正交的方向，计算过程类似于：      
![](https://i.imgur.com/V9xuW3c.png)       
在广告牌技术中需要指定一个锚点，这个锚点在旋转过程中是固定不变的，以此来确定多边形在空间中的位置。     

完整代码：     

		Shader "Custom/Chapter11_Billboarding" {
		Properties{
			_Color("Color",Color)=(1,1,1,1)
			_MainTex("MainTex",2D)="white"{}
			_VerticalBillboarding("VerticalBillboarding",Range(0,1))=1
		}
		SubShader{
			Tags{"Queue"="Transparent" "IgnoreProjector"="True" "RendererType"="Transparent" "DisableBatching"="True"}
			Pass{
				Tags{"LightMode"="ForwardBase"}
				ZWrite Off
				Blend SrcAlpha OneMinusSrcAlpha
				Cull Off  
				CGPROGRAM 
				#pragma vertex vert
				#pragma fragment frag
				#include "UnityCG.cginc"
				#include "Lighting.cginc"

				fixed4 _Color;
				sampler2D _MainTex;
				float4 _MainTex_ST;
				float _VerticalBillboarding;

				struct a2v{
					float4 vertex:POSITION;
					float4 texcoord:TEXCOORD0;

				};
				struct v2f{
					float4 pos:SV_POSITION;
					float2 uv:TEXCOORD0;
				};

				v2f vert(a2v v){
					v2f o;
					float3 center=float3(0,0,0);
					//选择模型空间的原点作为变换锚点
					float3 viewer=mul(unity_WorldToObject,float4(_WorldSpaceCameraPos,1));
					//将模型空间的观察方向作为法线方向
					float3 normalDir=viewer-center;
					normalDir.y=normalDir.y*_VerticalBillboarding;
					normalDir=normalize(normalDir);
					//当_VerticalBillboarding的值为1时，法线方向固定为视角方向，
					//当_VerticalBillboarding的值为0时，法线在Y方向上没有分量，那么向上的方向固定为(0,1,0)，这样才能保证与法线方向垂直
					float3 upDir=abs(normalDir.y)>0.999 ? float3(0,0,1) : float3(0,1,0);
					//这里对向上方向是否与法向方向相平行，防止得到错误的叉乘结果
					float3 rightDir=normalize(cross(normalDir,upDir));
					upDir=normalize(cross(normalDir,rightDir));  

					float3 offset=v.vertex.xyz-center;
					float3 localPos=center+rightDir*offset.x+upDir*offset.y+normalDir*offset.z;  

					o.pos=UnityObjectToClipPos(float4(localPos,1));
					o.uv=TRANSFORM_TEX(v.texcoord,_MainTex);
					return o;

				}

				fixed4 frag(v2f i):SV_Target{
					fixed4 c=tex2D(_MainTex,i.uv);
					c.rgb*=_Color.rgb;
					return c;
				}
				ENDCG
			}
		}
		FallBack  "Transparent/VertexLit"
	}

注意事项：    

- 在顶点变换的Shader中需要做特殊处理，即关闭批处理，这样能防止Unity自动对相关模型做合并处理，从而丢失模型空间。   
- 在对顶点进行变换后，如果想要得到正确的阴影，需要添加一个自定义的ShadowCaster Pass，否则得到的阴影会是变换前的阴影。自定义的Pass中，需要使用内置的宏。完整代码为:   

		Shader "Custom/Chapter 11_Vertex Animation With Shadow" {
		Properties {
		_MainTex ("Main Tex", 2D) = "white" {}
		_Color ("Color Tint", Color) = (1, 1, 1, 1)
		_Magnitude ("Distortion Magnitude", Float) = 1
		_Frequency ("Distortion Frequency", Float) = 1
		_InvWaveLength ("Distortion Inverse Wave Length", Float) = 10
		_Speed ("Speed", Float) = 0.5
		}
		SubShader {
	
		Tags {"DisableBatching"="True"}
		Pass {
		Tags { "LightMode"="ForwardBase" }
		Cull Off
		CGPROGRAM
		#pragma vertex vert
		#pragma fragment frag
		#include "UnityCG.cginc"
		sampler2D _MainTex;
		float4 _MainTex_ST;
		fixed4 _Color;
		float _Magnitude;
		float _Frequency;
		float _InvWaveLength;
		float _Speed;
		struct a2v {
		float4 vertex : POSITION;
		float4 texcoord : TEXCOORD0;
		};
		struct v2f {
		float4 pos : SV_POSITION;
		float2 uv : TEXCOORD0;
		};
		v2f vert(a2v v) {
		v2f o;
		float4 offset;
		offset.yzw = float3(0.0, 0.0, 0.0);
		offset.x = sin(_Frequency * _Time.y + v.vertex.x * _InvWaveLength + v.vertex.y * _InvWaveLength + v.vertex.z * _InvWaveLength) * _Magnitude;
		o.pos = mul(UNITY_MATRIX_MVP, v.vertex + offset);
		o.uv = TRANSFORM_TEX(v.texcoord, _MainTex);
		o.uv += float2(0.0, _Time.y * _Speed);
		return o;
		}
		fixed4 frag(v2f i) : SV_Target {
		fixed4 c = tex2D(_MainTex, i.uv);
		c.rgb *= _Color.rgb;
		return c;
		}
		ENDCG
		}
		
		Pass {
		Tags { "LightMode" = "ShadowCaster" }
		CGPROGRAM
		#pragma vertex vert
		#pragma fragment frag
		#pragma multi_compile_shadowcaster
		#include "UnityCG.cginc"
		float _Magnitude;
		float _Frequency;
		float _InvWaveLength;
		float _Speed;
		struct v2f {
		V2F_SHADOW_CASTER;
		};
		v2f vert(appdata_base v) {
		v2f o;
		float4 offset;
		offset.yzw = float3(0.0, 0.0, 0.0);
		offset.x = sin(_Frequency * _Time.y + v.vertex.x * _InvWaveLength + v.vertex.y * _InvWaveLength + v.vertex.z * _InvWaveLength) * _Magnitude;
		v.vertex = v.vertex + offset;
		TRANSFER_SHADOW_CASTER_NORMALOFFSET(o)
		return o;
		}
		fixed4 frag(v2f i) : SV_Target {
		SHADOW_CASTER_FRAGMENT(i)
		}
		ENDCG
		}
		}
		FallBack "VertexLit"
		}

### 通过顶点位置判断实现类似黑洞效果 ###
该效果的首发在[https://zhuanlan.zhihu.com/p/32131147](https://zhuanlan.zhihu.com/p/32131147)       
参考喵神的优化建议，自己对其中代码做了部分优化，自己实现的效果：     
![](https://i.imgur.com/jD5t3i5.png)        
![](https://i.imgur.com/pXmmLup.png)         
该效果的核心思想在与判断角色模型与场景中的“黑洞”的位置和距离关系，当两者距离到达一个阈值时，开始受到类似吸引的效果，实际上是将顶点向“黑洞“的方向靠近，若变化后的新顶点超过了”黑洞“的位置，则将其位置设置为”黑洞“的位置，即被吞噬，因此这个变化过程在顶点着色器中进行。     
为了实时获取”黑洞“位置，需要外部脚本将位置传入Shader中，即  

	public class BlackHole : MonoBehaviour
	{

    public Transform blackHole;
    private Material mat;
	// Use this for initialization
	void Start ()
	{
	    mat = GetComponent<Renderer>().material;
	}
	
	// Update is called once per frame
	void Update () {
		mat.SetVector("_BlackHolePos",blackHole.position);
	}
	}    
Shader代码：      

	Shader "Custom/BlackHole" {
	Properties{
		_MainColor("MainColor",Color)=(1,1,1,1)
		_MainTex("MainTex",2D)="white"{}
		_BumpTex("BumpTex",2D)="bump"{}
		_BumpScale("BumpScale",Float)=1.0
		_Specular("Specular",Color)=(1,1,1,1)
		_Gloss("Gloss",Range(8.0,255))=20   

		//设置开始受影响的范围
       _Range("Range",Float)=5
	   //靠近黑洞位置的影响系数
	   _HoleAmount("HoleAmount",Range(1.0,2.0))=1.5
	}
	SubShader{
		Pass{
			Tags{"LightMode"="ForwardBase"}

			CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag
			#include "Lighting.cginc"
			#include "UnityCG.cginc"

			fixed4 _MainColor;
			sampler2D _MainTex;
			float4 _MainTex_ST;
			sampler2D _BumpTex;
			float4 _BumpTex_ST;
			float _BumpScale;
			fixed4 _Specular;
			float _Gloss;

			float _Range;
			half _HoleAmount; 
			float3 _BlackHolePos;

			struct a2v{
				float4 vertex:POSITION;
				float3 normal:NORMAL;
				float4 tangent:TANGENT;
				float4 texcoord:TEXCOORD0;	
			};

			struct v2f{
				float4 pos:SV_POSITION;
				float4 uv:TEXCOORD0;
				float3 lightDir:TEXCOORD1;
				float3 viewDir:TEXCOORD2;
			};

			v2f vert(a2v v){
				v2f o;
				//获得模型顶点在世界空间的位置
				float4 oriWorldPos=mul(unity_ObjectToWorld,v.vertex);
				//判断与黑洞的距离
				float dis=distance(oriWorldPos,_BlackHolePos);
				//设置变化后的新顶点，初始值为原顶点的世界空间坐标
				float4 worldPos=oriWorldPos;

				//该部分为原作者对顶点变换的判断过程

				//if(dis<_Range){
				//新的顶点位置在靠近黑洞的方向上受到的偏移影响，越靠近黑洞，偏移值越大
				//	worldPos.xyz+=normalize(_BlackHolePos-oriWorldPos)*(_Range-dis);
				//当变换后的顶点位置超出了黑洞位置时，该顶点位置即为黑洞位置，即完全被吞噬
				//这里是通过判断(worldPos-_BlackHolePos)和(_BlackHolePos-oriWorldPos)向量的方向
				//来确定是否超过黑洞，若同向则超过，其实自己动手画一下向量关系很直白
				//	if(dot((worldPos-_BlackHolePos),(_BlackHolePos-oriWorldPos))>0){
				//		worldPos.xyz=_BlackHolePos;
				//	}
				//}

				//该部分通过lerp函数来避免上面的两次if判断(if判断相对比较耗性能)
				//_HoleAmount系数是为了使靠近黑洞时受到的吞噬效果更加明显
				worldPos.xyz=lerp(oriWorldPos,_BlackHolePos,clamp((_Range-dis)*_HoleAmount/_Range,0,1));
				
				o.pos=mul(UNITY_MATRIX_VP,worldPos);
				o.uv.xy=TRANSFORM_TEX(v.texcoord,_MainTex);
				o.uv.zw=TRANSFORM_TEX(v.texcoord,_BumpTex);

				float3 biNormal=cross(normalize(v.normal),normalize(v.tangent.xyz))*v.tangent.w;
				float3x3 rotation=float3x3(v.tangent.xyz,biNormal,v.normal);

				o.lightDir=mul(rotation,ObjSpaceLightDir(v.vertex).xyz);
				o.viewDir=mul(rotation,ObjSpaceViewDir(v.vertex).xyz);

				return o;
			}

			fixed4 frag(v2f i):SV_Target{
				fixed3 tangentLightDir=normalize(i.lightDir);
				fixed3 tangentViewDir=normalize(i.viewDir);

				fixed3 tangentNormal=UnpackNormal(tex2D(_BumpTex,i.uv.zw));
				tangentNormal.xy*=_BumpScale;
				tangentNormal.z=sqrt(1.0-saturate(dot(tangentNormal.xy,tangentNormal.xy)));

				fixed3 albedo=tex2D(_MainTex,i.uv.xy)*_MainColor.rgb;
				fixed3 ambient=UNITY_LIGHTMODEL_AMBIENT.xyz*albedo;
				fixed3 diffuse=_LightColor0.rgb*albedo*max(0,dot(tangentNormal,tangentLightDir));
				fixed3 halfDir=normalize(tangentLightDir+tangentViewDir);
				fixed3 specular=_LightColor0.rgb*_Specular.rgb*pow(saturate(dot(halfDir,tangentNormal)),_Gloss);

				return fixed4(ambient+diffuse+specular,1.0);
			}
			ENDCG
		}
	}
	}
和普通的法线高光Shader的区别在于，在顶点着色器中增加了黑洞变换的效果，通过设置_Range的值来控制影响范围。      
受影响前：   
![](https://i.imgur.com/ybaqJjP.png)    
受影响后：    
![](https://i.imgur.com/c1ggNo5.png)

### 屏幕后处理效果 ###
屏幕后处理是指渲染完整个场景得到屏幕图像后，再对屏幕图像做处理，实现屏幕特效。  

实现屏幕后处理效果的关键在于得到渲染后的屏幕图像,Unity提供了对应的接口 **OnRenderImage** ,其函数声明       

	MonoBehaviour.OnRenderImage(RnederTexture src, RenderTexture dest)   

参数 **src** ：源纹理，用于存储当前渲染的得到的屏幕图像  
参数 **dest** ：目标纹理，经过一系列操作后，用于显示到屏幕的图像     

在OnRenderImage函数中，通常调用 **Graphics.Blit函数** 完成对渲染纹理的**处理**      

	public static void Blit(Texture src,RenderTexture dest);     
	public static void Blit(Texture src, RenderTexture dest, Material mat, int pass=-1);
	public static void Blit(Texture src,Material mat, int pass=-1);   
参数 **src** ：源纹理 通常指当前屏幕渲染纹理或上一步处理得到的纹理     
参数 **dest** : 目标渲染纹理，如果值为null，则会直接将结果显示到屏幕上       
参数 **mat** : 使用的材质，该材质使用的Unity Shader将会进行各种屏幕后处理操作, src对应的纹理会传递给Shader中_MainTex的纹理属性    
参数 **pass**：默认值为-1，表示依次调用Shader内所有Pass，否则调用索引指定的Pass      

默认情况下，**OnRenderImage** 函数会在所有不透明和透明Pass执行完后被调用，若想在不透明Pass执行完后调用，即不对透明物体产生影响，可以在OnRenderImage函数前添加**ImageEffectOpaque**的属性实现。  

Unity中实现屏幕后处理出效果，通常步骤：  
1. 在摄像机添加屏幕后处理脚本，该脚本中会实现OnRenderImage函数获取当前屏幕渲染纹理    
2. 调用Graphics.Blit函数使用特定Shader对当前图像进行处理，再将最终目标纹理渲染到屏幕上。对于复杂的后处理特效，需要多次调用Graphics.Blit函数     

在进行屏幕后处理前，需要检查是否满足后处理条件，创建一个用于屏幕后处理效果的基类，在实现屏幕后处理效果时，只需继承该基类，再实现派生类中的具体操作，完整代码：   

	[ExecuteInEditMode]
	[RequireComponent(typeof(Camera)]
	public class PostEffectsBase : MonoBehaviour {
   	protected void CheckResources() {
        bool isSupported = CheckSupport();
        if (isSupported == false) {
            NotSupport();
        }
    }

    protected bool CheckSupport() {
        if (SystemInfo.supportsImageEffects == false || SystemInfo.supportsRenderTextures == false) {
            return false;
        }
        return true;
    }

    protected void NotSupport() {
        enabled = false;
    }
	// Use this for initialization
	void Start () {
        //检查资源和条件是否支持屏幕后处理
        CheckResources();  
	}

    protected Material CheckShaderAndCreateMaterial(Shader shader, Material material) {
        if (shader == null) {
            return null;
        }
        if (shader.isSupported && material && material.shader == shader) {
            return material;
        }
        if (!shader.isSupported)
        {
            return null;
        }
        else {
            material = new Material(shader);
            material.hideFlags = HideFlags.DontSave;
            if (material)
                return material;
            else
                return null;
        }
    }
	}

**调整屏幕的亮度、饱和度和对比度**   
摄像机脚本：  

	public class BrightnessSaturationAndContrast : PostEffectsBase{

    public Shader briSatConShader;
    private Material briSatConMaterial;

    public Material material {
        get {
            briSatConMaterial = CheckShaderAndCreateMaterial(briSatConShader, briSatConMaterial);
            return briSatConMaterial;
                }
    }

    [Range(0.0f, 3.0f)]
    public float brightness = 1.0f;
    [Range(0.0f, 3.0f)]
    public float saturation = 1.0f;
    [Range(0.0f, 3.0f)]
    public float contrast = 1.0f;


    void OnRenderImage(RenderTexture src,RenderTexture dest) {
        if (material != null)
        {
            material.SetFloat("_Brightness", brightness);
            material.SetFloat("_Saturation", saturation);
            material.SetFloat("_Contrast", contrast);
            //若材质可用，将参数传递给材质，在调用Graphics.Blit进行处理
            Graphics.Blit(src, dest, material);
        }
        else {
            //否则将图像直接输出到屏幕，不做任何处理
            Graphics.Blit(src, dest);
        }
    }
	}    
继承自屏幕处理基类PostEffectsBase，指定shader，并根据该shader创建新的材质，通过OnRenderImage方法和Graphics.Blit方法将参数传递到shader中，完成着色，shader代码：   

	Shader "Custom/Chapter12_BrightnessSaturateAndContrast" {
	Properties{
		_MainTex("Maintex",2D)="white"{}
		_Brightness("Brightness",Float)=1
		_Saturation("Saturation",Float)=1
		_Contrast("Contrast",Float)=1

		//Graphics.Blit(src,dest,material)会将第一个参数传递给Shader中名为_MainTex的属性
	}
	SubShader{
		Pass{
			ZTest Always
			Cull Off
			ZWrite Off

			CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag
			#include "UnityCG.cginc"

			sampler2D _MainTex;
			float4 _MainTex_ST;
			half _Brightness;
			half _Saturation;
			half _Contrast;

			struct v2f{
				float4 pos:SV_POSITION;
				float2 uv:TEXCOORD0;
			};

			//appdata_img为Unity内置的结构体，只包含图像处理必须的顶点坐标和纹理坐标
			v2f vert(appdata_img v){
				v2f o;
				o.pos=UnityObjectToClipPos(v.vertex);
				o.uv=v.texcoord; //屏幕后处理得到的纹理和要输出的纹理坐标是相同的

				return o;
			}

			fixed4 frag(v2f i):SV_Target{
				fixed4 renderTex=tex2D(_MainTex,i.uv);  

				//应用明亮度
				fixed3 finalColor=renderTex.rgb*_Brightness;

				//应用饱和度，乘以一定系数
				fixed luminance=0.2125*renderTex.r+0.7154*renderTex.g+0.0721*renderTex.b;
				fixed3 luminanceColor=fixed3(luminance,luminance,luminance);
				finalColor=lerp(luminanceColor,finalColor,_Saturation);

				//应用对比度
				fixed3 avgColor=fixed3(0.5,0.5,0.5);
				finalColor=lerp(avgColor,finalColor,_Contrast);

				return fixed4(finalColor,renderTex.a);
			}
			ENDCG
		}
	}
	FallBack Off
	}    
实现效果：          
调节前：   
![](https://i.imgur.com/uUfExxa.png)        
调节后：    
![](https://i.imgur.com/z3ZnYEO.png)     
摄像机脚本参数设置：    
![](https://i.imgur.com/yNxn11z.png)     

**边缘检测效果**      
屏幕后处理中的边缘检测是利用一些边缘检测算子对图像中的像素进行**卷积操作**,关于图像处理中的卷积计算，可以参考这篇博客（[http://www.cnblogs.com/freeblues/p/5738987.html](http://www.cnblogs.com/freeblues/p/5738987.html)）   
常见的边缘检测算子有：      
![](https://i.imgur.com/imCZYof.png)     
边缘检测算子包含两个方向的卷积核，用来计算水平方向和竖直方向的梯度值，得到两个方向的梯度值，而整体的梯度值可以是两个方向上的梯度值平方和开根，为了节约性能可以是两个方向梯度值的绝对值求和，整体梯度的值越大，说明该像素点则越有可能是边缘位置。

边缘检测过程中主要是获取中心像素点及周围的像素点的值，再与边缘检测算子做乘积，计算出对应的梯度，再通过梯度进行插值
完整代码：   

	Shader "Custom/Chapter12_EdgeDetetion" {
	Properties{
		_MainTex("MainTex",2D)="white"{}
		_EdgeOnly("Edge Only",Float)=1.0
		_EdgeColor("Edge Color",Color)=(0,0,0,1)
		_BackgroundColor("BackgroundColor",Color)=(1,1,1,1)
	}
	SubShader{
		Pass{
			ZTest Always
			Cull Off
			ZWrite Off

			CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag
			#include "UnityCG.cginc"

			sampler2D _MainTex;
			float4 _MainTex_ST;
			half4 _MainTex_TexelSize;
			float _EdgeOnly;
			fixed4 _EdgeColor;
			fixed4 _BackgroundColor;

			struct v2f{
				float4 pos:SV_POSITION;
				half2 uv[9]:TEXCOORD0;
			};

			v2f vert(appdata_img v){
				v2f o;
				o.pos=UnityObjectToClipPos(v.vertex);
				half2 uv=v.texcoord;
				//中线像素点及周围8个像素点的UV坐标
				o.uv[0]=uv+_MainTex_TexelSize.xy*half2(-1,-1);
				o.uv[1]=uv+_MainTex_TexelSize.xy*half2(0,-1);
				o.uv[2]=uv+_MainTex_TexelSize.xy*half2(1,-1);
				o.uv[3]=uv+_MainTex_TexelSize.xy*half2(-1,0);
				o.uv[4]=uv+_MainTex_TexelSize.xy*half2(0,0);
				o.uv[5]=uv+_MainTex_TexelSize.xy*half2(1,0);
				o.uv[6]=uv+_MainTex_TexelSize.xy*half2(-1,1);
				o.uv[7]=uv+_MainTex_TexelSize.xy*half2(0,1);
				o.uv[8]=uv+_MainTex_TexelSize.xy*half2(1,1);
			
				return o;
			}

			fixed luminance(fixed4 color){
				return color.r*0.2125+color.g*0.7154+color.b*0.0721;
			}
			half Sobel(v2f i){
				const half Gx[9]={-1,-2,-1,
											0,0,0,
											1,2,1};
				const half Gy[9]={-1,0,1,
											-2,0,2,
											-1,0,1};
				half texColor;
				half edgeX;
				half edgeY;
				for(int it=0;it<9;it++){
					texColor=luminance(tex2D(_MainTex,i.uv[it]));
					edgeX+=texColor*Gx[it];
					edgeY+=texColor*Gy[it];
				}
				half edge=1-abs(edgeX)-abs(edgeY);

				return edge;
			}

			fixed4 frag(v2f i):SV_Target{
				half edge=Sobel(i);

				fixed4 withEdgeColor=lerp(_EdgeColor,tex2D(_MainTex,i.uv[4]),edge);
				fixed4 withBackGroundColor=lerp(_EdgeColor,_BackgroundColor,edge);
				//非边缘的像素点withEdgeColor插值结果靠近原有颜色
				//而withBackGroundColor的插值结果就靠近给定的背景色
				return lerp(withEdgeColor,withBackGroundColor,_EdgeOnly);
				//最后的返回结果，通过_EdgeOnly来混合，
				//来控制处理后的图像非边缘的颜色是靠近原颜色还是给定的背景色
			}
			ENDCG
		}
	}
	FallBack Off
	}   
实例效果：    
![](https://i.imgur.com/GePR5mV.png)       
![](https://i.imgur.com/xarceQh.png)      


**高斯模糊**         
模糊实现的方式有多种，有均值模糊，中值模糊。均值模糊也会使用图像卷积操作，使用的卷积核的各个元素值都相等，相加的和为1，卷积后得到的像素值是周围像素值的平均值。而中值模糊是选择周围领域内的像素值并对其排序后选择中间值作为返回结果替换原来的颜色。     
高斯模糊是通过使用高斯核对周围像素进行计算，高斯核中的元素值并不是相等的而是满足正态分布，基于高斯方程，并且元素值进行归一化操作，使每个权重值除以所有的权重值的和，即所有的权重值相加的和为1，这样处理后的图像不会变暗。关于高斯核，这里有一篇详细介绍的博客[http://www.cnblogs.com/zxj015/archive/2013/05/12/3074612.html](http://www.cnblogs.com/zxj015/archive/2013/05/12/3074612.html)             
高斯核的维数越高，处理后的图像没模糊程度会越高。当使用一个NxN的高斯核对一个WxH的图像进行处理，那么对像素值的采样结果为NxNxWxH次，N越大采样的次数就越大。二维的高斯核可以拆成两个一维的高斯核，得到的结果和直接使用二维高斯核的结果是一样的，这样可以使采样次数降低到2xNxWxH次。而两个一维的高斯核中权重值有重复的权重值，例如一个5x5的一维高斯核只需要记录三个权重值。     
![](https://i.imgur.com/9Ebuc5Y.png)         
后续将使用该高斯核实现模糊效果，实现过程中将会使用两个Pass，第一个使用竖直方向的高斯核对图像进行处理，第二个使用水平方向的高斯核对图像进行处理。同时会增加降采样及应用次数（迭代次数）控制模糊程度。           

完整代码：     

	public class GaussianBlur : PostEffectsBase
	{
    public Shader gaussianBlurShader;
    private Material gaussianBlurMaterial = null;

    public Material material
    {
        get
        {
            gaussianBlurMaterial = CheckShaderAndCreateMaterial(gaussianBlurShader, gaussianBlurMaterial);
            return gaussianBlurMaterial;
        }
    }

    [Range(0, 4)]   //迭代次数，值越大，模糊应用次数越高
    public int iterations = 3;
    [Range(0.2f, 3.0f)] //模糊计算的范围，越大越模糊    
    public float blurSpread = 0.6f;
    [Range(1, 8)] //降采样数值，越大，计算的像素点越少，节约性能，但是降采样的值太大会出现像素化风格
    public int downSample = 2;

    void OnRenderImage(RenderTexture src,RenderTexture dest)
    {
        //最简单的处理
        //if (material != null)
        //{
        //    int rtW = src.width;
        //    int rtH = src.height;
        //    RenderTexture buffer = RenderTexture.GetTemporary(rtW, rtH, 0);
        //    //使用RenderTexture.GetTemporary()函数分配一块与屏幕图像大小相同的缓冲区
        //    //由于高斯模糊需要使用两个Pass，而第一个Pass的结果就放在这个缓冲区内保存
        //    Graphics.Blit(src, buffer, material, 0);
        //    Graphics.Blit(buffer, dest, material, 1);

        //    RenderTexture.ReleaseTemporary(buffer);
        //}
        //else
        //{
        //    Graphics.Blit(src,dest);
        //}  

        //增加降采样的处理
        //if (material != null)
        //{
        //    int rtW = src.width/downSample;
        //    int rtH = src.height/downSample;
        //    //增加降采样 
        //    RenderTexture buffer = RenderTexture.GetTemporary(rtW, rtH, 0);

        //    Graphics.Blit(src, buffer, material, 0);
        //    Graphics.Blit(buffer, dest, material, 1);

        //    RenderTexture.ReleaseTemporary(buffer);
        //}
        //else
        //{
        //    Graphics.Blit(src,dest);
        //}


        //增加降采样处理及迭代的影响
        if (material != null)
        {
            int rtW = src.width/downSample;
            int rtH = src.height/downSample;
            RenderTexture buffer0 = RenderTexture.GetTemporary(rtW, rtH, 0);
            buffer0.filterMode = FilterMode.Bilinear;

            Graphics.Blit(src, buffer0);
            for (int i = 0; i < iterations; i++)
            {
                material.SetFloat("_BlurSize", 1.0f + i*blurSpread);
                //_BlurSize用来控制采样的距离，在n次迭代下，每次迭代会计算向外一圈的采样结果，
                //再进行高斯核的横向和纵向的计算，结果也就会更加模糊
                RenderTexture buffer1 = RenderTexture.GetTemporary(rtW, rtH, 0);
                Graphics.Blit(buffer0, buffer1, material, 0);
                RenderTexture.ReleaseTemporary(buffer0);
                buffer0 = buffer1;
                buffer1 = RenderTexture.GetTemporary(rtW, rtH, 0);
                Graphics.Blit(buffer0, buffer1, material, 1);
                RenderTexture.ReleaseTemporary(buffer0);
                buffer0 = buffer1;

            }
            Graphics.Blit(buffer0, dest);
            RenderTexture.ReleaseTemporary(buffer0);
        }
        else
        {
            Graphics.Blit(src,dest);
        }
    }
	}

Shader部分：     

	Shader "Custom/Chapter12_GaussianBlur" {
	Properties {
		_MainTex("MainTex",2D)="white"{}
		_BlurSize("BlurSize",Float)=1.0
	}
	SubShader {

		
		CGINCLUDE          //使用时，在Pass中直接指定需要使用的着色器函数，避免编写完一样的片元着色器函数
			
		#include "UnityCG.cginc"

		sampler2D _MainTex;
		half4 _MainTex_TexelSize;
		float _BlurSize;

		struct v2f{
			float4 pos:SV_POSITION;
			half2 uv[5]:TEXCOORD0;
		};
		
		v2f vertBlurVertical(appdata_img v){
			v2f o;
			o.pos=UnityObjectToClipPos(v.vertex);
			half2 uv=v.texcoord;
			o.uv[0]=uv;
			o.uv[1]=uv+float2(0.0,_MainTex_TexelSize.y*1.0)*_BlurSize;
			o.uv[2]=uv+float2(0.0,_MainTex_TexelSize.y*2.0)*_BlurSize;
			o.uv[3]=uv-float2(0.0,_MainTex_TexelSize.y*1.0)*_BlurSize;
			o.uv[4]=uv-float2(0.0,_MainTex_TexelSize.y*2.0)*_BlurSize;

			return o;
		}

		v2f vertBlurHorizantal(appdata_img v){
			v2f o;
			o.pos=UnityObjectToClipPos(v.vertex);
			half2 uv=v.texcoord;
			o.uv[0]=uv;
			o.uv[1]=uv+float2(_MainTex_TexelSize.x*1.0,0.0)*_BlurSize;
			o.uv[2]=uv+float2(_MainTex_TexelSize.x*2.0,0.0)*_BlurSize;
			o.uv[3]=uv-float2(_MainTex_TexelSize.x*1.0,0.0)*_BlurSize;
			o.uv[4]=uv-float2(_MainTex_TexelSize.x*2.0,0.0)*_BlurSize;

			return o;
		}

		fixed4 frag(v2f i):SV_Target{
			float weight[3]={0.4026,0.2442,0.0545};  
			fixed3 sum=tex2D(_MainTex,i.uv[0]).rgb*weight[0]; 

			for(int it=1;it<3;it++){
				sum+=tex2D(_MainTex,i.uv[it])*weight[it];
				sum+=tex2D(_MainTex,i.uv[it+2])*weight[it];
			}

			return fixed4(sum,1.0);
		}
		ENDCG

		ZTest Always
		ZWrite Off
		Cull Off   

		Pass{
			NAME "GAUSSIAN_BLUR_VERTICAL"
			CGPROGRAM
				#pragma vertex vertBlurVertical
				#pragma fragment frag	
			ENDCG
		}

		Pass{
			NAME "GAUSSIAN_BLUR_HORIZANTAL"
			CGPROGRAM
				#pragma vertex vertBlurHorizantal
				#pragma fragment frag	
			ENDCG
		}
	}
	FallBack Off
	}

实例效果：     
原图：     
![](https://i.imgur.com/PsP1maV.png)       
模糊效果：      
![](https://i.imgur.com/YgfGPbo.png)    
参数设置     
![](https://i.imgur.com/XSuarq3.png)    

**Bloom效果**       
Bloom特效在游戏中应用比较常见，可以模拟真实摄像机的一种图像效果，使画面中较亮的区域“扩散”到周围的区域，造成朦胧的效果。    
Bloom实现原理：       
先根据一定的阈值提取图像中较亮的区域，并存储到一张渲染纹理中，然后对该渲染纹理做模糊处理，最后再将该渲染纹理与原纹理进行混合。    
完整代码：    

	public class Bloom : PostEffectsBase {
    public Shader bloomShader;
    private Material bloomMaterial;
    public Material material {
        get { bloomMaterial = CheckShaderAndCreateMaterial(bloomShader, bloomMaterial);
            return bloomMaterial;
        }
    }

    [Range(0, 4)]
    public int iterations = 3;
    [Range(0.2f, 3.0f)]
    public float blurSpread = 0.6f;
    [Range(1, 8)]
    public int downSample = 2;
    [Range(0.0f, 4.0f)]
    public float luminanceThreshold = 0.6f;

    void OnRenderImage(RenderTexture src, RenderTexture dest) {
        if (material != null)
        {
            material.SetFloat("_LuminanceThreshold", luminanceThreshold);

            int rtW = src.width / downSample;
            int rtH = src.height / downSample;

            RenderTexture buffer0 = RenderTexture.GetTemporary(rtW, rtH, 0);
            buffer0.filterMode=FilterMode.Bilinear;
            Graphics.Blit(src, buffer0, material, 0);

            for (int it = 0; it < iterations; it++)
            {
                material.SetFloat("_BlurSize", 1.0f + it * blurSpread);
                RenderTexture buffer1 = RenderTexture.GetTemporary(rtW, rtH, 0);
                Graphics.Blit(buffer0, buffer1, material, 1);
                RenderTexture.ReleaseTemporary(buffer0);
                buffer0 = buffer1;
                buffer1 = RenderTexture.GetTemporary(rtW, rtH, 0);
                Graphics.Blit(buffer0, buffer1, material, 2);
                RenderTexture.ReleaseTemporary(buffer0);
                buffer0 = buffer1;
            }
            material.SetTexture("_BloomTex", buffer0);
            Graphics.Blit(src, dest, material, 3);
            RenderTexture.ReleaseTemporary(buffer0);
        }
        else {
            Graphics.Blit(src,dest);
        }
    }
	}   
Shader部分：   

	Shader "Custom/Chapter12_Bloom" {
	Properties{
		_MainTex("MainTex",2D)="white"{}
		_BloomTex("BloomTex",2D)="white"{}
		_LuminanceThreshold("LuminanceThreshold",Float)=0.5
		_BlurSize("BlurSize",Float)=1.0
	}
	SubShader{
		CGINCLUDE
		#include "UnityCG.cginc"
		sampler2D _MainTex;
		float4 _MainTex_TexelSize;
		sampler2D _BloomTex;
		float _LuminanceThreshold;
		float _BlurSize;

		struct v2f{
			float4 pos:SV_POSITION;
			half2 uv:TEXCOORD0;
		};

		v2f vertExtractBright(appdata_img v){
			v2f o;
			o.pos=UnityObjectToClipPos(v.vertex);
			o.uv=v.texcoord;
			return o;
		}

		fixed luminance(fixed4 color){
			return 0.2125*color.r+0.7154*color.g+0.0721*color.b;
		}
		fixed4 fragExtractBright(v2f i):SV_Target{
			fixed4 c=tex2D(_MainTex,i.uv);
			fixed val=clamp(luminance(c)-_LuminanceThreshold,0.0,1.0);  

			return c*val;
		}

		struct v2fBloom{
			float4 pos:SV_POSITION;
			half4 uv:TEXCOORD0;
		};
		v2fBloom vertBloom(appdata_img v){
			v2fBloom o;
			o.pos=UnityObjectToClipPos(v.vertex);
			o.uv.xy=v.texcoord;
			o.uv.zw=v.texcoord; 

			#if UNITY_UV_STARTS_AT_TOP 
			if(_MainTex_TexelSize.y<0.0)
				o.uv.w=1.0-o.uv.w;
			#endif   

			return o;
		}

		fixed4 fragBloom(v2fBloom i):SV_Target{
			return tex2D(_MainTex,i.uv.xy)+tex2D(_BloomTex,i.uv.zw);
		}
		ENDCG 

		ZWrite Off 
		ZTest Always
		Cull Off
		Pass{
			CGPROGRAM
				#pragma vertex vertExtractBright
				#pragma fragment fragExtractBright
			ENDCG
		}
		UsePass "Custom/Chapter12_GaussianBlur/GAUSSIAN_BLUR_VERTICAL"
		UsePass "Custom/Chapter12_GaussianBlur/GAUSSIAN_BLUR_HORIZANTAL"
		Pass{
			CGPROGRAM
				#pragma vertex vertBloom
				#pragma fragment fragBloom
			ENDCG
		}
	}
	FallBack Off
	}  
实例效果：    
原图：    
![](https://i.imgur.com/2rh8HTu.png)      
Bloom效果：    
![](https://i.imgur.com/Ubbb7d1.png)         
参数设置：       
![](https://i.imgur.com/SU91CB8.png)    

**运动模糊**      
运动模糊是真实世界中的摄像机的一种效果。如果摄像机在曝光时，场景发生变化，就会产生模糊的画面。计算机产生的图像由于不存在曝光，渲染出来的图像往往是线条清晰，缺少运动模糊。       
运动模糊的实现有多种方式      

- 利用**积累缓存**混合多张连续图片   
当物体移动产生多张图片后，取这些图片的平均值作为最后的运动模糊图像。这种方式对性能消耗有较大影响，获取多张帧图像需要在同一帧内多次进行场景渲染。    
- 使用**速度缓存**      
这个缓存中存储各个像素的运动速度，使用该值决定模糊的方向和大小。      

这里使用类似第一种方法实现运动模糊，同一帧中不需要对场景渲染多次，而是保存当前结果，然后将当前结果叠加到之前的渲染图像中，产生类似拖尾的运动轨迹视觉效果。  
实例代码：    

	public class MotionBlur : PostEffectsBase
	{
    public Shader motionBlurShader;
    private Material motionBlurMaterial;

    public Material material
    {
        get
        {
            motionBlurMaterial = CheckShaderAndCreateMaterial(motionBlurShader, motionBlurMaterial);
            return motionBlurMaterial;
        }
    }  

    //运动模糊在混合图像时使用的模糊参数
    [Range(0.0f, 0.9f)]
    public float blurAmount = 0.5f;

    private RenderTexture accumulationTexture;

    //当脚本不运行时，销毁accumulationTure，目的是下一次开始应用运动模糊时重新叠加图像
    void OnDisable()
    {
        DestroyImmediate(accumulationTexture);
    }

    void OnRenderImage(RenderTexture src,RenderTexture dest)
    {
        if (material != null)
        {
            if (accumulationTexture == null || accumulationTexture.width != src.width ||
                accumulationTexture.height != src.height)
            {
                DestroyImmediate(accumulationTexture);
                accumulationTexture = new RenderTexture(src.width, src.height, 0);
                accumulationTexture.hideFlags = HideFlags.HideAndDontSave;
                //这里由于自己控制该变量的销毁，因此将hideFlags设置为HideAndDontSave，意味着该变量
                //不会显示在Hierarchy,也不会保存到场景中
                Graphics.Blit(src, accumulationTexture);
            }

            accumulationTexture.MarkRestoreExpected();
            //这里使用MarkRestoreExpected()方法表明需要进行一个渲染纹理的恢复操作
            //恢复操作发生在渲染到纹理而该纹理没有被提前清空或销毁的情况下，每次调用OnRenderImage()时需要把当前
            //的帧图像和accumulationTexture中的图像混合，accumulationTexture不需要提前清空，因为它保存了之前的混合结果  

            material.SetFloat("_BlurAmount", 1.0f - blurAmount);
            Graphics.Blit(src, accumulationTexture, material);
            Graphics.Blit(accumulationTexture, dest);
        }
        else
        {
            Graphics.Blit(src,dest);
        }
    }
	}

Shader代码：   

	Shader "Custom/Chapter12_MotionBlur" {
	Properties{
		_MainTex("Maintex",2D)="white"{}
		_BlurAmount("BlurAmount",Float)=1.0
	}
	SubShader{
		CGINCLUDE
			#include "UnityCG.cginc"  
			
			sampler2D _MainTex;
			float _BlurAmount;

			struct v2f{
				float4 pos:SV_POSITION;
				half2 uv:TEXCOORD0;
			};

			v2f vert(appdata_img v){
				v2f o;
				o.pos=UnityObjectToClipPos(v.vertex);
				o.uv=v.texcoord;

				return o;
			}

			//对当前图像进行采样，将其A通道的值设为_BlurAmount,在后续混合时可以使用透明通道进行混合
			fixed4 fragRGB(v2f i):SV_Target{
				return fixed4(tex2D(_MainTex,i.uv).rgb,_BlurAmount);
			}

			//直接返回当前图像的采样结果，维护渲染纹理的透明通道值，不让其受到混合时使用的透明度值的影响
			half4 fragA(v2f i):SV_Target{
				return tex2D(_MainTex,i.uv);
			}		
		ENDCG

		ZTest Always
		Cull Off
		ZWrite Off
		//这里将RGB和A通道分开，是由于在做混合时，按照_BlurAmount参数值将源图像和目标图像进行混合
		//而同时不让其纹理受到A通道值的影响，只是用来做混合，不改变其透明度
			Pass{
				Blend SrcAlpha OneMinusSrcAlpha
				ColorMask RGB
				CGPROGRAM
					#pragma vertex vert
					#pragma fragment fragRGB
				ENDCG
			}
			Pass{
				Blend One Zero
				ColorMask A
				CGPROGRAM
					#pragma vertex vert
					#pragma fragment fragA
				ENDCG
			}
	}
	}

实例效果：    
原场景：   
![](https://i.imgur.com/BRgy7NT.png)      
运动模糊效果：      
![](https://i.imgur.com/JRezmfW.png)     

### 使用深度和法线纹理 ###
很多时候不仅需要当前屏幕的信息，同时希望得到深度和法线信息。例如在做边缘检测时，是直接根据渲染完后的屏幕图像的RGB值进行计算的，如果某些物体受到阴影的影响，那么检测结果就不一定准确。如果可以直接在深度纹理和法线纹理上进行边缘检测，那么就不会受到场景中的阴影的影响。     

**获取深度和法线纹理**        
首先深度纹理是一张纹理图，既然是一张纹理图，那么其值的范围是在[0,1]之间的，那么这个值的范围是如何转化得到的呢？             
这些深度值是经过顶点变化后经过透视除法得到归一化的设备坐标（NDC）后转化得来，一个顶点在经过MVP的矩阵变换后从模型空间变换到投影空间，再经过裁剪，透视除法和屏幕映射最终在屏幕上成像。**这里有一点需要注意的是**，在顶点着色器中，顶点只经历了MVP的矩阵变换，得到的只是投影空间的坐标，其裁剪，透视除法和屏幕映射都是在顶点着色器之后进行的。 这里提供一个顶点转换过程及着色器处理的时间轴流程图：     
![](https://i.imgur.com/9VNt95C.png)   
在得到NDC后，深度值对应顶点坐标的z分量，由于NDC的z分量范围在[-1,1]之间，因此需要通过映射公式使其映射到[0,1]之间：   

	d=Zndc/2+1/2      
d对应深度纹理中的像素值，Zndc对应NDC坐标中的z分量的值。   
Unity中的深度纹理可以来自真正的深度缓存，也可以由一个单独的Pass渲染而得。当使用延迟渲染路径时，深度纹理可以通过G-buffer得到。而当无法直接获取深度缓存时，深度纹理和法线纹理是通过单独渲染的Pass得到。这时需要在Shader中**设置正确的RenderType标签**，使物体出现在深度和法线纹理中。      

在Unity中，可以选择让一个摄像机生成一张深度纹理和法线纹理（Unity会通过访问深度缓存或特定的Pass得到），获取深度纹理和法线纹理只需设置对应的摄像机模式，然后再在Shader中访问特定的纹理属性。       
例如获取深度纹理：       

	camera.depthTextureMode=DepthTextureMode.Depth;
	//然后在Shader中声明_CameraDepthTexture变量来访问
获取深度+法线纹理：    

	camera.depthTextureMode=DepthTextureMode.DepthNormals;
	//然后在Shader中声明_CameraDepthNormalsTexture变量来访问   
还可以组合这些模式，让一个摄像机同时产生深度和深度+纹理法线纹理：      

	camera.depthTextureMode |= DepthTextureMode.Depth;
	camera.depthTextureMode |= DepthTextureMode.DepthNormals;      
	//在Shader中声明对应的变量即可使用          

当在Shader中通过_CameraDepthTexture变量得到深度纹理后，可以使用当前像素的纹理坐标对深度纹理进行采样，大多数情况下直接使用Tex2D()函数采样即可。对于需要特殊处理的平台，Unity提供统一的宏SAMPLE _DEPTH _TEXTURE，用来处理平台差异的问题。在Shader中，使用该宏对深度纹理进行采样，   

	float d=SAMPLE_DEPTH_TEXTURE(_CameraDepthTxeture,i.uv);     
	//i.uv是对应当前的像素纹理坐标
类似宏还有SAMPLE _DEPTH _TEXTURE _PROJ和SAMPLE _DEPTH _TEXTURE _LOD，SAMPLE _DEPTH _TEXTURE _PROJ宏接受两个参数，深度纹理和一个float3或float4类型的纹理坐标，内部使用tex2Dproj进行投影纹理采样。SAMPLE _DEPTH _TEXTURE _PROJ的第二个参数通常是由顶点着色器输出的插值而得到的屏幕坐标，比如：      

	float d=SAMPLE_DEPTH_TEXTURE_PROJ(_CameraDepthTexTure,UNITY_PROJ_COORD(i.srcPos));  
	//i.srcPos是在顶点着色器中通过调用ComputeScreenPos(o.pos)得到的屏幕坐标     

当通过纹理采样得到的深度值后，得到的深度值是非线性的，这是由于观察空间到投影空间的矩阵变换是非线性的。而在计算过程中，需要在线性空间下，因此需要将得到深度值变换到线性空间下，比如观察空间。将深度纹理采样的结果反映射到投影空间，再由投影空间变换到观察空间可以通过顶点变换过程的逆向过程计算出来，Unity提供了辅助函数简便计算过程：      

	LinearEyeDepth();    //负责将深度纹理的采样结果转换到观察空间下     
	Linear01Depth();   //返回一个范围在[0,1]线性深度值   
如果需要获取深度+法线纹理，可以直接使用tex2D()对_CameraDepthNormalsTexture进行采样，得到里面存储的深度和法线信息。Unity提供了DecodeDepthNormal函数对采样结果进行解码，从而得到深度值和法线方向。DecodeDepthNormal函数定义：     

	inline void DecodeDepthNormal(float4 enc, out float depth, out float3 normal)
	{
		depth=DecodeFloatRG(enc.zw);
		normal=DecodeViewNormalStereo(enc);
	}
经过解码后得到的深度值是范围在[0,1]的**线性深度值**，得到的法线则是观察空间下的法线方向。
  
**通过深度纹理实现扫描相交部分着色功能**     
扫描相交部分着色其实在游戏中很常见，比如非常火的一款游戏      
![](https://i.imgur.com/Q9i3J0u.jpg)        
玩过的朋友可能会注意到当开始缩圈的时候，毒圈经过的界面在与树木或建筑物相交处会有一圈物体轮廓的高亮显示，那么这个相交处的高亮显示就是要实现的扫描相交部分着色功能。    
要想实现这个功能，需要判断扫描平面（也就是那个毒圈）与相交处物体的位置关系，只有足够近，才会有高亮着色效果。再来看这个位置关系，实际上是通过距离摄像机位置的远近来进行判断的。因此可以利用扫描平面当前的深度值和建筑物的深度值进行判断来得到远近关系，建筑物的深度值可以通过已经存在的深度纹理得到，而扫描平面当前的深度值可以通过变换得到。因此这里要确定正确的渲染顺序，即**先渲染场景中的树木和建筑物，最后再渲染扫描平面**    

实例代码：     

	Shader "Custom/Scan" {
	Properties{
		_MainColor("MainColor",Color)=(1,1,1,1)
		_HighLightingColor("HighLightingColor",Color)=(1,1,1,1)
		_Threshold("Threshold",Float)=2.0
		_MainTex("MainTex",2D)="white"{}
	}
	SubShader{
		Tags{"Queue"="Transparent" "RenderType"="Transparent"}
		//设置合理的渲染队列，使当前扫描平面在其他物体后渲染
		Pass{
			Tags{"LightMode"="ForwardBase"}

			Blend SrcAlpha OneMinusSrcAlpha
			//设置混合，使得扫描平面后面的部分仍然被看见
			CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag
			#include "Lighting.cginc"
			#include "UnityCG.cginc"

			//提前定义
			uniform sampler2D_float _CameraDepthTexture;

			 fixed4 _MainColor;
			 fixed4 _HighLightingColor;
			 float   _Threshold;
			 sampler2D  _MainTex;
			 float4         _MainTex_ST;

			 struct a2v{
				float4 vertex:POSITION;
				float4 texcoord:TEXCOORD0;
			 };

			 struct v2f{
				float4 pos:SV_POSITION;
				float4 projPos:TEXCOORD0;
				float3 viewPos:TEXCOORD1;
				float2 uv:TEXCOORD2; 
			};

			v2f vert(a2v v){
				v2f o;
				o.pos=UnityObjectToClipPos(v.vertex);
				o.projPos=ComputeScreenPos(o.pos);
				o.viewPos=UnityObjectToViewPos(v.vertex);
				o.uv=TRANSFORM_TEX(v.texcoord,_MainTex);
				return o;
			}

			fixed4 frag(v2f i):SV_Target{
				fixed4 finalColor=_MainColor;
				float sceneZ=LinearEyeDepth(tex2Dproj(_CameraDepthTexture,UNITY_PROJ_COORD(i.projPos)));
				//使用LinearEyeDepth得到在观察空间下的深度值，这里需要注意的是Unity的观察空间中，摄像机正向对应着的z值
				//为负值，而为了得到深度值的正数表示，将原观察空间的深度值这里做了一个取反的操作
				float partZ=-i.viewPos.z;
				//因此这里得到当前平面的观察空间深度值后，取了反，与上面得到的结果对应
				float diff=min((abs(sceneZ-partZ))/_Threshold,1.0);
				//这里通过两者深度值的插值/阈值控制颜色插值运算的结果，深度值相差太大则是扫描平面自身颜色
				//而差值越小，则越接近高亮颜色
				finalColor=lerp(_HighLightingColor,_MainColor,diff)*tex2D(_MainTex,i.uv);

				return finalColor;
			}
			ENDCG
		}
	}
	FallBack "Transparent/VertexLit"
	}  

这里可以注意下设置的渲染队列和混合操作，还有一点需要说明的是，为什么最后计算的时候都是使用的观察空间的深度值，之前提到过，投影矩阵的变换过程是非线性的，因此需要转化到线性空间进行计算，所以这里选择了观察空间。       

实例效果：     
![](https://i.imgur.com/SRMt7lr.png)         
参数设置：     
![](https://i.imgur.com/hO1LvkL.png)      

**利用深度纹理实现运动模糊**         
之前实现的运动模糊是通过混合多张屏幕图像来模拟运动模糊。另一种实现运动模糊的方式是通过速度映射图，而且该方式应用更加广泛。速度映射图中存储每个像素的速度，使用该速度决定模糊的方向和大小。生成速度缓冲可以将场景中所有物体的速度渲染到一张纹理中，不过该方法需要修改场景中所有物体的Shader代码，使其添加计算速度的代码并输出到下一个渲染纹理中。    
有一种方法是通过深度纹理在片元着色器中为每个像素计算其在世界空间下的位置，该过程是通过当前视角x投影矩阵的逆矩阵对NDC下的顶点坐标进行变换得到。再将该位置与上一帧的视角x投影矩阵运算，得到该位置在上一帧的投影空间中的位置，计算上一帧该位置和当前帧的位置差，生成该像素的速度。这种方法的优点在于在一个屏幕后处理特效中就能完成整个效果模拟，缺点是在片元着色器中需要进行两次矩阵运算，消耗部分性能。 
  
完整代码：     

	public class Chapter13_MotionBlurWithDepthTexture : PostEffectsBase
	{
    public Shader motionShader;
    private Material motionBlurMaterial = null;

    public Material material
    {
        get
        {
            motionBlurMaterial = CheckShaderAndCreateMaterial(motionShader, motionBlurMaterial);
            return motionBlurMaterial;
        }
    }

    [Range(0.0f, 1.0f)]
    public float blurSize = 1.0f;
    //定义Camera类型变量，以获取该脚本所在的摄像机组件
    //得到摄像机位置，构建观察空间变换矩阵
    private Camera myCamera;
    public Camera camera
    {
        get
        {
            if (myCamera == null)
            {
                myCamera = GetComponent<Camera>();
            }      
            return myCamera;
        }
    }

    //定义一个保存上一帧视角*投影矩阵
    private Matrix4x4 previousViewProjectionMatrix;

    //定义摄像机状态，获取深度纹理
    void OnEable()
    {
        camera.depthTextureMode |=DepthTextureMode.Depth;
    }

    void OnRenderImage(RenderTexture src,RenderTexture dest)
    {
        if (material != null)
        {
            material.SetFloat("_BlurSize", blurSize);

            material.SetMatrix("_PreviousViewProjectionMatrix", previousViewProjectionMatrix);
            Matrix4x4 currentViewProjectionMatrix = camera.projectionMatrix*camera.worldToCameraMatrix;
            Matrix4x4 currentViewProjectionInverseMatrix = currentViewProjectionMatrix.inverse;
            material.SetMatrix("_CurrentViewProjectionInverseMatrix", currentViewProjectionInverseMatrix);
            previousViewProjectionMatrix = currentViewProjectionMatrix;

            Graphics.Blit(src, dest, material);
        }
        else
        {
            Graphics.Blit(src,dest);
        }
    }
	}

   
Shader代码：

	Shader "Custom/Chapter13_MotionBlurWithDepthTexture" {
	Properties{
		_MainTex("Maintex",2D)="white"{}
		_BlurSize("BlurSize",Float)=1.0
	}
	SubShader{
		CGINCLUDE
			
			#include "UnityCG.cginc"
			sampler2D _MainTex;
			half4 _MainTex_TexelSize;
			sampler2D _CameraDepthTexture;
			float4x4 _PreviousViewProjectionMatrix;
			float4x4 _CurrentViewProjectionInverseMatrix;
			half _BlurSize;

			struct v2f{
				float4 pos:POSITION;
				half2 uv:TEXCOORD0;
				half2 uv_depth:TEXCOORD1;
			};


			v2f vert(appdata_img v){
				v2f o;
				o.pos=UnityObjectToClipPos(v.vertex);
				o.uv=v.texcoord;
				o.uv_depth=v.texcoord;
				#if UNITY_UV_STARTS_AT_TOP
				if(_MainTex_TexelSize.y<0)
					o.uv_depth.y=1-o.uv_depth.y;
				#endif

				return o;
			}

			fixed4 frag(v2f i):SV_Target{
				//得到深度缓冲中该像素点的深度值
				float d=SAMPLE_DEPTH_TEXTURE(_CameraDepthTexture,i.uv_depth);
				//得到深度纹理映射前的深度值
				float4 H=float4(i.uv.x*2-1,i.uv.y*2-1,d*2-1,1);
				//通过转换矩阵得到顶点的世界空间的坐标值
				float4 D=mul(_CurrentViewProjectionInverseMatrix,H);
				float4 worldPos=D/D.w;

				float4 currentPos=H;
				float4 previousPos=mul(_PreviousViewProjectionMatrix,worldPos);
				previousPos/=previousPos.w;

				float2 velocity=(currentPos.xy-previousPos.xy)/2.0f;

				float2 uv=i.uv;
				float4 c=tex2D(_MainTex,uv);
				//得到像素速度后，对邻域像素进行采样，并使用BlurSize控制采样间隔
				//得到的像素点进行平均后，得到模糊效果
				uv+=velocity*_BlurSize;
				for(int it=1;it<3;it++,uv+=velocity*_BlurSize){
					float4 currentColor=tex2D(_MainTex,uv);
					c+=currentColor;
				}
				c/=3;
				return fixed4(c.rgb,1.0);
			}
		ENDCG

		Pass{
			ZTest Always 
			Cull Off
			ZWrite Off

			CGPROGRAM
				#pragma vertex vert
				#pragma fragment frag
			ENDCG
		}
	}

	FallBack Off
	}
实例效果：       
![](https://i.imgur.com/UlhdEqq.png)       
可以看到，这里的模糊效果是整个场景图像都有模糊效果，而之前采用帧缓冲图像进行混合所产生的模糊效果在某一方向上是模糊效果，并不是整个场景的图像都有模糊效果。但是如果在一个物体运动，而摄像机静止的场景中不会产生模糊效果，这是由于整个速度计算依赖于摄像机的视角变化。而上一节中实现的运动模糊，只要摄像机视野中物体发生了相对运动都可以产生模糊效果。

**全局雾效**       
雾效在游戏中是一种常见的特效。比如吃鸡里就有大雾天，Unity内置的雾效可以生成基于距离的线性或指数雾效。如果在顶点/片元着色器实现这样的雾效，需要在Shader中添加#pragma multi _compile _fog指令，同时需要使用相关内置宏，比如 UNITY _FOG _COORDS, UNITY _TRANSFER _FOG, UNITY _APPLY _FOG等。使用这种方法，需要为场景中所有物体添加渲染代码，操作比较繁琐。       
使用基于屏幕后处理的全局雾效，不需要为场景中所有的物体添加渲染代码，仅通过屏幕后处理就可以实现，而且可以模拟均匀雾效、基于距离的线性/指数雾效，基于高度的雾效。       
基于屏幕后处理的全局雾效关键在于**通过深度纹理重建每个像素在世界空间的位置** ，在上一节中，通过深度纹理的采样，反映射得到NDC坐标，再通过视角x投影坐标的逆矩阵运算得到世界空间下的位置，这样的实现需要在片元着色器中进行矩阵运算。      
有一种快速从深度纹理中重建世界坐标的方法。该方法首先对图像空间下的视椎体射线（从摄像机出发，指向图像上的某点）进行插值，这条射线记录了该像素点在世界空间下到摄像机的方向信息。将该射线和线性化后的视角空间下的深度值相乘，加上摄像机的世界位置，得到世界空间下的像素点的位置。得到该位置后，可以使用公式来模拟全局雾效。   

从深度纹理中计算世界坐标        
像素的世界坐标是通过在世界空间下像素相对于摄像机的偏移量+世界空间下摄像机的位置得到，用代码表示：       

	float4 worldPos=_WorldSpaceCameraPos+linearDepth*interpolatedRay;
	//_WorldSpaceCameraPos：摄像机世界空间位置坐标，由Unity内置变量即可得到
	//linearDepth：由深度纹理得到的线性深度值
	//interpolatedRay：由定点着色器输出插值得到的射线，包含了像素到摄像机的方向和距离信息      
       
interpolatedRay计算过程：      
interpolatedRay是对摄像机的近裁剪平面的四个顶点的特定向量的插值。先来计算这四个顶点的特定向量，包含了顶点到摄像机的距离和方向信息。计算过程中用到的图：       
![](https://i.imgur.com/KiApblE.png)         
先计算toTop和toRight向量：     

	halfHeight=Near*tan(FOV/2)       
	//Near:近裁剪平面距离      FOV:视椎体竖直方向的张角   
	
	toTop=camera.up x halfHeight
 	toRight=camera.right x halfHeight*aspect     
	//aspect:横纵比    
再得到TL、TR、BL、BR四个向量的值：    

	TL=camera.forward*Near+toTop-toRight;
	TR=camera.forward*Near+toTop+toRight;
	BL=camera.forward*Near-toTop-toRight;
	BR=camera.forward*Near-toTop+toRight;   
以上四个向量得到了关于近裁剪平面四个顶点到摄像机的方向和距离信息，由于采样得到的深度值z是相对与摄像机的垂直距离，因此还不能直接使用上述四个向量的单位向量与深度值相乘来得到该方向上具有深度值的距离，因此需要计算出具有深度值的某点到摄像机的直线距离，以TL点为例，由相似原理：    

	depth/dist=Near/|TL|  
	dist=depth*(|TL|/Near)    
由于四点对称，其他三个点都可以使用同一个因子与该方向的单位向量相乘得到能与对应深度值直接相乘的向量    

	scale=|TL|/Near  
	Ray_TL=TL/|TL|*scale
	Ray_TR=TR/|TR|*scale
	Ray_BL=BL/|BL|*scale
	Ray_BR=BR/|BR|*scale   
这样得到可以直接与深度值相乘前的特定向量。屏幕后处理使用特定材质渲染一个刚好填充整个屏幕的四边形面片。面片的4个顶点对应近裁剪平面的4个角。将上面的计算结果传递给顶点着色器，再由顶点着色器选择对应的向量，输出传递给片元着色器就得到了经过插值的interpolatedRay，然后计算像素点在世界空间的位置。  

雾的计算      
简单雾效的计算通过混合因子将雾的颜色与原始颜色混合。     

	float3 afterFog=f*fogColor+(1-f)*originalColor   
混合因子f有线性、指数和指数平方，给定距离为z：

	//线性 
	f=(Dmax-|z|)/(Dmax-Dmin)       //Dmax与Dmin为受影响的最大和最小距离 
	
	//指数 
	f=e^-(D*|z|)             //D为控制雾浓度参数

	//指数平方   
	f=e^-(D*|z|)^2       //D为控制雾效浓度参数        
可以采用线性雾效计算方式，计算基于高度的雾效：    
	
	f=(H_end-y)/(H_end-H_start)      //H_end和H_start为高度起始位置      

实例代码：     

	public class Chapter13_FogWithDepthTexture :PostEffectsBase
	{

    public Shader fogWithDepthShader;
    private Material fogMaterial;

    public Material material
    {
        get
        {
            fogMaterial = CheckShaderAndCreateMaterial(fogWithDepthShader, fogMaterial);
            return fogMaterial;
        }
    }

    private Camera myCamera;
    public Camera camera
    {
        get
        {
            if (myCamera == null)
                myCamera = GetComponent<Camera>();
            return myCamera;
        }
    }

    private Transform myTransform;

    public Transform cameraTransform
    {
        get
        {
            if (myTransform == null)
                myTransform = camera.transform;
            return myTransform;
        }
    }

    //定义模拟雾效的参数
    [Range(0.0f, 3.0f)]
    public float fogDensity = 1.0f;

    public Color fogColor = Color.white;
    public float fogStart = 0.0f;
    public float fogEnd = 2.0f;

    void OnEnable()
    {
        camera.depthTextureMode |=DepthTextureMode.Depth;
    }

    void OnRenderImage(RenderTexture src,RenderTexture dest)
    {
        if (material != null)
        {
            Matrix4x4 frustumCorners = Matrix4x4.identity;

            float fov = camera.fieldOfView;
            float near = camera.nearClipPlane;
            float far = camera.farClipPlane;
            float aspect = camera.aspect;

            float halfHeight = near*Mathf.Tan(fov*0.5f*Mathf.Deg2Rad);
            Vector3 toTop = cameraTransform.up*halfHeight;
            Vector3 toRight = cameraTransform.right*halfHeight*aspect;

            Vector3 topLeft = cameraTransform.forward*near + toTop - toRight;
            float scale = topLeft.magnitude/near;
            topLeft.Normalize();
            topLeft *= scale;

            Vector3 topRight = cameraTransform.forward*near + toTop + toRight;
            topRight.Normalize();
            topRight *= scale;

            Vector3 bottomLeft = cameraTransform.forward*near - toTop - toRight;
            bottomLeft.Normalize();
            bottomLeft *= scale;

            Vector3 bottomRight = cameraTransform.forward*near - toTop + toRight;
            bottomRight.Normalize();
            bottomRight *= scale;

            frustumCorners.SetRow(0, bottomLeft);
            frustumCorners.SetRow(1, bottomRight);
            frustumCorners.SetRow(2, topRight);
            frustumCorners.SetRow(3, topLeft);

            material.SetMatrix("_FrustumCornerRay", frustumCorners);
          
            material.SetFloat("_FogDensity", fogDensity);
            material.SetColor("_FogColor", fogColor);
            material.SetFloat("_FogStart", fogStart);
            material.SetFloat("_FogEnd", fogEnd);

            Graphics.Blit(src, dest, material);
        }
        else
        {
            Graphics.Blit(src,dest);
        }
    }
	}

Shader代码：       

	Shader "Custom/Chapter13_FogWithDepthTexture" {
	Properties{
		_MainTex("MainTex",2D)="white"{}
		_FogDensity("Fog Density",Float)=1.0
		_FogColor("FogColor",Color)=(1,1,1,1)
		_FogStart("Fog Start",Float)=0.0
		_FogEnd("FogEnd",Float)=2.0
	}
	SubShader{
		CGINCLUDE
			#include "UnityCG.cginc"

			sampler2D _MainTex;
			half4 _MainTex_TexelSize;
			sampler2D _CameraDepthTexture;
			half _FogDensity;
			fixed4 _FogColor;
			float _FogStart;
			float _FogEnd;  

			float4x4 _FrustumCornerRay;

			struct v2f{
				float4 pos:SV_POSITION;
				half2 uv:TEXCOORD0;
				half2 uv_depth:TEXCOORD1;
				float4 interpolatedRay:TEXCOORD2;
			};

			v2f vert(appdata_img v){
				v2f o;
				o.pos=UnityObjectToClipPos(v.vertex);
				o.uv=v.texcoord;
				o.uv_depth=v.texcoord;   
				#if UNITY_UV_STARTS_AT_TOP
				if(_MainTex_TexelSize.y<0)
					o.uv_depth.y=1-o.uv_depth.y;
				#endif

				int index=0;
				if(v.texcoord.x<0.5&&v.texcoord.y<0.5){
					index=0;
				}
				else if(v.texcoord.x>0.5&&v.texcoord.y<0.5){
					index=1;
				}
				if(v.texcoord.x>0.5&&v.texcoord.y>0.5){
					index=2;
				}
				else{
					index=3;
				}
				#if UNITY_UV_STARTS_AT_TOP
				if(_MainTex_TexelSize.y<0){
					index=3-index;
				}
				#endif
				o.interpolatedRay=_FrustumCornerRay[index];
				return o;
			}

			fixed4 frag(v2f i):SV_Target{
				float linearDepth=LinearEyeDepth(SAMPLE_DEPTH_TEXTURE(_CameraDepthTexture,i.uv_depth));
				float3 worldPos=_WorldSpaceCameraPos+linearDepth*i.interpolatedRay.xyz;

				float fogDensity=(_FogEnd-worldPos.y)/(_FogEnd-_FogStart);
				fogDensity=saturate(fogDensity*_FogDensity);

				fixed4 finalColor=tex2D(_MainTex,i.uv);
				finalColor.rgb=lerp(finalColor.rgb,_FogColor.rgb,fogDensity);
				return finalColor;
			}
		ENDCG

		Pass{
			ZTest Always ZWrite Off Cull Off
			
			CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag
			ENDCG
		}
	}
	FallBack Off
	}
实例效果：     
![](https://i.imgur.com/urcxVzw.png)        
参数设置：     
![](https://i.imgur.com/7THQMrx.png)       
实例效果中可以看到，在没有游戏对象的区域是不会产生雾效的，这是因为计算的基础是深度纹理，如果场景中为空，那么不会产生深度值，所以不会有雾效效果。    
关于最后的效果这里其实还有一点值得讨论，从截图中可以看到，距离摄像机较远的物体在同一高度下相较于离摄像机较近的物体实际上雾效效果会更浅，再来看代码中，这里的 worldPos 的计算：      

	worldPos=_WorldSpaceCameraPos+linearDepth*i.interpolatedRay.xyz;      
是将屏幕图像的像素点分成四个部分，将该像素点的线性深度值与对应区域的顶点的世界坐标相乘，因此同一高度下，深度值更大的像素点那么计算得到的worldPos.y的值会更大，颜色混合因子的结果也会更倾向于屏幕图像的原来颜色，因此雾效效果也就更加浅。

**利用深度纹理实现边缘检测**        
前面的章节中通过边缘检测算子（Sobel算子）实现了对屏幕图像进行边缘检测，进行描边效果。但是当产生的屏幕图像中的物体具有色彩丰富的纹理和阴影时，这种基于颜色的描边效果会使纹理和阴影也出现描边效果，比如        
![](https://i.imgur.com/mFNPQEV.png)           
这样物体的纹理和阴影也会被描上黑边，给人一种很脏的感觉。由于深度纹理仅仅保存了当前渲染物体的模型信息，如果在深度纹理上进行描边效果则会更加可靠和干净。   
之前使用的是Sobel算子进行边缘检测，下面要实现的效果采用Roberts算子，该算子的卷积核：         
![](https://i.imgur.com/wkXW76K.png)       
该算子计算对角的差值，然后再将结果相乘来作为判断是否存在边界的判定依据。实现过程中也采用该判断方式。    
代码实例：     
	
	public class Chapter13_EdgeDetectNormalsAndDepth : PostEffectsBase
	{
    public Shader edgeDetectShader;
    private Material edgeDetectMaterial;

    public Material material
    {
        get
        {
            edgeDetectMaterial = CheckShaderAndCreateMaterial(edgeDetectShader, edgeDetectMaterial);
            return edgeDetectMaterial;
        }
    }

    [Range(0.0f, 1.0f)]
    public float edgeOnly = 0.0f;

    public Color edgeColor = Color.black;
    public Color backgroundColor = Color.white;

    //用于控制深度和法线纹理的采样距离，值越大，描边越宽
    public float sampleDistance = 1.0f;
    //灵敏值，影响领域的深度值或法线值相差多少时，会被认为存在一条边界
    public float sensitivityDepth = 1.0f;
    public float sensitivityNormals = 1.0f;

    void OnEnable()
    {
        GetComponent<Camera>().depthTextureMode |=DepthTextureMode.DepthNormals;
    }

    //默认情况下，OnRenderImage()会在所有的不透明和透明Pass执行完成后调用，以便对所有物体都产生影响
    //当希望在不透明物体的Pass完成后立即调用，不对透明物体产生影响，可以添加[ImageEffectOpaque]特性实现
    [ImageEffectOpaque]
    void OnRenderImage(RenderTexture src,RenderTexture dest)
    {
        if (material != null)
        {
            material.SetFloat("_EdgeOnly",edgeOnly);
            material.SetColor("_EdgeColor",edgeColor);
            material.SetColor("_BackgroundColor",backgroundColor);
            material.SetFloat("_SampleDistance",sampleDistance);
            material.SetVector("_Sensitivity",new Vector4(sensitivityNormals,sensitivityDepth,0.0f,0.0f));

            Graphics.Blit(src,dest,material);
        }
        else
        {
            Graphics.Blit(src,dest);
        }
    }
	}       
Shader代码：       

	Shader "Custom/Chapter13_EdgeDetectWithNormalsAndDepth" {
	Properties{
		_MainTex("MainTex",2D)="white"{}
		_EdgeOnly("EdgeOnly",Float)=1.0
		_EdgeColor("EdgeColor",Color)=(0,0,0,1)
		_BackgroundColor("BackgroundColor",Color)=(1,1,1,1)
		_SampleDistance("SampleDistance",Float)=1.0
		_Sensitivity("Sensitivity",Vector)=(1,1,1,1)
	}
	SubShader{
		CGINCLUDE
		#include "UnityCG.cginc"
		
		sampler2D _MainTex;
		half4 _MainTex_TexelSize;
		fixed _EdgeOnly;
		fixed4 _EdgeColor;
		fixed4 _BackgroundColor;
		float _SampleDistance;
		half4 _Sensitivity;
		sampler2D _CameraDepthNormalsTexture;
		
		struct v2f{
			float4 pos:SV_POSITION;
			half2 uv[5]:TEXCOORD0;
		};		

		v2f vert(appdata_img v){
			v2f o;
			o.pos=UnityObjectToClipPos(v.vertex);
			half2 uv=v.texcoord;
			o.uv[0]=uv;

			#if UNITY_UV_STARTS_AT_TOP	
			if(_MainTex_TexelSize.y<0)
			uv.y=1-uv.y;
			#endif

			o.uv[1]=uv+_MainTex_TexelSize.xy*half2(1,1)*_SampleDistance;
			o.uv[2]=uv+_MainTex_TexelSize.xy*half2(-1,-1)*_SampleDistance;
			o.uv[3]=uv+_MainTex_TexelSize.xy*half2(-1,1)*_SampleDistance;
			o.uv[4]=uv+_MainTex_TexelSize.xy*half2(1,-1)*_SampleDistance;

			return o;
		}

		//检测给定的采样点之间是否存在一条分界线
		half CheckSame(half4 center,half4 sample){
			half2 centerNormal=center.xy;
			float centerDepth=DecodeFloatRG(center.zw);
			half2 sampleNormal=sample.xy;
			float sampleDepth=DecodeFloatRG(sample.zw);

			//检测两者法线值的差异，如果两者法线值足够接近，那么说明不存在分界线
			//法线值并没有进行解码，因为只需要知道两者的差异，不需要准确的解码值
			half2 diffNormal=abs(centerNormal-sampleNormal)*_Sensitivity.x;
			int isSameNormal=(diffNormal.x+diffNormal.y)<0.1;  

			//检测两者深度值值的差异，如果两者深度值足够接近，那么说明不存在分界线
			float diffDepth=abs(centerDepth-sampleDepth)*_Sensitivity.y;
			int isSameDepth=diffDepth<0.1*centerDepth;

			//只有两者的法线和深度值差异均在阈值范围内，才可以看做是不存在分界线
			return isSameNormal*isSameDepth?1.0:0.0;
		}

		fixed4 fragRobertsCrossDepthAndNormal(v2f i):SV_Target{
			//根据Roberts算子对深度法线图对应像素的周围像素采样
			half4 sample1=tex2D(_CameraDepthNormalsTexture,i.uv[1]);
			half4 sample2=tex2D(_CameraDepthNormalsTexture,i.uv[2]);
			half4 sample3=tex2D(_CameraDepthNormalsTexture,i.uv[3]);
			half4 sample4=tex2D(_CameraDepthNormalsTexture,i.uv[4]);

			half edge=1.0;
			edge*=CheckSame(sample1,sample2);
			edge*=CheckSame(sample3,sample4);

			//通过计算得到edge，设置非边界像素的颜色为原色还是背景色的着色方案
			fixed4 withEdgeColor=lerp(_EdgeColor,tex2D(_MainTex,i.uv[0]),edge);
			fixed4 withBackgroundColor=lerp(_EdgeColor,_BackgroundColor,edge);
			//使用_EdgeOnly控制非边界像素的颜色混合结果
			return lerp(withBackgroundColor,withEdgeColor,_EdgeOnly);
		}
		ENDCG
		Pass{
		ZTest Always ZWrite Off Cull Off

		CGPROGRAM
		#pragma vertex vert
		#pragma fragment fragRobertsCrossDepthAndNormal
		ENDCG
		}
	}
	FallBack Off
	}     
原图效果：       
![](https://i.imgur.com/UEsvtv4.png)         
颜色边缘检测效果：          
![](https://i.imgur.com/u70l6Iy.png)
深度法线纹理边缘检测效果：     
![](https://i.imgur.com/zFYrkKm.png)          
![](https://i.imgur.com/KlFidg7.png)         
可以看到，利用深度法线纹理的描边效果要比之前的干净许多，只有物体之间存在明显边界的地方才会被描边，不会收到物体自身纹理和阴影的影响，深度法线纹理边缘检测效果的第二张图为非边缘部分着色为背景色（白色）的效果，这样就完全变成线条速写的风格了。    
**值得讨论的地方**，回看Shader代码，为什么在做判定的时候，需要同时对深度和法线值的插值都做判断？原因在于同一个物体出现拐角的地方，两侧的法线差值会很大，而他们的深度值可能不会相差很大，而两个平行的平面之间，各自平面上的像素点法线的差值可能会很小，而深度值的差异可能会很大，所以只有法线和深度值的差异都很小时，才能认为他们在同一个面上，不存在边界。同时检测法线和深度值的差异不会发生漏掉边界的情况。


### 非真实感渲染 ###
**非真实渲染（Non-Photorealistic Rendering   NPR**常用于游戏中来形成特别的视觉效果和风格。
         
**卡通风格渲染**         
卡通风格渲染的游戏画面通常物体颜色分界明显，具有黑色的线条描边。卡通渲染的实现有多种方法，**基于色调的着色技术是其中之一**，实现过程中通过使用漫反射系数对一维纹理进行采样，控制漫反射色调。之前通过一张渐变纹理来控制漫反射颜色实现过卡通风格的渲染效果。卡通风格的高光效果往往是一块分界明显的色块，而物体边缘通常会有描边。本节中将通过基于模型的方式进行描边，而不是之前的屏幕后处理的方式。       

轮廓线渲染       
轮廓线的渲染是实时渲染中应用非常广泛的一种效果。目前常用的5种绘制模型轮廓线的方法：        

- **基于观察角度和表面法线的轮廓线渲染**      
使用视角方向和表面法线的点乘结果得到轮廓线信息。简单快速，一个Pass可以得到结果，局限性较大，不能得到比较满意的描边效果。   

- **正背面渲染**     
使用两个Pass，一个渲染背面，另一个渲染正面面片。快速有效，适用于大多数表面平滑的模型。      

- **基于图像处理的轮廓线**      
之前屏幕后处理以及利用深度纹理就是采用的这种方式。可以用于任何模型，但深度和法线变化很小的轮廓无法检测，比如紧贴的薄平面。     

- **基于轮廓边缘的轮廓线检测**     
通过计算得到精确的轮廓边，然后直接渲染，渲染出独特的风格。检测一条边是否为轮廓边，只需检测和这条边相邻的三角面片是否满足：   

		(N0*V>0)!=(N1*V>0)     
N0和N1分别是相邻面片的法向，这种方式由于是单帧提取轮廓，当帧数较低时，会出现帧与帧之间的跳跃性。    

- **最后一种是以上的综合渲染方法**       
首先找到精确的轮廓边，将模型和轮廓渲染到纹理，再通过图形处理的方式识别轮廓线，在图像空间下进行风格化渲染。   

下面使用正背面渲染的方式进行轮廓线的勾勒，之前的正背面渲染中，是直接将顶点在裁剪空间中向裁剪空间下的法线方向进行偏移。这里使用观察空间，在观察空间下对顶点进行观察空间下的法向偏移，区别在于观察空间是一个线性空间，尽管之前的效果也基本达到要求，但线性空间下的处理的结果会更加连贯。为了防止一些内凹的模型在使用正面剔除后发生背面遮挡正面的情况，先对顶点法线的z分量进行定值处理，再将法线归一化后进行扩张。这样可以使扩张后背面更加扁平化，降低遮挡正面面片的可能性。即：    

	viewNormal.z=-0.5;
	viewNormal=normalize(viewNormalize);
	viewPos=viewPos+viewNormal*_Outline;

卡通风格的高光通常表现为在模型上是一块块分界明显的色块。为了得到这种效果不再使用之前的高光计算模型。之前Blinn-Phong时，使用法线方向点乘视角和光照方向和的一半，再与_Gloss参数进行指数操作得到系数：   

	float spec=pow(max(0,dot(normal,halfDir)),_Gloss);     
对于卡通风格的高光反射光照模型，同样需要计算normal和halfDir的点乘结果，然后直接与一个阈值相比较，大于该阈返回1,小于该阈值返回0，以形成分界明显的色块界限：     

	float spec=dot(normal,halfDir);
	spec=step(threshold,spec);
CG的**step函数**实现和阈值比较返回0,1结果，第一个为参考值，第二个参数大于第一个参数，返回1，否则返回0。  
这种直接0,1的取值方式会在高光的边缘区域形成锯齿，因为由0,1突变。为了得到高光边缘叫平滑的效果，可以在边界处的小块区域内进行平滑处理。      

	float spec=dot(normal,halfDir);
	spec=lerp(0,1,smoothstep(-w,w,spec-threshold));      
使用CG的**smoothstep函数**，w是一个较小的值，当spec-threshold小于-w时，返回0，大于w时，返回1，否则在0,1之间进行插值。这样的效果是在[-w,w]区间，即高光反射边缘，进行0到1的平滑过渡，防止出现锯齿。w的值可以使用CG的**fwidth函数**得到邻域像素之间的近似导数（像素之间的变化程度）值。   

代码实例：

	Shader "Custom/Chapter14_ToonShading" {
	Properties{
		_MainTex("MainTex",2D)="white"{}
		_Color("Color",Color)=(1,1,1,1)
		_RampTex("Ramp",2D)="white"{}
		_Outline("Outline",Range(0,1))=0.1
		_OutlineColor("OutlineColor",Color)=(0,0,0,1)
		_Specular("SpecularColor",Color)=(1,1,1,1)
		_SpecularScale("Specular Scale",Range(0,0.1))=0.01
	}
	SubShader{
		Pass{
			NAME "OUTLINE"
			Cull Front
			CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag
			#include "UnityCG.cginc"

			fixed _Outline;
			fixed4 _OutlineColor;

			struct a2v{
				float4 vertex:POSITION;
				float3 normal:NORMAL;
			};
			struct v2f{
				float4 pos:SV_POSITION;
			};

			v2f vert(a2v v){
				v2f o;
				float4 pos=mul(UNITY_MATRIX_MV,v.vertex);
				float3 normal=mul((float3x3)UNITY_MATRIX_IT_MV,v.normal);
				normal.z=-0.5;
				pos=pos+float4(normalize(normal),0)*_Outline;

				o.pos=mul(UNITY_MATRIX_P,pos);
				return o;
			}

			fixed4 frag(v2f i):SV_Target{
				return fixed4(_OutlineColor.rgb,1);
			}
			ENDCG
		}

		Pass{
			Tags{"LightMode"="ForwardBase"}
			Cull Back
			CGPROGRAM
			#pragma vertex   vert
			#pragma fragment  frag
			#pragma multi_compile_fwdbase
			#include "Lighting.cginc"
			#include "UnityCG.cginc"
			#include "AutoLight.cginc" 

			sampler2D _MainTex;
			float4 _MainTex_ST;
			fixed4 _Color;
			sampler2D _RampTex;
			fixed4 _Specular;
			fixed _SpecularScale;

			struct a2v{
				float4 vertex:POSITION;
				float3 normal:NORMAL;
				float2 texcoord:TEXCOORD0;
			};

			struct v2f{
				float4 pos:SV_POSITION;
				float2 uv:TEXCOORD0;
				float3 worldNormal:TEXCOORD1;
				float3 worldPos:TEXCOORD2;
				SHADOW_COORDS(3)
			};

			v2f vert(a2v v){
				v2f o;
				o.pos=UnityObjectToClipPos(v.vertex);
				o.uv=TRANSFORM_TEX(v.texcoord,_MainTex);
				o.worldNormal=mul(v.normal,(float3x3)unity_WorldToObject);
				o.worldPos=mul(unity_ObjectToWorld,v.vertex).xyz;

				TRANSFER_SHADOW(o);

				return o;
			}

			fixed4 frag(v2f i):SV_Target{
				fixed3 worldNormal=normalize(i.worldNormal);
				fixed3 worldLightDir=normalize(UnityWorldSpaceLightDir(i.worldPos));
				fixed3 worldViewDir=normalize(UnityWorldSpaceViewDir(i.worldPos));
				fixed3 worldHalf=normalize(worldLightDir+worldViewDir);

				fixed4 c=tex2D(_MainTex,i.uv);
				fixed3 albedo=c.rgb*_Color.rgb;

				fixed3 ambient=UNITY_LIGHTMODEL_AMBIENT.xyz*albedo;

				UNITY_LIGHT_ATTENUATION(atten,i,i.worldPos);
				fixed diff=dot(worldNormal,worldLightDir);
				diff=(diff*0.5+0.5)*atten;

				fixed3 diffuse=_LightColor0.rgb*albedo*tex2D(_RampTex,float2(diff,diff)).rgb;

				fixed spec=dot(worldNormal,worldHalf);
				fixed w=fwidth(spec)*2.0;
				fixed3 specular=_Specular.rgb*lerp(0,1,smoothstep(-w,w,spec+_SpecularScale-1))*step(0.0001,_SpecularScale);
				//最后添加的step(0.0001,_SpecularScale);是为了控制当Specular为0时，不出现高光效果
				
				return fixed4(ambient+diffuse+specular,1.0);
			}
			ENDCG
		}
	}
	FallBack "Diffuse"
	//这里的回调需要注意包含能够处理阴影的特殊Pass
	}   
实例效果：       
![](https://i.imgur.com/Vam0Mro.png)   

**素描风格渲染**       
素描风格的渲染在非真实渲染中应用也比较流行。目前实时的素描风格渲染是通过使用提前生成的素描纹理来实现的。     
![](https://i.imgur.com/BH7j7Op.png)         
这些纹理组成**色调艺术映射**，纹理从左到右笔触逐渐增多，用于模拟不同光照效果下的漫反射效果，从上到下对应每张纹理的多级渐远纹理。       
下面的过程不考虑多级渐远纹理的生成，直接使用6张纹理进行渲染。首先在顶点着色器计算逐顶点光照，根据光照结果决定纹理的混合权重，然后传递给片元着色器，片元着色器根据权重混合6张纹理的采样结果。 

实例代码：   

	Shader "Custom/Chapter14_Hatching" {
	Properties{
		_Color("Color",Color)=(1,1,1,1)
		_TileFactor("Tile Factor",Float)=1
		_Outline("Outline",Range(0,1))=0.1
		_Hatch0("Hatch 0",2D)="white"{}
		_Hatch1("Hatch 1",2D)="white"{}
		_Hatch2("Hatch 2",2D)="white"{}
		_Hatch3("Hatch 3",2D)="white"{}
		_Hatch4("Hatch 4",2D)="white"{}
		_Hatch5("Hatch 5",2D)="white"{}

		//TileFactor为纹理的平铺系数，值越大，素描线条越密集
	}
	SubShader{
		Tags{"RenderType"="Opaque" "Queue"="Geometry"}
		UsePass "Custom/Chapter14_ToonShading/OUTLINE"  
		//素描风格往往也需要绘制轮廓线，使用之前的渲染轮廓Pass
		Pass{
			Tags{"LightMode"="ForwardBase"}
			CGPROGRAM
				#pragma vertex vert
				#pragma fragment frag

				#pragma multi_compile_fwdbase

				#include "UnityCG.cginc"
				#include "AutoLight.cginc"

				fixed4 _Color;
				float _TileFactor;
				fixed _Outline;
				sampler2D _Hatch0;
				float4 _Hatch0_ST;
				sampler2D _Hatch1;
				float4 _Hatch1_ST;
				sampler2D _Hatch2;
				float4 _Hatch2_ST;
				sampler2D _Hatch3;
				float4 _Hatch3_ST;
				sampler2D _Hatch4;
				float4 _Hatch4_ST;
				sampler2D _Hatch5;
				float4 _Hatch5_ST;

				struct a2v{
					float4 vertex:POSITION;
					float3 normal:NORMAL;
					half4 texcoord:TEXCOORD0;
				};

				struct v2f{
					float4 pos:SV_POSITION;
					float2 uv:TEXCOORD0;
					fixed3 hatchWeight0:TEXCOORD1;
					fixed3 hatchWeight1:TEXCOORD2;
					float3 worldPos:TEXCOORD3;

					SHADOW_COORDS(4)

					//6个权重值分别存储在2个float3类型变量中
				};

				v2f vert(a2v v){
					v2f o;
					o.pos=UnityObjectToClipPos(v.vertex);
					o.uv=v.texcoord.xy*_TileFactor;
					//_TileFactor用来控制素描线条的密集程度（TEX的WrapMode为Repeat）
					
					float3 worldLightDir=normalize(WorldSpaceLightDir(v.vertex));
					float3 worldNormal=UnityObjectToWorldNormal(v.normal);
					float3 diff=max(0,dot(worldLightDir,worldNormal));
					//这里的关键便是通过计算漫反射系数来区分采样权重，并将权重与不同密集程度的TEX相对应

					o.hatchWeight0=fixed3(0,0,0);
					o.hatchWeight1=fixed3(0,0,0);

					//使用世界空间下的光照方向和法线方向得到漫反射系数
					//初始化权重值，*7分为7个区间，并根据hatchFactor的值，为权重赋值
					float hatchFactor=diff*7;
					if(hatchFactor>6){
						//不做任何赋值，保持纯白
					}
					else if(hatchFactor>5.0){
						o.hatchWeight0.x=hatchFactor-5.0;
					}
					else if(hatchFactor>4.0){
						o.hatchWeight0.x=hatchFactor-4.0;
						o.hatchWeight0.y=1.0-o.hatchWeight0.x;
					}
					else if(hatchFactor>3.0){
						o.hatchWeight0.y=hatchFactor-3.0;
						o.hatchWeight0.z=1.0-o.hatchWeight0.y;
					}
					else if(hatchFactor>2.0){
						o.hatchWeight1.x=hatchFactor-2.0;
					}
					else if(hatchFactor>1.0){
						o.hatchWeight1.x=hatchFactor-1.0;
						o.hatchWeight1.y=1.0-o.hatchWeight1.x;
					}
					else{
						o.hatchWeight1.y=hatchFactor;
						o.hatchWeight1.z=1.0-o.hatchWeight1.y;
					}

					o.worldPos=mul(unity_ObjectToWorld,v.vertex).xyz;

					TRANSFER_SHADOW(o)

					return o;
				}
				fixed4 frag(v2f i):SV_Target{
					fixed4 hatchTex0=tex2D(_Hatch0,i.uv)*i.hatchWeight0.x;
					fixed4 hatchTex1=tex2D(_Hatch1,i.uv)*i.hatchWeight0.y;
					fixed4 hatchTex2=tex2D(_Hatch2,i.uv)*i.hatchWeight0.z;
					fixed4 hatchTex3=tex2D(_Hatch3,i.uv)*i.hatchWeight1.x;
					fixed4 hatchTex4=tex2D(_Hatch4,i.uv)*i.hatchWeight1.y;
					fixed4 hatchTex5=tex2D(_Hatch5,i.uv)*i.hatchWeight1.z;
					//得到6张素描纹理采样结果，并乘以对应的权重
					fixed4 whiteColor=fixed4(1,1,1,1)*(1.0-i.hatchWeight0.x-i.hatchWeight0.y-i.hatchWeight0.z-i.hatchWeight1.x-i.hatchWeight1.y-i.hatchWeight1.z);
					fixed4 hatchColor=hatchTex0+hatchTex1+hatchTex2+hatchTex3+hatchTex4+hatchTex5+whiteColor;
					//计算纯白的占比程度，素描风格中会有留白，并且高光部分也是白色
					UNITY_LIGHT_ATTENUATION(atten,i,i.worldPos);

					return fixed4(hatchColor.rgb*_Color.rgb*atten,1.0);
					//混合各个颜色，并与衰减和模型颜色相乘得到最终颜色
				}
			ENDCG
		}
	}
	FallBack "Diffsue"
	}      
实例效果：       
![](https://i.imgur.com/6XyaRpJ.png)           

### 使用噪声 ###
噪声的使用会使结果充满随机性，从而表现效果更加自然。     
**消融效果**               
消融效果常在游戏中角色死亡，物品烧毁中被使用。消融效果往往从不同区域开始，并向看似随机的方向进行扩张，最后整个物体都消失不见。消融效果可以通过噪声纹理+透明度测试去实现。使用对噪声纹理进行采样的结果与某个控制消融效果的阈值进行比较，小于阈值，使用**clip函数**将该像素点裁剪掉，对应被烧毁的部分，而边缘部分的烧焦效果则将两种颜色进行混合，在使用pow函数处理，得到最终边缘烧毁颜色。    
实例代码：    

	Shader "Custom/Chapter15_Dissolve" {
	Properties{
		_BurnAmount("Burn Amount",Range(0.0,1.0))=0.0
		_LineWidth("Burn Line Width",Range(0.0,0.2))=0.1
		_MainTex("MainTex",2D)="white"{}
		_BumpMap("Bump Map",2D)="bump"{}
		_BurnFirstColor("Burn First Color",Color)=(1,0,0,1)
		_BurnSecondColor("Burn Second Color",Color)=(1,0,0,1)
		_BurnMap("Burn Map",2D)="white"{}

		//_BurnAmount 用于控制消融的程度，值为0时为正常效果，值为1时物体完全消融
		//_LineWidth 控制模拟烧焦效果的线宽
		//_BurnFirstColor 和 _BurnSecondColor 对应火焰边缘两种颜色
		//_BurnMap 对应消融效果的噪声纹理
		}
		SubShader{
			Pass{
				Tags{"LightMode"="ForwardBase"}
				Cull Off  
				//关闭剔除功能，也就是说模型的正面和背面对会被渲染出来
				//这是由于消融效果会显示出物体内部的结构，因此背面的剔除需要关闭
				CGPROGRAM
					#pragma vertex vert
					#pragma fragment frag

					#include "Lighting.cginc"
					#include "AutoLight.cginc"
					#pragma multi_compile_fwdbase

					fixed _BurnAmount;
					fixed _LineWidth;
					sampler2D _MainTex;
					float4 _MainTex_ST;
					sampler2D _BumpMap;
					float4 _BumpMap_ST;
					fixed4 _BurnFirstColor;
					fixed4 _BurnSecondColor;
					sampler2D _BurnMap;
					float4 _BurnMap_ST;

					struct a2v{
						float4 vertex:POSITION;
						float3 normal:NORMAL;
						float4 tangent:TANGENT;
						float4 texcoord:TEXCOORD0;
					};

					struct v2f{
						float4 pos:SV_POSITION;
						float2 uvMainTex:TEXCOORD0;
						float2 uvBumpMap:TEXCOORD1;
						float2 uvBurnMap:TEXCOORD2;
						float3 lightDir:TEXCOORD3;
						float3 worldPos:TEXCOORD4;

						SHADOW_COORDS(5)
					};

					v2f vert(a2v v){
						v2f o;
						o.pos=UnityObjectToClipPos(v.vertex);

						o.uvMainTex=TRANSFORM_TEX(v.texcoord,_MainTex);
						o.uvBumpMap=TRANSFORM_TEX(v.texcoord,_BumpMap);
						o.uvBurnMap=TRANSFORM_TEX(v.texcoord,_BurnMap);

						TANGENT_SPACE_ROTATION;
						o.lightDir=mul(rotation,ObjSpaceLightDir(v.vertex)).xyz;
						o.worldPos=mul(unity_ObjectToWorld,v.vertex).xyz;
						TRANSFER_SHADOW(o);

						return o;
					}

					fixed4 frag(v2f i):SV_Target{
						fixed3 burn=tex2D(_BurnMap,i.uvBurnMap);
						clip(burn.r-_BurnAmount);
						//通过噪声纹理的采样值与阈值比较来控制像素的剔除
						float3 tangentLightDir=normalize(i.lightDir);
						fixed3 tangentNormal=UnpackNormal(tex2D(_BumpMap,i.uvBumpMap));

						fixed3 albedo=tex2D(_MainTex,i.uvMainTex).rgb;
						fixed3 ambient=UNITY_LIGHTMODEL_AMBIENT.xyz*albedo;
						fixed3 diffuse=_LightColor0.rgb*albedo*max(0,dot(tangentLightDir,tangentNormal));

						fixed t=1-smoothstep(0.0,_LineWidth,burn.r-_BurnAmount);
						fixed3 burnColor=lerp(_BurnFirstColor,_BurnSecondColor,t);
						burnColor=pow(burnColor,5);
						//当t的值为1时，表明该像素点在消融边缘，t为0时，则为正常颜色
						//中间部分的插值模拟烧焦的痕迹，并使用pow函数

						UNITY_LIGHT_ATTENUATION(atten,i,i.worldPos);
						fixed3 finalColor=lerp(ambient+diffuse*atten,burnColor,t*step(0.0001,_BurnAmount));
						
						return fixed4(finalColor,1.0);
					}
				ENDCG
			}

			//这里需要定义一个投射阴影的Shader，避免被剔除的区域投射阴影
			Pass{
				Tags{"LightMode"="ShadowCaster"}	
				CGPROGRAM
				#pragma vertex vert
				#pragma fragment frag
				#pragma multi_compile_shadowcaster
				//用于投射阴影的Pass的LightMode要设置为ShadowCaster
				//同时需要使用 #pragma multi_compile_shadowcaster 编译指令
				#include "UnityCG.cginc"

				fixed _BurnAmount;
				sampler2D _BurnMap;
				float4 _BurnMap_ST;

				struct v2f{
					V2F_SHADOW_CASTER;
					//利用V2F_SHADOW_CASTER定义阴影投射需要定义的变量
					float2 uvBurnMap:TEXCOORD1;
				};

				v2f vert(appdata_base v){
					v2f o;
					TRANSFER_SHADOW_CASTER_NORMALOFFSET(o)
					//使用TRANSFER_SHADOW_CASTER_NORMALOFFSET(o)填充
					//V2F_SHADOW_CASTER在背后声明的部分变量，由Unity去完成
					o.uvBurnMap=TRANSFORM_TEX(v.texcoord,_BurnMap);
					return o;
				}

				fixed4 frag(v2f i):SV_Target{
					fixed3 burn=tex2D(_BurnMap,i.uvBurnMap).rgb;
					clip(burn.r-_BurnAmount);
					SHADOW_CASTER_FRAGMENT(i)
					//根据噪声纹理将消融的像素剔除，剩下的通过内置宏由Unity去完成对应的阴影投射计算
				}
				ENDCG
			}
		}
		FallBack "DIFFUSE"
	}

实例效果：     
![](https://i.imgur.com/j7fmFFC.png)        
通过设置_BurnAmount的值随时间变化可以得到动态消融效果  

**使用噪声实现扰动的动态效果**     
扰动效果类似于动漫中人物不动，身后的背景流动，比如说       
![](https://i.imgur.com/h9gyR3c.jpg)      
王也小哥手中的两团气团来回流动的效果
扰动效果的实现关键点在于：      
使用遮罩纹理区别流动和非流动区域，      
使用噪声纹理来控制流动过程的随机性，    
这里使用的源纹理：      
![](https://i.imgur.com/orm8wZm.jpg)       
遮罩纹理一般是对源纹理做处理：      
![](https://i.imgur.com/qMOdlKC.png)        
黑色区域即为不受扰动影响的区域，而红色部分为需要应用扰动效果的区域，因为是通过采样遮罩纹理的值去控制，所以黑色部分采样值乘以处理结果的像素颜色后得0，这样就不受扰动效果的影响，而其他部分为红色是由于采样的通道定为R，当然也可以使用其他通道。      
噪声纹理可以通用，WarpMode设为Repeat就可以     
主要思想是：       
通过噪声纹理和时间变量得到对源纹理UV采样的动态偏移值，       
通过对遮罩纹理的采样结果*动态偏移值来控制受扰动的区域       
实例代码：       

	Shader "Custom/Disturbance" {
	Properties{
		_MainTex("MainTex",2D)="white"{}
		_Mask("Mask",2D)="white"{}
		_Noise("Noise",2D)="white"{}
		_NoiseSpeedX("NoiseSpeedX",Range(0.0,3.0))=1.0
		_NoiseSpeedY("NoiseSpeedY",Range(0.0,3.0))=1.0
		_NoiseIntensity("NoiseIntensity",Range(0.0,3.0))=1.0
	}
	SubShader{
		Pass{
			Tags{"LightMode"="ForwardBase"}
			CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag
			#include "UnityCG.cginc"

			sampler2D _MainTex;
			float4 _MainTex_ST;
			sampler2D _Mask;
			float4 _Mask_ST;
			sampler2D _Noise;
			float4 _Noise_ST;
			half _NoiseSpeedX;
			half _NoiseSpeedY;
			half _NoiseIntensity;

			struct a2v{
				float4 vertex:POSITION;
				float2 texcoord:TEXCOORD0;
			};

			struct v2f{
				float4 pos:SV_POSITION;
				float2 uv1:TEXCOORD0;
				float2 uv2:TEXCOORD1;
			};

			v2f vert(a2v v){
				v2f o;
				o.pos=UnityObjectToClipPos(v.vertex);
				o.uv1=TRANSFORM_TEX(v.texcoord,_MainTex);
				o.uv2=TRANSFORM_TEX(v.texcoord,_Noise);

				return o;
			}

			//噪声纹理采样得到随机值并映射
			fixed2 SamplerFromNoise(float2 uv){
				float2 newUv=uv*_Noise_ST.xy+_Noise_ST.zw;
				fixed4 noiseColor=tex2D(_Noise,newUv);
				noiseColor=(noiseColor*2-1)*0.05;
				return noiseColor;
			}

			fixed4 frag(v2f i):SV_Target{
				//遮罩纹理采样
				fixed4 mask=tex2D(_Mask,i.uv1);
				//时间变量(t/20,t,2t,3t)
				float2 time=float2(_Time.x,_Time.x);
				//计算噪声偏移
				fixed2 noiseOffset=fixed2(0,0);
				noiseOffset=SamplerFromNoise(i.uv2+time*float2(_NoiseSpeedX,_NoiseSpeedY));
				//主纹理采样,使用噪声纹理控制扰动区域
				fixed4 mainColor=tex2D(_MainTex,i.uv1+noiseOffset*_NoiseIntensity*mask.r);
				
				return mainColor;
			}
			ENDCG
		}
	}
	FallBack "Diffuse"
	}           
实例效果：     
扰动效果前：       
![](https://i.imgur.com/Q32kP6y.jpg)        
扰动效果：      
![](https://i.imgur.com/xpjrWXf.jpg)        
这个效果时动态的，因为使用了时间变量，当然可以使用粒子效果和多层次扰动实现更加精细的效果。   

**水波效果**          
噪声纹理也可以应用在实时水面的模拟中。通常将噪声纹理用作高度图，不断修改水面的法线方向。通过使用时间变量来对噪声纹理进行采样以得到不断流动的效果，得到法线信息后，再进行正常的反射+折射的计算，得到动态水面效果。在之前实现玻璃效果时，通过使用一个Cubemap纹理作为环境纹理以采样得到反射效果，折射效果的模拟是使用GrabPass抓取当前的渲染纹理，使用切线空间下的法线方向对采样坐标进行偏移，并使用该坐标对屏幕图像进行采样，近似模拟折射效果。在实现水波的过程中，将使用噪声纹理得到法线，并结合时间变量实现动态效果。得到折射与反射值后，其混合值的混合系数通过公式得到： 

	fresnel=pow(1-max(0,v*n),4)            
观察方向与法线方向夹角越小，fresnel值越小，反射越弱，折射越强，在水面非常清澈的情况下。  

实例代码：     

	Shader "Custom/Chapter15_WaterWave" {
	Properties{
		_Color("MainColor",Color)=(1,1,1,1)   //水面颜色
		_MainTex("MainTex",2D)="white"{}    //水面纹理
		_WaveMap("WaveMap",2D)="bump"{}
		_CubeMap("CubeMap",Cube)="_Skybox"{}
		_WaveXSpeed("WaveXSpeed",Range(-0.1,0.1))=0.01
		_WaveYSpeed("WaveYSpeed",Range(-0.1,0.1))=0.01
		_Distortion("Distortion",Range(0,100))=10   //控制图像扭曲程度
	}

	SubShader{
		Tags{"Queue"="Transparent" "RenderType"="Opaque"}
		//这里设置渲染队列为Transparent,是为了保证渲染该物体前，其他所有不透明物体已经被渲染
		//而设置渲染类型则是为了使用摄像机的深度和法线纹理时，物体被正确渲染（会使用着色器替换技术）
		GrabPass{"_RefractionTex"}
		Pass{
			Tags{"LightMode"="ForwardBase"}
			CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag
			#include "UnityCG.cginc"

			fixed4 _Color;
			sampler2D _MainTex;
			float4 _MainTex_ST;
			sampler2D _WaveMap;
			float4 _WaveMap_ST;
			samplerCUBE _CubeMap;
			fixed _WaveXSpeed;
			fixed _WaveYSpeed;
			float _Distortion;
			sampler2D _RefractionTex;
			float4 _RefractionTex_TexelSize;

			struct a2v{
				float4 vertex:POSITION;
				float3 normal:NORMAL;
				float4 tangent:TANGENT;
				float4 texcoord:TEXCOORD0;
			};

			struct v2f{
				float4 pos:SV_POSITION;
				float4 srcPos:TEXCOORD0;
				float4 uv:TEXCOORD1;
				float4 TtoW0:TEXCOORD2;
				float4 TtoW1:TEXCOORD3;
				float4 TtoW2:TEXCOORD4;
			};

			v2f vert(a2v v){
				v2f o;
				o.pos=UnityObjectToClipPos(v.vertex);    
				o.srcPos=ComputeGrabScreenPos(o.pos);
				o.uv.xy=TRANSFORM_TEX(v.texcoord,_MainTex);
				o.uv.zw=TRANSFORM_TEX(v.texcoord,_WaveMap);

				float3 worldPos=mul(unity_ObjectToWorld,v.vertex);
				float3 worldNormal=UnityObjectToWorldNormal(v.normal);
				float3 worldTangent=UnityObjectToWorldDir(v.tangent.xyz);
				float3 worldBinormal=cross(worldNormal,worldTangent)*v.tangent.w;

				o.TtoW0=float4(worldTangent.x,worldBinormal.x,worldNormal.x,worldPos.x);
				o.TtoW1=float4(worldTangent.y,worldBinormal.y,worldNormal.y,worldPos.y);
				o.TtoW2=float4(worldTangent.z,worldBinormal.z,worldNormal.z,worldPos.z);

				return o;
			}

			fixed4 frag(v2f i):SV_Target{
				float3 worldPos=float3(i.TtoW0.w,i.TtoW1.w,i.TtoW2.w);
				fixed3 viewDir=normalize(UnityWorldSpaceViewDir(worldPos));
				float2 speed=_Time.y*float2(_WaveXSpeed,_WaveYSpeed); //_Time(t/20,t,2t,3t)

				//切线空间下的法线采样并反解
				fixed3 bump1=UnpackNormal(tex2D(_WaveMap,i.uv.zw+speed)).rgb;
				fixed3 bump2=UnpackNormal(tex2D(_WaveMap,i.uv.zw-speed)).rgb;
				fixed3 bump=normalize(bump1+bump2);
				//两次对法线纹理采样，模拟两层水面交叉效果

				//使用切线空间下的法线进行偏移，该空间下的法线可以反映顶点局部空间下的法线方向
				float2 offset=bump.xy*_Distortion*_RefractionTex_TexelSize;
				i.srcPos.xy=offset*i.srcPos.z+i.srcPos.xy; //使用i.srcPos.z与偏移相乘，模拟深度越大，折射越强的效果
				fixed3 refrCol=tex2D(_RefractionTex,i.srcPos.xy/i.srcPos.w).rgb;

				//进行矩阵变换，得到世界空间下的法线方向
				bump=normalize(half3(dot(i.TtoW0.xyz,bump),dot(i.TtoW1.xyz,bump),dot(i.TtoW2.xyz,bump)));
				fixed3 texColor=tex2D(_MainTex,i.uv.xy+speed);
				fixed3 reflDir=reflect(-viewDir,bump);
				fixed3 reflCol=texCUBE(_CubeMap,reflDir).rgb*texColor*_Color;

				fixed3 fresnel=pow(1-max(0,dot(viewDir,bump)),4);
				fixed3 finalColor=refrCol*(1-fresnel)+reflCol*fresnel;

				return fixed4(finalColor,1);
			}
			ENDCG
		}
	}
	FallBack "Transparent"
	}    
实例效果：      
![](https://i.imgur.com/AbvgFsZ.png)                  

**通过噪声纹理实现不均匀雾效**           
之前通过深度纹理实现基于屏幕后处理的雾效，由深度纹理重建像素的世界空间下的位置，使用基于高度的公式计算全局雾效系数，然后使用该系数混合雾的颜色和原屏幕颜色。这种雾效是一种均匀的雾效，通过噪声纹理可以实现不均匀雾效。

实例代码：   

	public class Chapter15_FogWithNoise : PostEffectsBase
	{
    public Shader fogShader;

    private Material fogMaterial;
    public Material material
    {
        get
        {
            fogMaterial = CheckShaderAndCreateMaterial(fogShader, fogMaterial);
            return fogMaterial;
        }
    }

    private Camera myCamera;
    public Camera camera
    {
        get
        {
            if (myCamera == null)
            {
                myCamera = GetComponent<Camera>();
            }
            return myCamera;
        }
    }

    private Transform myCameraTransform;
    public Transform cameraTransform
    {
        get
        {
            if (myCameraTransform == null)
            {
                myCameraTransform = camera.transform;
            }
            return myCameraTransform;
        }
    }

    [Range(0.1f, 3.0f)]
    public float fogDensity = 1.0f;

    public Color fogColor = Color.white;
    public float fogStart = 0.0f;
    public float fogEnd = 2.0f;

    public Texture noiseTexture;

    [Range(-0.5f, 0.5f)]
    public float fogXSpeed = 0.1f;
    [Range(-0.5f, 0.5f)]
    public float fogYSpeed = 0.1f;

    [Range(0.0f, 3.0f)]
    public float noiseAmount = 1.0f;

    void OnEnable()
    {
        camera.depthTextureMode |=DepthTextureMode.Depth;
    }

    void OnRenderImage(RenderTexture src, RenderTexture dest)
    {
        if (material != null)
        {
            Matrix4x4 frustumCornors = Matrix4x4.identity;

            float fov = camera.fieldOfView;
            float near = camera.nearClipPlane;
            float aspect = camera.aspect;

            float halfHeight = near * Mathf.Tan(fov * 0.5f * Mathf.Deg2Rad);
            Vector3 toRight = cameraTransform.right*halfHeight*aspect;
            Vector3 toTop = cameraTransform.up*halfHeight;

            Vector3 topLeft = camera.transform.forward*near + toTop - toRight;
            float scale = topLeft.magnitude/near;

            Vector3 topRight = camera.transform.forward*near + toTop + toRight;
            topRight.Normalize();
            topRight *= scale;

            Vector3 bottomLeft = camera.transform.forward*near - toTop - toRight;
            bottomLeft.Normalize();
            bottomLeft *= scale;

            Vector3 bottomRight = camera.transform.forward*near - toTop + toRight;
            bottomRight.Normalize();
            bottomRight *= scale;

            frustumCornors.SetRow(0,bottomLeft);
            frustumCornors.SetRow(1,bottomRight);
            frustumCornors.SetRow(2,topRight);
            frustumCornors.SetRow(3,topLeft);

            material.SetMatrix("_FrustumCornorsRay",frustumCornors);
            material.SetFloat("_FogDensity",fogDensity);
            material.SetColor("_FogColor",fogColor);
            material.SetFloat("_FogStart",fogStart);
            material.SetFloat("_FogEnd",fogEnd);

            material.SetTexture("_NoiseTex",noiseTexture);
            material.SetFloat("_FogXSpeed",fogXSpeed);
            material.SetFloat("_FogYSpeed",fogYSpeed);
            material.SetFloat("_NoiseAmount",noiseAmount);

            Graphics.Blit(src,dest,material);
        }
        else
        {
            Graphics.Blit(src,dest);
        }
    }
	}

Shader代码： 

	Shader "Custom/Chapter15_FogWithNoise" {
	Properties{
		_MainTex("MainTex",2D)="white"{}
		_FogDensity("FogDensity",Float)=1.0
		_FogColor("FogColor",Color)=(1,1,1,1)
		_FogStart("FogStart",Float)=0.0
		_FogEnd("FogEnd",Float)=1.0
		_NoiseTex("NoiseTex",2D)="white"{}
		_FogXSpeed("FogXPeed",Float)=0.1
		_FogYSpeed("FogYSpeed",Float)=0.1
		_NoiseAmount("NoiseAmount",Float)=1
	}

	SubShader{
		CGINCLUDE
		#include "UnityCG.cginc"

			float4x4 _FrustumCornorsRay;
			sampler2D _MainTex;
			half4 _MainTex_TexelSize;
			sampler2D _CameraDepthTexture;
			half _FogDensity;
			fixed4 _FogColor;
			float _FogStart;
			float _FogEnd;
			sampler2D _NoiseTex;
			float _FogXSpeed;
			float _FogYSpeed;
			float _NoiseAmount;

			struct v2f{
				float4 pos:POSITION;
				half2 uv:TEXCOORD0;
				half2 uv_depth:TEXCOORD1;
				float4 interpolatedRay:TEXCOORD2;
			};

			v2f vert(appdata_img v){
				v2f o;
				o.pos=UnityObjectToClipPos(v.vertex);
				o.uv=v.texcoord;
				o.uv_depth=v.texcoord;

				#if UNITY_UV_STARTS_AT_TOP
				if(_MainTex_TexelSize.y<0)
					o.uv_depth.y=1-o.uv_depth.y;
				#endif

				int index=0;
				if(v.texcoord.x<0.5&&v.texcoord.y<0.5){
					index=0;
				}
				if(v.texcoord.x>0.5&&v.texcoord.y<0.5){
					index=1;
				}
				if(v.texcoord.x>0.5&&v.texcoord.y>0.5){
					index=2;
				}
				else{
					index=3;
				}

				#if UNITY_UV_STARTS_AT_TOP
					if(_MainTex_TexelSize.y<0)
					index=3-index;
				#endif

				o.interpolatedRay=_FrustumCornorsRay[index];
				return o;
			}

			fixed4 frag(v2f i):SV_Target{
				float linearDepth=LinearEyeDepth(SAMPLE_DEPTH_TEXTURE(_CameraDepthTexture,i.uv_depth));
				float3 worldPos=_WorldSpaceCameraPos+linearDepth*i.interpolatedRay.xyz;

				float2 speed=_Time.y*float2(_FogXSpeed,_FogYSpeed);
				float noise=(tex2D(_NoiseTex,i.uv+speed).r-0.5)*_NoiseAmount;

				float fogDensity=(_FogEnd-worldPos.y)/(_FogEnd-_FogStart);
				fogDensity=saturate(fogDensity*_FogDensity*(1+noise));

				fixed4 finalColor=tex2D(_MainTex,i.uv);
				finalColor.rgb=lerp(finalColor.rgb,_FogColor.rgb,fogDensity);

				return finalColor;
			}

		ENDCG

		Pass{
			CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag
			ENDCG
		}
	}
	FallBack Off
	}      
实例效果：       
![](https://i.imgur.com/t14XzPS.png)             
由于在采样时加入时间变量，因此雾效会有动态效果，加上噪声纹理来控制雾效密度后，可以得到非均匀的雾效效果。

### 使用顶点投射的方法生成实时阴影 ###
在喵神的专栏里看到了这个实现方法：       
[https://zhuanlan.zhihu.com/myashader](https://zhuanlan.zhihu.com/myashader)            
自己试着写下看看效果，完整的计算思路大家可以参考喵神的专栏，主要思想是在顶点着色器内根据光源，顶点和地面的位置关系，利用相似原理计算投影点,即：       
![](https://i.imgur.com/AtJzqLO.png)           

Shader完整代码：        

	Shader "Custom/VertShadow" {
	Properties {
		_Color("Color", Color) = (1,1,1,1)
		_MainTex("Albedo (RGB)", 2D) = "white" {}
		_BumpTex("BumpTex",2D)="bump"{}
		_BumpScale("BumpScale",Float)=1.0
		_Specular("Specular",Color)=(1,1,1,1)
		_Gloss("Gloss",Range(8.0,256))=20

		_LightDir("LightPos&PlaneHeight",vector)=(0,0,0,0)  //这里使用Float4类型变量存储场景中的光源位置和地面高度
		_ShadowColor("ShadowColor",Color)=(0,0,0,1)
		_ShadowFalloff("ShadowFalloff",Float)=1.0
	}
	SubShader {
	    //第一个Pass正常渲染模型
		Pass{
			Tags{"LightMode"="ForwardBase"}

			CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag
			#include "Lighting.cginc"

			fixed4 _Color;
			sampler2D _MainTex;
			float4 _MainTex_ST;
			sampler2D _BumpTex;
			float4 _BumpTex_ST;
			float _BumpScale;
			fixed4 _Specular;
			float _Gloss;

			struct a2v{
				float4 vertex:POSITION;
				float3 normal:NORMAL;
				float4 tangent:TANGENT;
				float4 texcoord:TEXCOORD0;
			};

			struct v2f{
				float4 pos:SV_POSITION;
				float4 uv:TEXCOORD0;
				float3 lightDir:TEXCOORD1;
				float3 viewDir:TEXCOORD2;
			};

			v2f vert(a2v v){
				v2f o;
				o.pos=UnityObjectToClipPos(v.vertex);
				o.uv.xy=TRANSFORM_TEX(v.texcoord,_MainTex);
				o.uv.zw=TRANSFORM_TEX(v.texcoord,_BumpTex);

				float3 biNormal=cross(normalize(v.normal),normalize(v.tangent.xyz))*v.tangent.w;
				float3x3 rotation=float3x3(v.tangent.xyz,biNormal,v.normal);

				o.lightDir=mul(rotation,ObjSpaceLightDir(v.vertex).xyz);
				o.viewDir=mul(rotation,ObjSpaceViewDir(v.vertex).xyz);

				return o;
			}

			fixed4 frag(v2f i):SV_Target{
				fixed3 tangentLightDir=normalize(i.lightDir);
				fixed3 tangentViewDir=normalize(i.viewDir);

				fixed3 tangentNormal=UnpackNormal(tex2D(_BumpTex,i.uv.zw));
				tangentNormal.xy*=_BumpScale;
				tangentNormal.z=sqrt(1.0-saturate(dot(tangentNormal.xy,tangentNormal.xy)));

				fixed3 albedo=_Color.rgb*tex2D(_MainTex,i.uv.xy);
				fixed3 ambient=UNITY_LIGHTMODEL_AMBIENT.xyz*albedo;
				fixed3 diffuse = _LightColor0.rgb*albedo*max(0,dot(tangentNormal,tangentLightDir));
				fixed3 halfDir=normalize(tangentLightDir+tangentViewDir);
				fixed3 specular = _LightColor0.rgb*_Specular.rgb*pow(max(0,dot(tangentNormal,halfDir)),_Gloss);

				return fixed4(ambient+diffuse+specular,1.0);
			}
			ENDCG
		}
					//第二个Pass计算世界空间下顶点的的阴影投影点
			Pass{
				//设置透明混合模式
				Blend SrcAlpha OneMinusSrcAlpha
				//关闭深度写入
				ZWrite off
				//深度偏移防止阴影与地面穿插
				Offset -1,0

				CGPROGRAM
				#pragma vertex vert
				#pragma fragment frag

				#include "UnityCG.cginc"

				float4 _LightDir;
				float4 _ShadowColor;
				float4 _ShadowFalloff;

				struct a2v{
					float4 vertex:POSITION;
				};

				struct v2f{
					float4 pos:SV_POSITION;
					float4 color:COLOR;
				};

				//计算阴影投影点
				float3 ShadowProjectPos(float4 vertexPos){
					float3 shadowPos;
					//计算顶点的世界空间坐标
					float3 worldPos=mul(unity_ObjectToWorld,vertexPos).xyz;

					//灯光方向
					float3 lightDir=normalize(_LightDir.xyz);

					//计算阴影的世界空间坐标(如果顶点低于地面，则阴影点实际就是顶点在世界空间的位置，不做改变)
					shadowPos.y=min(worldPos.y,_LightDir.w);
					shadowPos.xz=worldPos.xz-lightDir.xz*max(0,worldPos.y-_LightDir.w)/(lightDir.y-_LightDir.w);

					return shadowPos;
				}

				v2f vert(a2v v){
					v2f o;

					//得到阴影的世界空间坐标
					float3 shadowPos=ShadowProjectPos(v.vertex);
					
					//将阴影点转换到裁剪空间
					o.pos=UnityWorldToClipPos(shadowPos);

					//得到模型在世界空间地面投影点的位置，然后与地面上的阴影点计算距离算衰减
					float3 center=float3(unity_ObjectToWorld[0].w,_LightDir.w,unity_ObjectToWorld[2].w);
					//这里的unity_ObjectToWorld矩阵前三行的最后一个分量存储的是子对象在父空间下的坐标位置
					float falloff=1-saturate(distance(shadowPos,center)*_ShadowFalloff);

					o.color=_ShadowColor;
					o.color.a*=falloff;

					return o;
				}

				fixed4 frag(v2f i):SV_Target{
					return i.color;
				}
				
				ENDCG
			}
		}		
	}        
实例效果：        
只使用第一个Pass：        
![](https://i.imgur.com/X7PbXN6.png)            
使用第二个Pass后：       
![](https://i.imgur.com/Cqxo4Fk.png)         

这种阴影效果基本能满足要求，而且非常节约性能，只是需要注意的是场景中的主光源以及地面高度平整且不发生变化的情况下比较适用。

### Unity中的渲染优化技术 ###
游戏中主要使用 CPU 和 GPU两种资源，其中CPU负责一定的帧率，GPU负责图像的成像质量，造成游戏性能的瓶颈主要与这两种资源有关：       

- **CPU**     
1.过多的 Draw Call命令   
2.复杂的脚本或者物理模拟      

- **GPU**     
1.顶点处理（包括过多的顶点和逐顶点计算）
2.片元处理（包括过多的片元和逐片元计算）

- **带宽**      
1.使用尺寸较大的未压缩纹理
2.分辨率过高的帧缓存  

针对上述瓶颈，相关的优化处理方式：    

- **CPU优化**   
使用批处理技术减少DrawCall命令数目（DrawCall的合并提交） 

- **GPU优化**      
1.减少需要处理的顶点数目（例如优化几何体，使用模型的LOD（Level of Detail）技术）    
2.使用遮挡剔除技术（这样可以减少需要渲染的图元数目）    
3.减少需要处理的片元数目（例如控制绘制顺序，警惕透明物体，减少实时光照）         
4.减少计算复杂度（使用Shader的LOD技术，代码优化）        

- **节省内存带宽**        
1.减少纹理大小         
2.利用分辨率缩放         

**关于减少DrawCall数目**        
最常见的是使用批处理技术，批处理的思想是每次在针对DrawCall命令时，尽可能处理多个物体。使用同一个材质的物体可以被一起处理，这些物体的差异仅仅在于定点位置不同，将这些定点位置合并后一起提交到GPU，完成一次批处理。      
Unity中提供两种批处理技术，分别是**静态批处理**和 **动态批处理**。       
**动态批处理**        
当场景中的模型共享同一个材质并满足部分条件时，Unity会自动进行批处理操作，花费一个DrawCall就可以渲染该批次的所有模型。动态批处理的基本原理是，每一帧将可以进行批处理的模型网格进行合并，再将合并后的模型数据传递给GPU，使用同一个材质对这些模型进行渲染。动态批处理的物体也可以进行移动。同时，动态批处理需要满足部分条件：       

- 网格的顶点属性规模小于900        
这里是顶点属性，如果在Shader中使用顶点，法线和纹理坐标这3个顶点属性，那么模型的顶点数目不能超过300       

- 使用光照纹理的物体需要特别处理      
使用关照纹理的物体需要额外的渲染参数，包括光照纹理的索引、偏移量和缩放信息等，为了保证这些物体能够被动态批处理，需要保证他们指向光照纹理中的同一个位置。    

- 使用多Pass的物体会中断批处理       
例如，在使用额外的Pass计算阴影和光照效果时，动态批处理会被破坏掉。           

如何查看场景动态批处理？在场景运行时，点击统计按钮，可以得到当前场景的渲染信息，例如上一节中的使用噪声纹理实现的动态雾效的渲染信息：            
![](https://i.imgur.com/wWKjbMZ.png)         
这里的Batches数目即为当前批处理的数目，Saved by batching为21，即动态批处理节约的DrawCall数目为21，在场景中三面墙和三个Cube使用的是同一个材质。            

**静态批处理**         
相比于动态批处理对于顶点属性的限制，静态批处理适用于任何大小的几何模型。静态批处理的实现原理为，只在运行开始阶段，将需要静态批处理的模型合并到一个新的网格结构中，这些模型**只能是场景中的静态物体，不能移动**。静态批处理只需进行一次合并操作，因此相对于动态批处理更加高效，但它需要耗费更多的内存来存储合并后的几何结构。在静态批处理前如果一些物体共享了相同的网格，在内存中每一个物体都会对应一个这个网格的复制品，原本一个网格会变成多个网格发送给GPU。在物体勾选 Batching Static 选项后，Unity会根据该物体的状态实现静态批处理。     

**关于共享材质**         
不管是静态批处理还是动态批处理，都要求物体之间使用同一个材质。针对每个模型不同的渲染要求，不可能大面积使用同一个材质，但可以采取一些方法尽可能多的满足共享材质：         

- 使用图集（atlas）           
如果材质之间仅仅是纹理不一致，可以将纹理合并到一张更大的纹理中，即生成图集，这样这些纹理就成为同一张纹理，采样时使用不同的采样坐标即可。       

- 使用顶点存储额外数据       
有时候，除了纹理不一致，不同的物体在材质的某些参数上也会有微小变化，而不管是静态还是动态批处理都要求物体指向的是同一个材质而不是使用了同一个Shader的材质，即材质的参数也要保持一致，这时候可以通过模型自身的顶点存储这些信息，例如将数据映射到顶点颜色数据保存。        

**关于减少处理的顶点数目**       

- **优化几何体**      
尽可能减少模型的三角面片数目。移除不必要的硬边和纹理衔接，避免边界平滑和纹理分离。         

- **使用模型的LOD技术**       
即根据对象距离摄像机的远近，而使用不同细节的模型。在Unity中，可以使用LOD Group组件为一个物体构建一个LOD。此时需要为同一个对象准备包含不同细节层次的模型，赋予LOD Group组件中的不同等级，Unity会自动判断当前位置上需要使用的层次模型。例如，使用了LOD技术的石狮子在距离摄像机不同距离时的细节：        
![](https://i.imgur.com/a6MLRRC.png)             
![](https://i.imgur.com/VHEvMfk.png)             

**关于减少处理的片元数目**          
处理过多的片元也会造成GPU的性能瓶颈，这部分的优化在于减少OverDraw，OverDraw是指同一个像素被多次绘制。可以采取一些措施来减少OverDraw的发生：          

- **控制绘制顺序**         
在Unity中由于深度测试的存在，可以保证物体从前向后绘制，可以很大程度上减少OverDraw，因为后面的物体无法通过深度测试，因此不会被绘制。Unity中渲染队列数目小于2500（如“Background"、“Geometry”、“AlphaTest”）的对象被认为是不透明（Opaque）的物体，绘制顺序从前向后，而其它队列（“Transparent”、“Overlay”）则是由后向前绘制。因此，尽可能使用不透明渲染队列。       

- **注意半透明物体**         
对于半透明物体来说，由于没有开启深度写入，因此为了得到正确的渲染效果必须从后向前渲染。因此半透明物体几乎肯定会有OverDraw发生，因此需要注意半透明物体的使用和画面占比。       

- **减少实时光照和阴影**           
实时光照的开销比较大，尤其对于移动平台。被逐像素的光源照亮的物体需要被再次渲染，而且不管是静态还是动态批处理，都会被这种多Pass打断。在针对实时光源的处理上，常见的方式是通过生成场景烘焙贴图，然后使用改贴图纹理进行采样得到光照颜色。更为复杂的做法是将复杂的光照计算结果存储在一张查找纹理中，在运行时根据光源方向、视角方向、法线方向等参数，对该查找纹理进行采样。   

**关于节省带宽**      

- **减小纹理大小**     
除了使用纹理图集可以尽可能使用批处理技术从而减少DrawCall外，纹理自身的大小也需要注意。在使用纹理时，纹理的长宽最好是2的整数幂，这样许多内置的优化策略才会奏效。        
另外，尽可能使用多级渐远纹理和压缩纹理。Unity可以在纹理导入时，设置纹理的Advanced选项，勾选Generate Mip Maps生成多级渐远纹理，运行过程汇中根据距离物体的远近而使用不同级别的纹理，类似LOD技术。       
- **设置合理的分辨率**      
移动平台中，可以设置合理的屏幕分辨率以保证一定的帧率运行。

### Unity表面着色器 ###
Unity渲染工程师认为顶点/几何/片元着色器这种Shader是对硬件一种友好的操作，但不符合开发者的思考方式。提出表面着色器/光照模型/光照着色器的模式，其中             

- 表面着色器定义模型表面的反射率、法线和高光    
- 光照模型选择兰伯特还是Blinn-Phong等模型
- 光照着色器负责计算衰减和阴影等   

这种划分模式让开发者大部分时间关注表面着色器，纹理和颜色，光照模型可以提前定义好，使用的时候直接选择使用哪种模型。光照着色器由系统实现，不会轻易改动，这样的话Shader的编写会简便许多。     
**SurfaceShader**在2010年Unity3中被添加，实际上是对顶点/片元着色器的基础上添加了一层抽象。       

一个基础的法线+漫反射纹理的**表面着色器**：           

Shader代码：       

	Shader "Custom/Chapter17_BumpedDiffuse" {
	Properties{
		_Color("Color",Color)=(1,1,1,1)
		_MainTex("MainTex",2D)="white"{}
		_BumpMap("NormalMap",2D)="bump"{}
	}
	SubShader{
		Tags{"Renderer"="Opaque"}
		LOD 300

		CGPROGRAM
		#pragma surface surf Lambert
		#pragma target 3.0

		fixed4 _Color;
		sampler2D _MainTex;
		sampler2D _BumpMap;

		struct Input{
			float2 uv_MainTex;
			float2 uv_BumpMap;
		};

		void surf (Input IN,inout SurfaceOutput o){
			fixed4 tex=tex2D(_MainTex,IN.uv_MainTex);
			o.Albedo=tex.rgb*_Color.rgb;
			o.Alpha=tex.a*_Color.a;
			o.Normal=UnpackNormal(tex2D(_BumpMap,IN.uv_BumpMap));
		}
		ENDCG
	}
	FallBack "Legacy Shader/Diffuse"
	}         

实际效果：      
![](https://i.imgur.com/DurG3GY.png)       
同时可以添加点光源，而无需改动Shader代码：            
![](https://i.imgur.com/60Z717a.png)          

若要实现同样的效果，在顶点片元着色器内所需的代码行数超过100行，而在表面着色器中只需30行就能实现，光照的计算和处理由Unity处理，开发者无需与光照变量打交道。    

表面着色器与顶点/片元着色器不同，CG代码不需要写在一个特定的Pass内，而是直接写在SubShader内，Unity会后台生成多个对应Pass。      

表面着色器最重要的部分为**两个结构体**& **编译指令**。结构体是表面着色器不同函数之间的信息传递介质，而编译指令是与Unity进行沟通。      

**编译指令**       
表面着色器的编译指令用来与Unity进行沟通，设置表面函数的表面属性，使用哪个光照模型，以及其他设置，例如阴影、环境光和雾效等。      
编译指令最重要的是设置**表面函数**和 **光照函数**，并且设置一些可选参数。编译指令格式：       

	#pragma surface surfaceFunction lightModel [optionalparams]     
	//#pragma surface用来指明该编译命令用于表面着色器，       
	//后面需要指定使用的表面函数（surfaceFunction），      
	//光照模型（lightModel）,同时可以通过可选参数进行更多设置      

**表面函数**         
编译指令中的surfaceFunction用来定义表面函数来指定表面的反射率，透明度，法线等属性。通常surfaceFunction即为名为surf的函数，函数格式固定为以下几种：         

	void surf (Input IN, inout SurfaceOutput o)
	void surf (Input IN, inout SurfaceOutputStandard o)
	void surf (Input IN, inout SurfaceOutputStandardSpecular o)     
通常，在表面函数中会使用输入结构体Input IN 设置各种表面属性，将这些属性存储在输出结构体中（不同的输出结构体配合不同的光照模型），然后传递给光照函数计算光照结果。         

**光照函数**      
在指定完表面函数后，需要指定光照函数。光照函数会使用表面函数中设定的各种表面属性，来应用于指定的光照函数，模拟表面光照效果。Unity内置了基于物理的光照模型函数**Standard**和 **StandardSpecular**,和简单的非基于物理的光照模型函数**Lambert**和 **BlinnPhong**。前面的示例中，使用的是Lambert光照函数。同时，也可以定义自己的光照模型函数。    

**其他可选参数**      
在编译指令的最后，可以设定可选参数，如开启/设置透明度混合/透明度测试，指明自定义顶点和颜色修改函数，控制生成的代码等。比较常用和重要的参数包括：       

- **自定义修改函数**        
除了表面函数和光照函数，表面着色器可以支持其他两种自定义函数：顶点修改函数&最后的颜色修改函数，顶点修改函数可以自定义顶点属性，如传递顶点颜色给表面函数，修改顶点位置，实现顶点动画，而最后的颜色修改函数在颜色绘制到屏幕前，最后一次修改颜色值，例如实现自定义雾效。      

- **阴影**     
可以通过指令控制阴影相关的代码。如，**addshadow** 参数可以为表面着色器生成一个阴影投射的Pass。 **fullforwardshadows**参数可以在前向渲染中支持所有光源类型的阴影。如果需要让点光源或聚光灯在前向渲染有阴影，可以添加该参数。如果不想对使用该Shader的物体进行任何阴影计算，可以添加**noshadow**参数禁用阴影。      

- **透明度混合和透明度测试**      
通过**alpha** 和 **alphatest**指令控制透明度混合和透明度测试。如 **alphatest:VariableName**指令会使用对应的变量剔除不满足条件的片元。        

- **光照**       
一些指令可以控制光照对物体的影响。如**noambient**参数不要应用任何环境光照或光照探针(light probe)。 **novertexlights**不用应用任何逐顶点光照。 **noforwardadd**会去掉所有前向渲染中额外的Pass。另外还有控制光照烘焙和雾效模拟的参数，如 **nolightmap** ， **nofog**   

- **控制代码的生成**       
默认情况下，Unity会为一个表面着色器生成相应的前向路径、延迟渲染路径使用的Pass，这样会导致生成的Shader文件比较大。如果知道该表面着色器只会在某些渲染路径中使用，可以通过 **exclude_path:deferred、exclude_path:forward、exclude_prepass**参数来确定不需要为某些渲染路径生成Shader文件。      

**两个结构体**         
表面着色器需要使用两个结构体：表面函数的输入结构体**Input**,以及存储了表面属性的结构体 **SurfaceOutput、SurfaceOutputStandard、SurfaceOutputStandardSpecular**      

**Input**结构体包含了许多表面数据来源，作为表面函数的输入，如果自定义了顶点修改函数，该结构体也会是顶点修改函数的输出结构体。Input支持许多内置变量名。在之前的示例中，使用了 uv_MainTex和uv_BumpMap作为主纹理和法线纹理的采样坐标，类似的变量包括：       
![](https://i.imgur.com/nPWCiaH.png)             
上述变量不需要自己计算，Unity会在背后准备好这些变量，使用者可以直接在表面函数中使用。     

**SurfaceOutput**用于存储表面属性，这些表面属性根据Input结构体提供的数据计算得到，类似的表面属性结构体还包括**SurfaceOutputStandrd、SurfaceOutputStandardSpecular**，这些结构体会作为表面函数的输出，随后作为光照函数的输入进行光照计算。表面函数结构体内的变量是提前声明好的，不可以增加和减少，没有赋值的变量采取默认值，例如 **SurfaceOutput**结构体的变量：    

	struct SurfaceOutput{
		fixed3 Albedo;
		fixed3 Normal;
		fixed3 Emission;
		half Specular;
		fixed Gloss;
		fixed Alpha;
	}            
**SurfaceOutputStandard**           

	struct SurfaceOutputStandard{
		fixed3 Albedo;
		fixed3 Normal;
		half3 Emission;
		half Metallic;
		half Smoothness;
		half Occlusion;
		half Alpha;
	}         
**SurfaceOutputStandardSpecular**        

	struct SurfaceOutputSatanardSpecular{
		fixed3 Albedo;
		fixed3 Specular;
		fixed3 Normal;
		half3 Emission;
		half Smoothness;
		half Occlusion;
		fixed Alpha;
	}     

不同的表面属性结构体的选择取决于所使用的光照模型，如果使用简单的光照模型，如Lambert,BlinnPhong使用第一个结构体即可，而如果使用的是基于物理的光照模型Standard或者是StandardSpecular,分别对应使用SurfaceOutputStandard和SurfaceOutputStandardSpecular结构体。       

**表面着色器的实例分析**        
自定义表面着色器中的4种可自定义函数（顶点修改函数、表面函数函数、光照函数、最后的颜色修改函数），实现顶点沿法线方向扩张效果。              
实例代码：   

	Shader "Custom/Chapter17_NormalExtrusion" {
	Properties{
		_Color("Color",Color)=(1,1,1,1)
		_MainTex("MainTex",2D)="white"{}
		_BumpMap("BumpMap",2D)="bump"{}
		_Amount("ExtrusionAmount",Range(-0.5,0.5))=0.1
	}
	SubShader{
		Tags{"RenderType"="Opaque"}
		LOD 300

		CGPROGRAM
		#pragma surface surf CustomLambert vertex:myvert finalcolor:mycolor addshadow exclude_path:deferred exclude_path:prepass nometa
		#pragma target 3.0

		fixed4 _Color;
		sampler2D _MainTex;
		sampler2D _BumpMap;
		half _Amount;

		struct Input{
			float2 uv_MainTex;
			float2 uv_BumpMap;
		};

		//自定义顶点修改函数
		void myvert(inout appdata_full v){
			v.vertex.xyz+=v.normal*_Amount;
		}

		//自定义表面函数
		void surf(Input IN,inout SurfaceOutput o){
			fixed4 tex=tex2D(_MainTex,IN.uv_MainTex);
			o.Albedo=tex.rgb;
			o.Normal=UnpackNormal(tex2D(_BumpMap,IN.uv_BumpMap));
			o.Alpha=tex.a;	
		}

		//自定义的光照模型函数，兰伯特光照模型，返回漫反射颜色值
		half4 LightingCustomLambert(SurfaceOutput s,half3 lightDir,half atten){
			half NdotL=dot(s.Normal,lightDir);
			half4 c;
			c.rgb=s.Albedo*_LightColor0.rgb*(NdotL*atten);
			c.a=s.Alpha;
			return c;
		}

		//自定义最后颜色修改函数，将光照模型函数的输出作为输入，与_Color的值进行叠加
		void mycolor(Input IN,SurfaceOutput o,inout fixed4 color){
			color*=_Color;
		}
		ENDCG
	}
	FallBack "Legacy Shaders/Diffuse"
	}     
实例效果：     
![](https://i.imgur.com/Gm5k93E.png)        
左边为Amount值为0，即不进行扩张，右边Amount值为0.08    

**SurfaceShader缺点**             
尽管在示例使用的SurfaceShader用了较少的代码量实现了顶点/片元着色器较多代码才能实现的效果，但是性能上是有差别的，因为在使用SurfaceShader时，Unity在背后实现了对应了的顶点/片元着色器代码。SurfaceShader是对顶点/片元着色器的一种封装，这种封装一定程度上会给开发者带来便利，但同时会带来性能上的损耗，尽管能够较快的实现各种光照效果，但是会失去各种优化和特效实现的控制。



----------
### 相关参考 ###
《UnityShader入门精要》    冯乐乐     
### 相关学习链接 ###
关于ZTest和ZWrite :  
[http://www.cnblogs.com/ljx12138/p/5341381.html](http://www.cnblogs.com/ljx12138/p/5341381.html)  
Unity着色器训练营（1）：   
入门篇 [http://forum.china.unity3d.com/thread-27522-1-1.html  ](http://forum.china.unity3d.com/thread-27522-1-1.html   "Unity着色器训练营（1）入门篇")     
小小的顶点变换能实现大大的效果  
(出处: Unity官方中文论坛)  
分享Shader的专栏：    
[https://zhuanlan.zhihu.com/myashader](https://zhuanlan.zhihu.com/myashader)
[https://zhuanlan.zhihu.com/MeowShader](https://zhuanlan.zhihu.com/MeowShader)






	