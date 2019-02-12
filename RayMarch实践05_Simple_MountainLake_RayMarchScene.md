### RayMarch简单实践05 Simple-MountainLake-RayMarchingScene ###
**写在前面**   
本篇笔记记录在上一篇 RayMarch简单实践04 Simple-Mountain-RayMarchingScene 的基础上为场景中添加湖泊的过程：  
![](https://i.imgur.com/G5WX94z.png)     
由于需要模拟水面的随机扰动，因此需要用到分形噪声来产生连续的随机值，同样以像素点在世界空间中的坐标的位置作为输入，由于水平面的高度统一，主要使用的是坐标位置的x,z分量，同时水面的着色过程分为折射和反射两部分，先计算折射部分，在计算反射部分。     
**简单实践**   
可复用部分与 RayMarch简单实践04 Simple-Mountain-RayMarchingScene 大致相同：     
![](https://i.imgur.com/uxTpB8I.png)       
Mountain.cginc部分与上一篇一致，通过分形噪声来生成山体的随机高度，以分形迭代的次数作为参数对应低中高三个不同的精度，shader部分增加了水面法线计算函数和水面的着色计算。水面法线函数：     

	float3 WaterNormal(float3 pos,float rz){
		float EPSILON = 0.003*rz*rz;
		float3 dx = float3( EPSILON, 0.,0. );
		float3 dz = float3( 0.,0., EPSILON );
					
		float3	normal = float3( 0., 1., 0. );
		float bumpfactor = 0.4 * pow(1.-clamp((rz)/1000.,0.,1.),6.);//根据距离所见Bump幅度
				
		normal.x = -bumpfactor * (WaterMap(pos + dx) - WaterMap(pos-dx) ) / (2. * EPSILON);
		normal.z = -bumpfactor * (WaterMap(pos + dz) - WaterMap(pos-dz) ) / (2. * EPSILON);
		return normalize( normal );	
		}   

这里直接改变水面碰撞点的法线，通过法线相关的着色计算来模拟水面的扰动，同时根据步进距离rz来确定扰动的强度，达到近处扰动大，远处扰动小的效果。    
水体的着色计算过程：   

	float maxT = 10000;
	float minT = 0.1;
	float3 col  = float3 (0.,0.,0.);
	float waterT = maxT;
		if(rd.y <-0.01){
			float t = -(ro.y - waterHeight)/rd.y;
			waterT = min(waterT,t);}
		float sundot = clamp(dot(rd,_LightDir),0.0,1.0);
		float rz = RaycastTerrain(ro,rd);
		float firstInsertRZ = min(rz,waterT);
		float fresnel = 0;
		float3 refractCol = float3(0.,0.,0.);
		bool reflected = false;
		// 视线触及水平面
		if(rz >= waterT && rd.y < -0.01){
			float3 waterPos = ro + rd * waterT; 
			float3 nor = WaterNormal(waterPos,waterT);
			//float3 nor = float3(0.,1.,0.);
			float ndotr = dot(nor,-rd);
			fresnel = pow(1.0-abs(ndotr),6.); 
			float3 diff = pow(dot(nor,_LightDir) * 0.4 + 0.6,3.);
			// 水体自身颜色
			float3 waterCol = _BaseWaterColor + diff * _LightWaterColor * 0.12; 
			float transPer = pow(1.0-clamp( rz - waterT,0,waterTranDeep)/waterTranDeep,3.);
			// 水体折射颜色
			float3 bgCol = RenderMountain(ro,rd +nor* clamp(1.-dot(rd,-nor),0.,1.),rz);
			refractCol = lerp(waterCol,bgCol,transPer);

			ro = waterPos;
			rd = reflect( rd, nor);
			rz = RaycastTerrain(ro,rd);
			reflected = true;
			col = refractCol; }   
水体着色计算中，先确定像素点从rd方向射出，到水面的距离waterT，再通过与步进的结果值rz比较来判断当前是否进行水面的着色计算。折射计算过程中，根据水面法线对rd方向进行偏移，然后作为输入对山体进行取色，以透明度值作为插值进行混合：  

	float3 bgCol = RenderMountain(ro,rd +nor* clamp(1.-dot(rd,-nor),0.,1.),rz);
			refractCol = lerp(waterCol,bgCol,transPer);  
计算完水体折射后，根据水面法线方向对之前的rd进行反射计算得到新的步进方向，再一次做光线步进，根据得到的步进结果rz来进行水体的反射着色：   

	if(rz >= maxT){
		col = Sky( ro, rd,_LightDir);}
	else{
		col = RenderMountain(ro,rd,rz);}
	if( reflected == true ) {
		col = lerp(refractCol,col,fresnel);
		float spec=  pow(max(dot(rd,_LightDir),0.0),128.) * 3.;
		col += float3(spec,spec,spec);}      
此前未被水面覆盖的区域，不会进入水面的着色计算过程，进行到此处的判断则是计算水体区域外的山体和天空；而在水面覆盖的区域，在进行水面的着色计算后，进行到此处，由于更改了rd，计算得到新的rz，计算的是水面的反射颜色，最后通过frenel系数对折射和反射进行插值混合，得到最终颜色。

参考：[https://blog.csdn.net/tjw02241035621611/article/details/80108319](https://blog.csdn.net/tjw02241035621611/article/details/80108319)


