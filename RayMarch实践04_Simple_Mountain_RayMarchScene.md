### RayMarch简单实践04 Simple-Mountain-RayMarchingScene ###
**写在前面**   
本篇笔记记录在上一篇 RayMarch简单实践03 Simple-Sky-RayMarchingScene 的基础上为场景中添加山体的过程：  
![](https://i.imgur.com/WwMhWiZ.png)
思路与生成云朵类似，以像素点在世界空间的坐标作为输入，利用分形噪声来形成连续带有随机性的值，再通过光线步进来确定对应位置点的着色情况。与生成云朵不同之处在于，山体的着色部分会稍微复杂一些，另外山体的生成过程无需与时间变量相关联。    
**简单实践**    
可复用部分与 RayMarch简单实践03 Simple-Sky-RayMarchingScene 大致相同：   
![](https://i.imgur.com/3Zh4ccH.png)  
 
增加了Mountain.cginc部分，通过分形噪声来生成山体的随机高度，以分形迭代的次数作为参数对应低中高三个不同的精度，由于Raycast中做光线步进是会循环多次，为简化计算采用低精度的Map函数，阴影计算过程中步进函数的次数少于Raycast，采用中精度的MAP函数，而法线计算过程中只有一次，因此采用高精度Map函数：   

	float2 TerrainMap_L(float3 pos);
	float2 TerrainMap_M(float3 pos);
	float2 TerrainMap_H(float3 pos);

	float RaycastTerrain(float3 ro,float3 rd){
	_MACRO_RAY_CAST(ro,rd,10000,TerrainMap_L);
	}

	float3 NormalTerrain(in float3 pos,float rz){
	_MACRO_CALC_NORMAL(pos,rz,TerrainMap_H);
	}

	float SoftShadow(in float3 ro,in float3 rd,float tmax){
	_MACRO_SOFT_SHADOW(ro,rd,tmax,TerrainMap_M);
	}  

Mountain.shader部分主要是山体的着色计算，同时实现TerrainMap函数：   

	//地形噪声FBM分形处理
	#define TerrainMap(pos,NUM)\
	float2 p=pos.xz*0.9/_MaxTerrainH;\
	float a=0.0;\
	float b=0.491;\
		for(int i=0;i<NUM;i++){\
			float n=VNoise(p);\
			a+=b*n;\
			b*=0.49;\
			p*=2.01;\
		}\
	return float2(pos.y-_MaxTerrainH*a,1.0);

	float2 TerrainMap_L(float3 pos){
				TerrainMap(pos,5.);}
	float2 TerrainMap_M(float3 pos){
				TerrainMap(pos,9.);}
	float2 TerrainMap_H(float3 pos){
				TerrainMap(pos,15.);}
着色部分包括基础光照，阴影和雾效的计算：   

	float3 MountainRender(float3 pos,float3 ro,float3 rd,float rz,float3 nor,float3 lightDir){
		float3 col=float3(0.0,0.0,0.0);
		//山体基础颜色
		col=float3(0.1,0.09,0.08);

		//基础光照部分
		float amb=clamp(0.5+0.5*nor.y,0.0,1.0);
		float dif=clamp(dot(lightDir,nor),0.0,1.0);
		float bac=clamp(0.2+0.8*dot(normalize(float3(-lightDir.x,0.0,lightDir.z)),nor),0.0,1.0);

		//阴影
		float sh=SoftShadow(pos+lightDir*_MaxTerrainH*_ShadowScale,lightDir,_MaxTerrainH*1.2);

		float3 lin=float3(0.0,0.0,0.0);
		lin+=amb*float3(0.40,0.60,1.00)*1.2;
		lin+=dif*float3(7.00,5.00,3.00)*float3(sh, sh*sh*0.5+0.5*sh, sh*sh*0.8+0.2*sh);
		lin+=bac*float3(0.40,0.50,0.60);
		col*=lin;
		//步进距离累加值的指数雾效
		float fo=1.0-exp(-pow(0.1*rz/_MaxTerrainH,1.5));
		//float fo=(rz/_MaxTerrainH)*0.2;
		float3 fco=0.65*float3(.4,.65,1.0);
		col=lerp(col,fco,fo);

		return col;}   

最后根据光线步进的结果来确定天空和山体的着色：   

	float4 ProcessRayMarch(float2 uv,float3 ro,float3 rd,inout float sceneDep,float4 sceneCol){
		float tmax=5000.0;
		float rz=RaycastTerrain(ro,rd).x;
		float3 pos=ro+rd*rz;
		float3 nor=NormalTerrain(pos,rz);
				
		float3 col=float3(0.0,0.0,0.0);
		if(rz>tmax){
			col=Sky(ro,rd,_LightDir);
		}
		else{
			col=MountainRender(pos,ro,rd,rz,nor,_LightDir);
		}
		col=pow(col,float3(0.4545,0.4545,0.4545));
		sceneCol.xyz=col;
		return sceneCol;
		}  

参考：   
[https://blog.csdn.net/tjw02241035621611/article/details/80106320](https://blog.csdn.net/tjw02241035621611/article/details/80106320)
