# RayMarching实践01_SimpleRayMarchingScene #
**写在前面**  
光线追踪简单来说是通过若干条从摄像机发出的光线，通过步进的方式去和场景中的物体求交，根据交点处的信息和光源信息计算交点处的光照。
这种方式区别于与光栅化渲染，也就是将场景中物体的三角面通过空间的变换最终映射到屏幕上，并将三角面覆盖的区域逐像素处理，做插值计算。   
关于光线追踪的简化处理过程，可以参考下图（网络截图，侵删）：     
![](https://i.imgur.com/l1Sa2D2.png)     
**入门简单实践**     
根据光线追踪的处理思路，实现步骤可简化如下：      

1. 获取摄像机的位置 ro 
2. 根据相机位置和朝向，计算当前像素发出的射线方向 rd
3. 求解射线与场景中的物体的交点（使用步进方式求解，直到碰到物体或者达到检测的最大距离）
4. 得到交点处相关信息
5. 根据得到的信息计算最终光照效果   

摄像机的位置通过内置变量_WorldSpaceCameraPos得到，而当前像素的射线方向 rd 则可以通过对视锥体射线进行插值的方式，得到像素点在世界空间下到摄像机的方向向量interpolatedRay,即为所求 rd。interpolatedRay是对近裁剪平面的4个角的特定向量进行插值，这4个向量包含到摄像机的距离和方向信息，可以利用摄像机的相关参数（近裁剪面距离，FOV，横纵比）计算得到。在专栏之前的[利用深度纹理实现更多特效](https://zhuanlan.zhihu.com/p/32870766)笔记中有求过interpolatedRay，即：     
![](https://i.imgur.com/Ncb0KUV.jpg)     
相关代码：      

		Matrix4x4 frustumCorners = Matrix4x4.identity;

        float fov = camera.fieldOfView;
        float near = camera.nearClipPlane;
        float aspect = camera.aspect;

        float halfHeight = near * Mathf.Tan(fov*0.5f*Mathf.Deg2Rad);
        Vector3 toRight = camera.transform.right * halfHeight * aspect;
        Vector3 toTop = camera.transform.up * halfHeight;

        Vector3 topLeft = cameraTransform.forward * near + toTop - toRight;
        float scale = topLeft.magnitude / near;

        topLeft.Normalize();
        topLeft *= scale;

        Vector3 topRight = cameraTransform.forward * near + toTop + toRight;
        topRight.Normalize();
        topRight *= scale;

        Vector3 bottomLeft = cameraTransform.forward * near - toTop - toRight;
        bottomLeft.Normalize();
        bottomLeft *= scale;

        Vector3 bottomRight = cameraTransform.forward * near - toTop + toRight;
        bottomRight.Normalize();
        bottomRight *= scale;

        frustumCorners.SetRow(0,bottomLeft);
        frustumCorners.SetRow(1,bottomRight);
        frustumCorners.SetRow(2,topRight);
        frustumCorners.SetRow(3,topLeft);

        material.SetMatrix("_FrustumCornorsRay",frustumCorners);   

在得到相机位置 ro 和射线方向 rd 后，然后通过步进的方式去求与场景中物体的交点，步进函数代码：     

		#define MARCH_NUM 256 //最多光线检测次数
		float2 RayCast(float3 ro,float3 rd){
			float tmin=0.1;
			float tmax=20.0;

			float t=tmin;
			float2 res=float2(0,-1.0);
			for(int i=0;i<MARCH_NUM;i++){
				float precis=0.0005;
				float3 pos=ro+t*rd;
				res=Map(pos);
				if(res.x<precis||t>tmax) break;
				t+=res.x*0.5;
			}
			if(t>tmax) return float2(t,-1.0);
			return float2(t,res.y);
		}

这里给定一个距离计算函数Map，该函数计算当前步进点距离场景中物体的距离，当计算结果达到给定精度时，认为已经相交，步进策略可以根据需求调整，这里采用的步进策略为每次取余量的1/2。 距离计算函数Map:  

	float MapSphere(float3 pos){
			float radius=0.5;
			float3 centerPos=float3(0.,1.0+sin(_Time.y*1.0)*0.5,0.);
			return length(pos-centerPos)-radius;
		}
	float MapFloor(float3 pos){
			float3 n=float3(0.,1.,0.);
			float  d=0;
			return dot(pos,n)-d;
		}
	float2 Map(float3 pos){
			float distSphere=MapSphere(pos);
			float distFloor=MapFloor(pos);
			if(distSphere<distFloor){
				return float2(distSphere,SPHERE_ID);
			}else{
				return float2(distFloor,FLOOR_ID);
			}
		}
这里通过代码在空间中生成了一个Plane和一个Sphere,    
接下来根据碰撞点信息计算相关光照效果，这里先只做漫反射计算，因此只需要碰撞点的法线向量就可以，法线计算：   

	float3 Normal(float3 pos,float t){
			float val=0.0001*t*t;
			float3 eps=float3(val,0.0,0.0);
			float3 nor=float3(
				Map(pos+eps.xyy).x-Map(pos-eps.xyy).x,
				Map(pos+eps.yxy).x-Map(pos-eps.yxy).x,
				Map(pos+eps.yyx).x-Map(pos-eps.yyx).x);
				return normalize(nor);
			}  
根据碰撞点信息做光照计算：     
	
	float3 ShadingShpere(float3 n){
				float3 col=n;//将法线向量映射为颜色
				float  diff=clamp(dot(n,lightDir),0.,1.);
				float  bklig=clamp(dot(n,-lightDir),0.,1.)*0.05;//加背光
				return col*(diff+bklig);
			}

	float3 ShadingFloor(float3 n,float sd,float3 pos){
				//CheckersGradBox 棋盘纹理绘制
				float f=CheckersGradBox(2.0*pos.xz);
				float3 col=float3(0.2,0.3,0.7)*f;
				float  diff=clamp(dot(n,lightDir),0.,1.);
				return col*diff*sd;
			}

	float3 ShadingBG(float3 rd){
				float val=pow(rd.y,2.0);
				float3 bCol=float3(0,0,0);
				float3 uCol=float3(0.3,0.5,0.8);
				return lerp(bCol,uCol,val);
			}

	float3 Shading(float3 pos,float3 n,float matID){
				float sd=SoftShadow(pos,lightDir);
				if(matID>=(FLOOR_ID-0.5)){
					return ShadingFloor(n,sd,pos);
				}
				else{
					return ShadingShpere(n);
				}
			}

 SoftShadow为阴影计算，该过程从碰撞点出发，向着光源方向做步进，若在步进过程中碰撞到其他物体，说明原先的碰撞点会被遮挡，所以应该进行阴影着色，最后通过Clamp做一个阴影之间的平滑过渡。在Shading着色函数中，只对Floor做了阴影计算。关于ShadingFloor着色中的棋盘纹理绘制，可以参考iq大神的这篇[博客](http://iquilezles.org/www/articles/checkerfiltering/checkerfiltering.htm)，介绍棋盘格纹理的生成及过滤原理。   

 上述过程的完整流程：    

	float4 ProcessRayMarch(float2 uv,float3 ro,float3 rd,inout float sceneDep,float4 sceneCol){
			float2 ret=RayCast(ro,rd);
			float3 pos=ro+rd*ret.x;
			float  matID=ret.y;
			float  rz=ret.x;
			//计算碰撞点法线信息
			float3 nor=Normal(pos,ret.x);
			//使用碰撞点的信息计算当前像素颜色值
			float3 col=Shading(pos,nor,matID);
			if(ret.y<-0.5){
				col=ShadingBG(rd);
			}

			if(sceneDep<rz){
				col=sceneCol;
				rz=sceneDep;
			}
				return float4(col,1.0);
			} 
最后对代码生成的物体与场景中的物体做了深度比较来决定相互的遮挡关系。       

顶点着色器部分主要获取摄像机近裁剪面的4个角向量，提供给片元着色器做像素的插值计算：      

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
		else if(v.texcoord.x>0.5&&v.texcoord.y>0.5){
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
片元着色器部分主要是对深度纹理采样，方便后续的深度值比较，同时对每个像素做步进处理，计算碰撞点的光照：     

	float4 frag(v2f i):SV_Target{
		//SAMPLE_DEPTH_TEXTURE 对深度纹理进行采样
		//LinearEyeDepth 得到视角空间下的线性深度值
		float depth=LinearEyeDepth(SAMPLE_DEPTH_TEXTURE(_CameraDepthTexture,i.uv_depth));
		depth*=length(i.interpolatedRay.xyz);
		//这里MainTex为每一帧相机的渲染纹理
		fixed4 sceneCol=tex2D(_MainTex,i.uv);
		float2 uv=i.uv*float2(_ScreenParams.x/_ScreenParams.y,1.0);
		fixed3 ro=_WorldSpaceCameraPos;
		fixed3 rd=normalize(i.interpolatedRay.xyz);
		return ProcessRayMarch(uv,ro,rd,depth,sceneCol);
	}

最终的渲染场景：       
	![](https://i.imgur.com/hkKwT9a.png)    
白色的Cube与Sphere为Scene下已有的模型，彩色的Sphere和地板则是通过代码生成，光源位置为给定的位置。      

参考：    
[https://blog.csdn.net/tjw02241035621611/article/details/80057928](https://blog.csdn.net/tjw02241035621611/article/details/80057928)          
[https://www.zhihu.com/search?q=%E5%85%89%E7%BA%BF%E8%BF%BD%E8%B8%AA&type=content](https://www.zhihu.com/search?q=%E5%85%89%E7%BA%BF%E8%BF%BD%E8%B8%AA&type=content)       
