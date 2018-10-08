##指向目标Shader实现##
先上效果：
![](https://i.imgur.com/zHLT9ZR.gif)

该效果原贴来自 [实现目标指向的Shader](https://zhuanlan.zhihu.com/p/45321893)   
很感谢作者的耐心交流，这里做简单学习记录     

该效果的实现需要解决两个问题：   

- 使用箭头和准心纹理的条件判断 
- 纹理位置和角度的计算  

### 箭头和准心纹理的条件判断 ###
使用箭头纹理时，说明跟踪目标此时已超出视线范围，因此使用箭头纹理进行提示；    
而是用准星纹理时，说明跟从目标此时就在视野范围内；   

因此，箭头和准星的纹理的选择取决于跟踪目标是否在视野内。对应到裁剪空间里需要判断跟踪目标是否被裁剪：如果被裁剪，说明在视野范围外，需要使用箭头纹理；如果未被裁剪，则说明在视野范围内，使用准星纹理即可。   

首先通过外部脚本，获得跟踪目标的世界位置坐标：     

	public class TargetScripts : MonoBehaviour {

    public GameObject Target;
    void Update()
    {
        GetComponent<Renderer>().material.SetVector("_TargetPosition", Target.transform.position);
    }
	}

在顶点着色器中将跟踪目标的坐标从世界空间转换到裁剪空间：     

	_TargetPosition = UnityWorldToClipPos(_TargetPosition);
	float w = 1/_TargetPosition.w;
	_TargetPosition = float4(_TargetPosition.x*w,_TargetPosition.y*w,_TargetPosition.z*w,0);    

这里需要注意的是，由于顶点着色器内并未完成归一化处理，因此这里手动进行了齐次除法，得到归一化坐标。同时，这里将 TargetPosition 的w分量设置为0，是将 TargetPosition 作为一个方向矢量来处理，来确定纹理位置。 

归一化后，判断是否超出裁剪空间：   

	float l=max(abs(_TargetPosition.x),abs(_TargetPosition.y));
	if( l>=1)
	{
		o.TargetEdge = 1;
    }

超出后，记录需要设置的纹理状态

### 纹理位置的角度计算 ###
纹理位置的计算好说，将选定的纹理按照TargetPosition所指向的方向进行平移即可： 

	o.vertex =_TargetPosition+v.vertex;  
角度的计算主要是在跟踪目标超出视野范围后的处理：       

	//箭头角度旋转计算
	half angle = atan2(_TargetPosition.y,_TargetPosition.x);
	angle -= UNITY_HALF_PI;
	float4 temp=v.vertex;
	v.vertex.x = cos(angle)*temp.x - sin(angle)*temp.y;
	v.vertex.y = sin(angle)*temp.x + cos(angle)*temp.y;   

完整Shader代码：

	Shader "Other/AimedToTarget"
	{
	Properties
	{
		_Color("Color",Color) = (1,1,1,1)
		_ArrowTex ("ArrowTexture", 2D) = "white" {}
		_TargetTex ("TargetTexture", 2D) = "white" {}
		_TargetPosition("TargetPosition",Vector) = (0,0,0,0)
		_Scale("Scale",float) = 1000
	}
	SubShader
	{
		Tags { "RenderType"="TransparentCutout"  "Queue"="Overlay"}
		LOD 100
		Cull Off
		ZWrite Off
		ZTest Off
		Blend SrcAlpha OneMinusSrcAlpha
		
		Pass
		{
			CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag
			#pragma target 3.0
			
			#include "UnityCG.cginc"

			uniform fixed4 _Color;
			uniform sampler2D _ArrowTex;
			uniform sampler2D _TargetTex;
			uniform float4 _TargetPosition;
			uniform float _Scale;

			struct appdata
			{
				float4 vertex : POSITION;
				float2 uv : TEXCOORD0;
			};
			
			struct v2f
			{
				float4 vertex : SV_POSITION;
				float2 uv : TEXCOORD0;
				fixed TargetEdge :  TEXCOORD1;
			};
			
			
			
			v2f vert (appdata v)
			{
				v2f o;
				o.uv = v.uv;
				o.TargetEdge = 0;

				_TargetPosition = UnityWorldToClipPos(_TargetPosition);
				float w = 1/_TargetPosition.w;
				//_TargetPosition 方向矢量(由观察空间转换到裁剪空间，以相机位置为坐标原点)
				_TargetPosition = float4(_TargetPosition.x*w,_TargetPosition.y*w,_TargetPosition.z*w,0);
				
				//_TargetPosition 方向矢量与摄像机方向同反向时的处理
				fixed i = step(0,_TargetPosition.z);
				_TargetPosition.x*=2*i-1;
				_TargetPosition.y*=2*i-1;

				float l=max(abs(_TargetPosition.x),abs(_TargetPosition.y));
				//超出视野范围,更换箭头纹理，并计算旋转角度
				if( l>=1)
				{
					o.TargetEdge = 1;
					//限定坐标范围
					_TargetPosition.x*=0.9/l;
					_TargetPosition.y*=0.9/l;
				   //箭头角度旋转计算
					half angle = atan2(_TargetPosition.y,_TargetPosition.x);
					angle -= UNITY_HALF_PI;
					float4 temp=v.vertex;
					v.vertex.x = cos(angle)*temp.x - sin(angle)*temp.y;
					v.vertex.y = sin(angle)*temp.x + cos(angle)*temp.y;
				}

				_TargetPosition.z=1;
				
				//乘屏幕分辨率系数
				v.vertex.x *= _Scale/_ScreenParams.x;
				v.vertex.y *= _Scale/_ScreenParams.y;

				o.vertex =_TargetPosition+v.vertex;

				return o;
			}
			
			fixed4 frag (v2f i) : SV_Target
			{
				fixed4 col;
				if(i.TargetEdge==1)
					col=tex2D(_ArrowTex, i.uv);
				else
					col=tex2D(_TargetTex, i.uv);
				col*=_Color;
				return col;
			}
			ENDCG
		}
	}
	}