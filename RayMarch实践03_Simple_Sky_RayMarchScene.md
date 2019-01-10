### RayMarch简单实践03 Simple-Sky-RayMarchingScene ###
**写在前面**   
本篇笔记记录利用从摄像机发出到像素点的射线在世界空间下的位置点坐标，来计算像素最终着色的方式生成带云层的天空：   
![](https://i.imgur.com/P45eTNK.png)  
云层天空的生成过程中，场景中没有生成其他物体，因此针对物体着色的光线步进检测，碰撞点的法线计算，以及从碰撞体发出朝光照方向进行的步进检测来计算阴影着色的过程并不会使用到。由于计算过程中需要使用摄像机的位置ro,以及从摄像机发出到像素的射线rd，因此还是将本篇笔记归于 RayMarch 实践。   

**简单实践**  
可复用的部分与 [RayMarch简单实践02](https://zhuanlan.zhihu.com/p/53522599) 大致相同：    
![](https://i.imgur.com/KQW2NbQ.png)   
由于不需要生成场景物体，去掉SDF.cginc部分，增加针对天空渲染的SkyScene.shader， BaseFrame.cginc中是顶点着色器和片元着色器的处理过程，同上一篇中一样，顶点着色器主要是传递摄像机4个角向量供片元着色器进行插值计算射线方向rd,而片元着色器里需要处理的过程主要是对深度纹理采样，方便后续的深度值比较。核心部分是SkyScene.shader中对云层，太阳的着色过程：   

- **云层着色过程**    
云层的着色过程主要利用射线rd对噪声进行采样：    


		float3 Cloud(float3 bgCol,float3 ro,float3 rd,float3 cloudCol,float spd,float layer){
			float3 col=bgCol;
			float time=_Time.y*0.05*spd;
			//不同的云层分层 添加不同的基本高度
			for(int i=0;i<layer;i++){
			//采样坐标与rd.xz/rd.y关联，使得同一个xz的圆截面下,根据rd.y的值连续采样
			//越远(rd.y的值越小),采样点越靠近，相对采样结果越相近，使得远处的云越密，近处的云越疏
			//float2 sc=rd.xz*((i+3)*40000.0)/(rd.y);
			float2 sc=mul(float2x2(0.8,-0.6,0.6,0.8),rd.xz*((i+3)*40000.0)/(rd.y));
			//云层颜色与背景色混合
			col=lerp(col,cloudCol,0.5*smoothstep(0.4,0.8,TimeFBM(sc*0.00002,time*(i+3))));
			}
		return col;
		}  
TimeFBM:    
	
		float TimeFBM(float2 p,float t){
			float2 f=0.0;
			float s=0.5;
			float sum=0;
			//分形叠加
			for(int i=0;i<5;i++){
				//采样位置添加时间偏移
				p+=t;
				//每一层时间偏移不同，达到分层下每一层不同的移速效果
				t*=1.5;
				//噪声纹理采样结果，单通道
				//f+=s*tex2D(_NoiseTex,p/256).x;
				f+=s*VNoise(p);
				//采样点做旋转，产生流动感
				p=mul(float2x2(0.8,-0.6,0.6,0.8),p)*2.02;
				//FBM系数操作
				sum+=s;
				s*=0.6;
			}
		return f/sum;
		}  
TimeFBM是对采样过程的一个分形叠加，其基本原理与之前笔记 [程序噪声](https://zhuanlan.zhihu.com/p/40984211) 里的分形叠加原理一样，另外对于采样过程，可以使用噪声纹理，也可以直接使用程序噪声来生成，如 PerlinNoise,ValueNoise等  

**太阳着色过程**   
太阳的着色直接利用rd与场景中的lightDir进行点乘，同时再做pow进行约束处理，这样越与lightDir平行的rd方向，其太阳光在屏幕上着色越明显：    

		//太阳绘制
		//指定太阳光方向，利用pow约束增强rd与光照方向点积结果，
		//形成某一区域内的颜色值较强,绘制太阳效果
		col+=0.35*float3(1.0,0.2,0.1)*pow(sundot,5.0);
		col+=0.35*float3(1.0,0.3,0.2)*pow(sundot,64.0);
		col+=0.25*float3(1.0,0.8,0.6)*pow(sundot,512.0);   

**完整的天空着色过程**    
		
		float3 SkyRender(float3 ro,float3 rd){
				fixed3 col=fixed3(0.0,0.0,0.0);
				float3 light1=normalize(float3(0.8,0.4,0.8));
				float sundot=clamp(dot(rd,light1),0.0,1.0);

				//基本天空颜色过渡
				//利用rd在高度方向上的分量值进行颜色混合插值
				col=float3(0.2,0.5,0.85)*1.1-rd.y*rd.y*0.5;
				col=lerp(col,0.85*float3(0.7,0.75,0.85),pow(1.0-max(rd.y,0.0),4.0));

				//太阳绘制
				//指定太阳光方向，利用pow约束增强rd与光照方向点积结果，
				//形成某一区域内的颜色值较强,绘制太阳效果
				col+=0.35*float3(1.0,0.2,0.1)*pow(sundot,5.0);
				col+=0.35*float3(1.0,0.3,0.2)*pow(sundot,64.0);
				col+=0.25*float3(1.0,0.8,0.6)*pow(sundot,512.0);

				//云
				col=Cloud(col,ro,rd,float3(1.0,0.9,0.9),1,1);
				//过滤地平线以下的部分
				col=lerp(col,0.68*float3(0.4,0.65,1.0),pow(1.0-max(rd.y,0.0),16.0));
				return col;
			}  

这样在   ProcessRayMarch() 中调用

		float4 ProcessRayMarch(float2 uv,float3 ro,float3 rd,inout float sceneDep,float4 sceneCol){
				sceneCol.xyz=SkyRender(ro,rd);
				return sceneCol;
			} 

完成天空着色过程：   
![](https://i.imgur.com/e2OSWnh.png)          

	
		