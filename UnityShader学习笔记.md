## Unity Shader入门学习##
**学习教材：《UnityShader入门精要》**  

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
模型-->切线空间下的转换矩阵计算，首先切线空间-->模型空间的变换矩阵为模型顶点的切线方向（x轴）、副切线方向（y轴）、法线方向（z轴）的**按列排列**形式，即模型-->切线空间变换矩阵的逆矩阵，而对于一个方向矢量而言，一个变换矩阵若只存在平移和旋转变换，则该矩阵为一个正交阵，即变换矩阵的逆矩阵与转置矩阵相等，因此模型-->切线空间的变换矩阵为逆矩阵的转置，即将模型顶点的切线方向（x轴）、副切线方向（y轴）、法线方向（z轴）的 **按行排列**。   
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


