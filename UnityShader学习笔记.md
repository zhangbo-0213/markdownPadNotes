## Unity Shader入门学习##
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