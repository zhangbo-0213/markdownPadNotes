# Unity Shader中的 ZTest & ZWrite #
### Part1 理论部分 ###
ZTest：深度测试，开启后测试结果决定片元是否被舍弃，可配置    
ZWrite：深度写入，开启后决定片元的深度值是否写入深度缓冲，可配置     

ZTest可设置的测试规则：      

- ZTest Less：深度小于当前缓存则通过     
- ZTest Greater：深度大于当前缓存则通过   
- ZTest LEqual：深度小于等于当前缓存则通过      
- ZTest GEqual：深度大于等于当前缓存则通过      
- ZTest Equal：深度等于当前缓存则通过     
- ZTest NotEqual：深度不等于当前缓存则通过       
- ZTest Always：不论如何都通过           

注意，**ZTest Off等同于ZTest Always**，关闭深度测试等于完全通过。    

ZWrite配置：       
ZWrite On    深度写入开启
ZWrite Off    深度写入关闭

ZTest和ZWrite发生在逐片元操作过程，处于片元着色后，最终屏幕输出前。ZTest与ZWrite的具体操作流程：      

![](https://i.imgur.com/rUeLv3v.png)   
图片来自 《UnityShader 入门精要——冯乐乐》   
从流程图中可以看出：     

- 在开启ZTest下，没有通过测试的片元部分是直接被舍弃，通过测试的片元被保留下来
- 在关闭ZTest下，不存在片元被舍弃的情况，也就是说，关闭深度测试，整个片元是被保留下来的
- 在ZWrite开启状态下，**只有保留下来片元**深度值才能被写入深度缓冲    

即：      
1.深度测试通过，深度写入开启：写入深度缓冲区，写入颜色缓冲区    
2.深度测试通过，深度写入关闭：不写深度缓冲区，写入颜色缓冲区    
3.深度测试失败，深度写入开启：不写深度缓冲区，不写颜色缓冲区  
4.深度测试失败，深度写入关闭：不写深度缓冲区，不写颜色缓冲区        

### part2 实践部分 ###
场景中放置3个Cube，这里为了测试方便采用了自定义的shader，使用统一的渲染队列，关于渲染队列可以[参考这里](https://blog.csdn.net/u013477973/article/details/80607989)     
自定义shader为一个简单的颜色输出：      

	Shader "S01_Origin"
	{
	Properties
	{
		_MainColor ("MainColor", color) =(0.5,0.5,0.5,1)
	}
	SubShader
	{
		Tags{ "Queue" = "Geometry" }
		Pass
		{
		CGPROGRAM
		#pragma vertex vert
		#pragma fragment frag		
		#include "UnityCG.cginc"

		fixed4 _MainColor;

		struct a2v
		{
		float4 vertex : POSITION;
		};

		struct v2f
		{
		float4 pos : SV_POSITION;
		};

		v2f vert(a2v v)
		{
		v2f o;
		o.pos = UnityObjectToClipPos(v.vertex);
		return o;
		}

		fixed4 frag(v2f i) : SV_Target
		{
		return _MainColor;
		}
		ENDCG
		}
	}
	}  
指定的渲染队列 "Queue"="Geometry" 不透明物体的默认队列，目前场景中的Cube均使用该shader的材质，只是更改一下颜色       

![](https://i.imgur.com/fpleYd3.png)        

3个Cube左侧为Cube,距离摄像机最近，中间为Cube(1),右侧为Cube(2)，距离摄像机最远，通过Frame Debug窗口查看这三个Cube的渲染顺序为：     

![](https://i.imgur.com/4AWeKc6.png)       
先渲染Cube,然后Cube(1),最后Cube(2) ,因此是距离摄像机最近（深度值小）及远进行渲染。   

现在将中间Cube更换材质所对应的Shader：
	
	Shader "S01_ZTestZWrite"
	{
	Properties
	{
		_ColorOut("ColorOut",color) = (0,0,1,0.5)
		_ColorIn("CoorIn",color) = (0,1,0,0.5)
	}
	SubShader
	{
		Tags{ "Queue" = "Geometry" }
		Pass
			{
				CGPROGRAM
				#pragma vertex vert
				#pragma fragment frag		
				#include "UnityCG.cginc"

				fixed4 _ColorIn;

				struct a2v
				{
					float4 vertex : POSITION;
				};

				struct v2f
				{
					float4 pos : SV_POSITION;
				};

				v2f vert(a2v v)
				{
					v2f o;
					o.pos = UnityObjectToClipPos(v.vertex);
					return o;
				}

				fixed4 frag(v2f i) : SV_Target
				{
					return _ColorIn;
				}
					ENDCG
			}
		}
	}

**1.开启ZTest(LEqual),开启ZWrite  默认状态**      
![](https://i.imgur.com/CG6FbXu.png)     
Cube,Cube(1),Cube(2)依次遮挡      

**2.开启ZTest(LEqual),关闭ZWrite**         
![](https://i.imgur.com/y1Eq7PA.png)       
渲染顺序是Cube,Cube(1),Cube(2)      
由于Cube(1)开启了深度测试，在Cube渲染完成后，被Cube遮挡的地方没有通过深度测试，因此没有显示，在渲染Cube(2)时，由于Cube(1)关闭了深度写入，因此即使被Cube(1)挡住的部分，Cube(2)仍然会通过深度测试，并显示出来      

**3.关闭Ztest，开启Zwrite**         
![](https://i.imgur.com/aNDQzIx.png)
渲染顺序是Cube,Cube(1),Cube(2)    
由于Cube(1)关闭深度测试，因此Cube(1)不存在片面被舍弃的情况，所以全部保留，同时开启深度写入，因此在渲染Cube(2)时，Cube(2)在进行深度测试时就舍弃掉被Cube(1)遮挡的地方       

**4.关闭Ztest,关闭ZWrite**        
![](https://i.imgur.com/aOChVAD.png)
渲染顺序是Cube,Cube(1),Cube(2)    
由于Cube(1)关闭深度测试，因此Cube(1)不存在片面被舍弃的情况，所以全部保留，同时关闭深度写入，因此在渲染Cube(2)时，Cube(2)在进行深度测试时，即使被Cube(1)挡住的部分，Cube(2)仍然会通过深度测试，并显示出来       

对于上述四种情况，如果能够理解，在改变Cube(1)的渲染队列的情况下产生的结果同样也能知道为什么，主要关心下深度缓存中的值是什么，例如改变Cube(1)的 "Queue"="Transparent" ，即在Cube和Cube(2)渲染完成后再进行渲染          
**5.Queue=Transparent  / 关闭ZTest  关闭ZWrite**
![](https://i.imgur.com/b3Xbs4l.png)      
可以看到与第4种情况相比，Cube(1)即使关闭了深度写入，Cube(2)也被Cube(1)遮挡的部分也没有显示出来，这是由于Cube(1)的渲染队列在Cube(2)之后，这个时候Cube(1)的ZWrite开启和关闭的效果是一样的


对于默认情况下，即开启Ztest(LEqual),开启Zwrite情况下，更改Ztest的测试规则：     

**6.ZTest Greater  开启ZWrite**      
![](https://i.imgur.com/BZzn39n.png)      
渲染顺序是Cube,Cube(1),Cube(2)     
由于Cube(1)的深度测试规则是大于当前深度缓存中的值的片元保留下来，否则舍弃该片元，因此Cube(1)中被Cube遮挡部分深度值是大于做深度测试时深度缓存中的值，因此该片元被保留下来      

如果此时想要Cube(1)的没有被遮挡的部分也显示，则需要在原有的shader中再增加一个Pass，设置第二个Pass的Ztest为LEqual     

![](https://i.imgur.com/Y5TtGxj.png)   
这里第二个Pass中的颜色改为蓝色加以区分，第一个Pass做深度测试时保留深度值较大的片元，即绿色部分，第二个Pass做深度测试时保留深度值较小的片元，即蓝色部分

