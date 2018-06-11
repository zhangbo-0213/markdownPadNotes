# UnityShader RenderType&Queue 理解 #
在Unity Shader中会经常在SubShader中使用Tags，其中就会涉及RenderType和Queue,如：     

	SubShader{
		Tags{"RenderType"="Opaque"  "Queue"="Geometry"}
		...
	}     

### RenderType ###
RenderType通常使用的值包括：      

- **Opaque**: 用于大多数着色器（法线着色器、自发光着色器、反射着色器以及地形的着色器）。
- **Transparent**:用于半透明着色器（透明着色器、粒子着色器、字体着色器、地形额外通道的着色器）。
- **TransparentCutout**: 蒙皮透明着色器（Transparent Cutout，两个通道的植被着色器）。
- **Background**: Skybox shaders. 天空盒着色器。
- **Overlay**: GUITexture, Halo, Flare shaders. 光晕着色器、闪光着色器。
- **TreeOpaque**: terrain engine tree bark. 地形引擎中的树皮。
- TreeTransparentCutout: terrain engine tree leaves. 地形引擎中的树叶。
- **TreeBillboard**: terrain engine billboarded trees. 地形引擎中的广告牌树。
- **Grass**: terrain engine grass. 地形引擎中的草。
- **GrassBillboard**: terrain engine billboarded grass. 地形引擎何中的广告牌草。     

这些RenderType的类型名称实际上是一种约定，用来区别这个Shader要渲染的对象，当然你也可以改成自定义的名称，只不过需要自己区别场景中不同渲染对象使用的Shader的RenderType的类型名称不同，也就是说RenderType类型名称使用自定义的名称并不会对该Shader的使用和着色效果产生影响。     

指定RenderType的名称，主要是为了配合使用替代渲染的方法：     

	Camera.SetReplacementShader("shader","RenderType")  
在使用替代渲染方法时，相机会使用指定的 shader 来代替场景中的其他 shader 对场景进行渲染。比如现在有 shader1：    

	Shader "shader1"{
		Properties{...}
		SubShader{
		Tags{"RenderType"="Opaque"}
		Pass{...}	
		}
		SubShader{
		Tags{"RenderType"="Transparent"}
		Pass{...}	
		}
	}

场景中一部分物体当前使用的是 shader2：

	Shader "shader2"{
		Properties{...}
		SubShader{
		Tags{"RenderType"="Opaque"}
		Pass{...}	
		}
	}
另一部分使用的是 shader3:    

	Shader "shader3"{
		Properties{...}
		SubShader{
		Tags{"RenderType"="Transparent"}
		Pass{...}	
		}
	}

调用替代渲染的方法：     

	Camera.SetReplacementShader("shader1","") 	
这种情况下，场景中所有的物体就都使用shader1进行渲染（当Shader中包含多个SubShader，在渲染时显卡根据性能从上到下选择第一个能支持的shader）   
如果在调用时，第二个参数不为空字符串，即：      

	Camera.SetReplacementShader("shader1","RenderType")   
这种情况下，首先在场景中找到标签中包含该字符串（这里为"RenderType"）的shader，再去看标签中的该字符串的值与shader1中包含该字符串的值是否一致，一致的话，替换渲染，否则不渲染；由于shader2中包含"RenderType"="Opaque"，而且shader1中的第一个SubShader中包含"RenderType"="Opaque"，因此将shader1中的第一个SubShader替换场景中的所有shader2，同理，将shader1中的第二个SubShader替换场景中的所有的shader3。  

如果shader1为：    

	Shader "shader1"{
		Properties{...}
		SubShader{
		Tags{"RenderType"="Opaque" "A"="On"}
		Pass{...}	
		}
		SubShader{
		Tags{"RenderType"="Transparent"  "A"="Off"}
		Pass{...}	
		}
	}   

shader2为：      

	Shader "shader2"{
		Properties{...}
		SubShader{
		Tags{"RenderType"="Opaque" "A"="On"}
		Pass{...}	
		}
	}  

shader3为：      

	Shader "shader3"{
		Properties{...}
		SubShader{
		Tags{"RenderType"="Transparent" "A"="On"}
		Pass{...}	
		}
	}	   

替代渲染的调用方式为：     

	Camera.SetReplacementShader("shader1","A")      
最后的结果是，shader1的第一个SubShader将会替换shader2和shader3  

### Queue ###
Queue渲染队列，用来指定当前shader作用的对象的渲染顺序：      
Unity中的几种内置的渲染队列，按照渲染顺序，从先到后进行排序，队列数越小的，越先渲染，队列数越大的，越后渲染。

- Background（1000） 最早被渲染的物体的队列。
- Geometry   （2000） 不透明物体的渲染队列。大多数物体都应该使用该队列进行渲染，也是Unity Shader中默认的渲染队列。
- AlphaTest   （2450） 有透明通道，需要进行Alpha Test的物体的队列，比在Geomerty中更有效。
- Transparent（3000） 半透物体的渲染队列。一般是不写深度的物体，Alpha Blend等的在该队列渲染。
- Overlay      （4000） 最后被渲染的物体的队列，一般是覆盖效果，比如镜头光晕，屏幕贴片之类的   

