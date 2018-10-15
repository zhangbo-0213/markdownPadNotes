## 初识 Unity SRP##
Unity SRP 即 Unity Scriptable Rendering Pipeline(可编程渲染管线)，是Unity 2018的新功能，使开发者可以通过脚本按需构建自己的渲染过程。在学习和参考：

[吉祥的游戏编程笔记](https://zhuanlan.zhihu.com/c_180198728) 

中关于Unity SRP的相关内容后，这里做一个简单的学习记录，如有错误之处，希望可以多多交流。  
SRP中的内容可以用一张图说明：
![](https://i.imgur.com/UBkd3SX.png)
SRP的创建过程分为3个部分：   


- Custom Render Pipeline 
- Custom Render Pipeline Asset
- Shader   

### Custom Render Pipeline ###
该部分是自定义渲染管线的起点，也是核心部分，继承自RenderPipeline类。在该类中的Render()方法中，定义自己的渲染流程中的规则及进行参数设置。一般来说，渲染的过程是将摄像机视野中3D或2D的场景对象进过一系列的处理最后转换成一张2D的图片输出到屏幕上。因此在Render()方法中，处理起点是从编辑场景里的每一个输出相机开始，经过如下过程：      

- **设置渲染目标**
- **绘制天空盒**
- **执行裁剪过程**
- **执行过滤过程**
- **绘制场景准备**
- **执行管线**

在这一系列的过程中，需要将相关的指令和设置提交，通过 CommandBuffer和ScriptableRenderContext对象实现。 CommandBuffer对象作为指令缓存集，记录部分指令然后集中提交。ScriptableRenderContext对象可以理解成渲染过程中的一个管理器，CommandBuffer对象的指令提交需要通过ScriptableRenderContext对象完成，同时ScriptableRenderContext对象还可以管理如天空球的绘制、下达执行管线渲染等操作。      
Custom Render Pipeline 完整代码：      

	using UnityEngine;
	using UnityEngine.Experimental.Rendering;
	using UnityEngine.Rendering;


	namespace Kata03 {
    public class CustomRenderPipeline : RenderPipeline
    {
        CommandBuffer _cb;

        //该函数在管线销毁时调用
        public override void Dispose()
        {
            base.Dispose();
            if (_cb != null) {
                _cb.Clear();
                _cb = null;
            }
        }

        //该函数在管线渲染时调用
        public override void Render(ScriptableRenderContext renderContext, Camera[] cameras)
        {
            base.Render(renderContext, cameras);

            if (_cb == null) {
                _cb = new CommandBuffer();
            }

            //设置Shader中要使用的光源变量名
            var _LightDir = Shader.PropertyToID("_LightDir");
            var _LightColor = Shader.PropertyToID("_LightColor");
            var _CameraPos = Shader.PropertyToID("_CameraPos");

            //对于每个相机执行的操作
            foreach (var camera in cameras)
            {
                //设置渲染上下文相机属性
                renderContext.SetupCameraProperties(camera);

                _cb.name = "Setup";
                //显式设置渲染目标为相机BackBuffer(如果相机没有指定渲染纹理，则直接绘制到屏幕)
                _cb.SetRenderTarget(BuiltinRenderTextureType.CameraTarget);
                //设置渲染目标颜色为相机背景色
                _cb.ClearRenderTarget(true, true, camera.backgroundColor);

                //设置相机的着色器全局变量
                Vector4 CameraPosition = new Vector4(camera.transform.localPosition.x, camera.transform.localPosition.y, camera.transform.localPosition.z, 1.0f);
                _cb.SetGlobalVector(_CameraPos, camera.transform.localToWorldMatrix * CameraPosition);
                renderContext.ExecuteCommandBuffer(_cb);
                _cb.Clear();

                //天空盒绘制
                renderContext.DrawSkybox(camera);

                //执行裁剪
                var culled = new CullResults();
                CullResults.Cull(camera, renderContext, out culled);

                /*
                 裁剪结果包括：
                    可见的物体列表:visibleRenderers
                    可见灯光列表:visibleLights
                    可见反射探针(CubeMap):visibleReflectionProbes
                 裁剪结果并未排序
                 */

                //获取所有灯光
                var lights = culled.visibleLights;
                _cb.name = "RenderLights";
                foreach (var light in lights)
                {
                    //挑选出平行光处理
                    if (light.lightType != LightType.Directional) continue;
                    //获取光源方向
                    Vector4 pos = light.localToWorld.GetColumn(0);
                    Vector4 lightDir = new Vector4(pos.x,pos.y,pos.z,0);
                    //获取光源颜色
                    Color lightColor = light.finalColor;
                    //构建shader常量缓存
                    _cb.SetGlobalVector(_LightDir,lightDir);
                    _cb.SetGlobalColor(_LightColor,lightColor);
                    renderContext.ExecuteCommandBuffer(_cb);
                    _cb.Clear();

                    var rs = new FilterRenderersSettings(true);
                    //只渲染固体范围
                    rs.renderQueueRange = RenderQueueRange.opaque;
                    //包括所有层
                    rs.layerMask = ~0;

                    //渲染设置，使用Shader中LightMode为"BaseLit"的Pass
                    var ds = new DrawRendererSettings(camera,new ShaderPassName("BaseLit"));
                    //物体绘制
                    renderContext.DrawRenderers(culled.visibleRenderers,ref ds,rs);

                    break;
                }

                //开始执行管线
                renderContext.Submit();
            }
        }
      }
	}
 
### Custom Render Pipeline Asset ###
Custom Render Pipeline Asset 继承自 Render Pipeline Asset，用来在项目中生成自定义渲染管线资源，主要实现 IRenderPipelineAsset接口方法， InternalCreatePipeline()，生成自定义的 Custom RenderPipeline 对象，完整代码：  

	using UnityEngine.Experimental.Rendering;

	#if UNITY_EDITOR
	using UnityEditor;
	using UnityEditor.ProjectWindowCallback;
	#endif

	namespace Kata03 {
   
    public class CustomRenderPipelineAsset : RenderPipelineAsset
    {
	#if UNITY_EDITOR
        [MenuItem("Assets/Create/Render Pipeline/Kata03/Pipeline Asset")]
        static void CreateKata03Pipeline() {
            ProjectWindowUtil.StartNameEditingIfProjectWindowExists(0,CreateInstance<CreateKata03PipelineAsset>(),"Kata03 Pipeline.asset",null,null);
        }
        class CreateKata03PipelineAsset : EndNameEditAction
        {
            public override void Action(int instanceId, string pathName, string resourceFile)
            {
                var instance = CreateInstance<CustomRenderPipelineAsset>();
                AssetDatabase.CreateAsset(instance,pathName);
            }
        }

	#endif

        protected override IRenderPipeline InternalCreatePipeline()
        {
            return new CustomRenderPipeline();
        }
      }
	}
有了 Custom Render Pipeline Asset 后，在Projects中通过 Create->RdnerPipeline->Kata03->Pipeline 即可创建对应的自定义渲染管线资源，然后替换原有的管线，在Edit->Project->Settings->Graphics中的Scriptable Render PipelineLine Settings中完成替换。
### Shader ###
Shader是配合当前的自定义管线，完成场景内的物体着色，这和之前使用Unity内置管线没有区别。只是这里有一点需要注意的是：  
在CustomRenderPipeline的渲染设置中，

	var ds = new DrawRendererSettings(camera,new ShaderPassName("BaseLit"));

这里在构建ShaderPassName对象时，传递的参数为使用的Shader的Pass中"LightMode"对应的名称，因为在自定义渲染管线中，场景内物体光照着色的区分是通过Shader内的Pass里的"LightMode"进行的。包含光照计算的Shader代码：    

	Shader "Custom/BaseDirLit"
	{
	Properties
	{
		_Color("Tint", Color) = (0.5,0.5,0.5,1)
		_DiffuseFactor("Diffuse Factor", Range(0,1)) = 1
		_SpecularColor("Specular Color",Color)=(1,1,1,1)
		_SpecularFactor("Specular Factor", Range(0,1)) = 1
		_SpecularPower("Specular Power",Float) = 100
	}
  
	HLSLINCLUDE

	#include "UnityCG.cginc"
	#define PI 3.14159265359

	uniform float4 _LightDir;
	uniform float4 _LightColor;
	uniform float4 _CameraPos;
	uniform float4 _Color;
	uniform float _DiffuseFactor;
	uniform float _SpecularFactor;
	uniform float _SpecularColor;
	uniform float _SpecularPower;

	struct a2v
	{
		float4 vertex : POSITION;
		float4 normal : NORMAL;
	};

	struct v2f
	{
		float4 pos : SV_POSITION;
		float4 normalWorld : TEXCOORD1;
		float4 worldPos : TEXCOORD2;
	};

	v2f vert(a2v v)
	{
		v2f o;
		UNITY_INITIALIZE_OUTPUT(v2f,o);
		o.pos = UnityObjectToClipPos(v.vertex);
		o.normalWorld = float4(normalize(mul(normalize(v.normal.xyz), (float3x3)unity_WorldToObject)),v.normal.w);
		o.worldPos = mul(unity_ObjectToWorld,v.vertex);
		return o;
	}

	half4 frag(v2f i) : SV_Target
	{
		fixed4 diffuse=_DiffuseFactor*max(0.0,dot(_LightDir.xyz,normalize(i.normalWorld.xyz)))*_Color*_LightColor;
		fixed3 viewDir=normalize(_CameraPos.xyz-i.worldPos.xyz);
		fixed3 halfDir=normalize(_LightDir.xyz+viewDir);
		fixed4 specular= _LightColor*_SpecularFactor*pow(max(0,dot(normalize(i.normalWorld.xyz),halfDir)),_SpecularPower)*max(0,saturate(dot(normalize(i.normalWorld.xyz),_LightDir.xyz)))*(_SpecularPower+8)/(8+PI);
		return diffuse+specular;
	}

	ENDHLSL

	SubShader
	{
		Tags{ "Queue" = "Geometry" }
		LOD 100
		Pass
		{
			Tags {"LightMode" = "BaseLit"}

			HLSLPROGRAM
			#pragma vertex vert
			#pragma fragment frag

			ENDHLSL
		}
	  }
	}
在场景中使用该Shader的材质，得到的对应效果：     
![](https://i.imgur.com/nGeW75y.png)  