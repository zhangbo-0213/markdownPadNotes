# RayMarch简单实践02 Simple-SDF-RayMarchingScene #
**写在前面**     
在上一篇笔记[RayMarch简单实践01](https://zhuanlan.zhihu.com/p/53491836)中，通过从摄像机发出射线，并采用步进的方式求解场景中的碰撞点，利用碰撞点信息进行着色，在屏幕上绘制了一个Sphere和Plane，本篇笔记中将丰富上一个场景，并做相对复杂的光照处理。采取和上篇中绘制Sphere,Plane相同的方式绘制更多的三维图形，因此需要引入更多距离计算函数，也就是SDF(Sign Distance Functions)，这些函数用来计算空间中点到目标模型的最短距离，结合RayMarch的步进，可以知道当前的射线是否碰到目标物体。SDF函数的实现来源于Inigo’s Quilez大神的[这篇博客](http://iquilezles.org/www/articles/distfunctions/distfunctions.htm)。      

	//----------集合操作----------
	//差集操作
	float OpS(float d1,float d2){
	return max(-d2,d1);
		}
	//交集操作
	float OpI(float d1,float d2){
		return max(d1,d2);
	} 
	//并集操作
	float OpU(float d1,float d2){
		return min(d1,d2);
	}
	//平滑并集
	float OpSmoothUnion( float d1, float d2, float k ) {
    float h = clamp( 0.5 + 0.5*(d2-d1)/k, 0.0, 1.0 );
    return lerp( d2, d1, h ) - k*h*(1.0-h); }
	//平滑差集
	float OpSmoothSubtraction( float d1, float d2, float k ) {
    float h = clamp( 0.5 - 0.5*(d2+d1)/k, 0.0, 1.0 );
    return lerp( d2, -d1, h ) + k*h*(1.0-h); }
	//平滑交集
	float OpSmoothIntersection( float d1, float d2, float k ) {
    float h = clamp( 0.5 - 0.5*(d2-d1)/k, 0.0, 1.0 );
    return lerp( d2, d1, h ) + k*h*(1.0-h); }

	//重复操作
	float3 OpRep(float3 p,float3 c){
	return fmod(p,c)-0.5*c;
	}
	//重复操作;
	float2 OpRep( float2 p, float2 c )
	{
    return fmod(p,c)-0.5*c;
	}
	//扭曲操作
	float3 OpTwist(float3 p){
	float c=cos(10.0*p.y+10.0);
	float s=sin(10.0*p.y+10.0);
	float2x2 m=float2x2(c,-s,s,c);
	return float3(mul(m,p.xz),p.y);
	}

	//平面 p为 点到图形中心相对向量
	float SdPlane(float3 p){
	return p.y;
	}
	//球体 p为 点到图形中心相对向量 s为球体半径
	float SdSphere(float3 p,float s){
	return length(p)-s;
	}
	//立方体 p为 点坐标 b为图形本身参数（长，宽，高）
	float SdBox(float3 p,float3 b){
	return length(max(abs(p)-b,0.0));
	}
	//圆角立方体 p为点坐标 b为图形本身参数（长，宽，高） r为圆角半径
	float SdRoundBox(float3 p,float3 b,float r){
	return length(max(abs(p)-b,0.0))-r;
	}
	//椭球
	//椭球面完全被封闭在一个长方体的内部，这个长方体由六个平面：x=±a, y=±b, z=±c所围成.
	//参数 p 点到图形中心相对向量,r 为包裹椭球长方体的本身参数
	float SdEllipsoid(in float3 p,in float3 r){
	return (length(p/r)-1.0)*min(min(r.x,r.y),r.z);
	}
	//圆环
	//参数 p 为点坐标，t为圆环内外径中心处坐标
	float SdTorus(float3 p,float2 t){
	return length(float2(length(p.xz)-t.x,p.y))-t.y;
	}
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         
	//胶囊体 
	//参数 p为到胶囊体中心相对向量 ，a，b为胶囊体两球心坐标,r为半径
	float SdCapsule(float3 p,float3 a,float3 b,float r){
    p=p+(a+b)/2.0;
	float3 pa=p-a,ba=b-a;
	float h=clamp(dot(pa,ba)/dot(ba,ba),0.0,1.0);
	return length(pa-ba*h)-r;
	}
	//等边三角形（2维SDF）
	float SdEquilateralTriangle(  in float2 p )
	{
    const float k = 1.73205;//sqrt(3.0);
    p.x = abs(p.x) - 1.0;
    p.y = p.y + 1.0/k;
    if( p.x + k*p.y > 0.0 ) p = float2( p.x - k*p.y, -k*p.x - p.y )/2.0;
    p.x += 2.0 - 2.0*clamp( (p.x+2.0)/2.0, 0.0, 1.0 );
    return -length(p)*sign(p.y);
	}
	//三棱柱
	float SdTriPrism( float3 p, float2 h )
	{
    float3 q = abs(p);
    float d1 = q.z-h.y;
	#if 1
    // distance bound
    float d2 = max(q.x*0.866025+p.y*0.5,-p.y)-h.x*0.5;
	#else
    // correct distance
    h.x *= 0.866025;
    float d2 = SdEquilateralTriangle(p.xy/h.x)*h.x;
	#endif
    return length(max(float2(d1,d2),0.0)) + min(max(d1,d2), 0.);
	}    
	//圆柱
	float SdCylinder( float3 p, float2 h )
	{
  	float2 d = abs(float2(length(p.xz),p.y)) - h;
  	return min(max(d.x,d.y),0.0) + length(max(d,0.0));
	}

	//圆锥
	float SdCone( in float3 p, in float3 c )
	{
    float2 q = float2( length(p.xz), p.y );
    float d1 = -q.y-c.z;
    float d2 = max( dot(q,c.xy), q.y);
    return length(max(float2(d1,d2),0.0)) + min(max(d1,d2), 0.);
	}

	//圆台
	float SdConeSection( in float3 p, in float h, in float r1, in float r2 )
	{
    float d1 = -p.y - h;
    float q = p.y - h;
    float si = 0.5*(r1-r2)/h;
    float d2 = max( sqrt( dot(p.xz,p.xz)*(1.0-si*si)) + q*si - r2, q );
    return length(max(float2(d1,d2),0.0)) + min(max(d1,d2), 0.);
	}

	//4棱锥
	float SdPryamid4(float3 p, float3 h ) // h = { cos a, sin a, height }
	{
    // Tetrahedron = Octahedron - Cube
    float box = SdBox( p - float3(0,-2.0*h.z,0), float3(2.0*h.z,2.0*h.z,2.0*h.z) );
 
    float d = 0.0;
    d = max( d, abs( dot(p, float3( -h.x, h.y, 0 )) ));
    d = max( d, abs( dot(p, float3(  h.x, h.y, 0 )) ));
    d = max( d, abs( dot(p, float3(  0, h.y, h.x )) ));
    d = max( d, abs( dot(p, float3(  0, h.y,-h.x )) ));
    float octa = d - h.z;
    return max(-box,octa); // Subtraction
 	}

 	//倒角环
 	float SdTorus82( float3 p, float2 t )
	{
    float2 q = float2(length2(p.xz)-t.x,p.y);
    return length8(q)-t.y;
	}
	//倒角环
	float SdTorus88( float3 p, float2 t )
	{
    float2 q = float2(length8(p.xz)-t.x,p.y);
    return length8(q)-t.y;
	}

	//倒角柱体
	float SdCylinder6( float3 p, float2 h )
	{
    return max( length6(p.xz)-h.x, abs(p.y)-h.y );
	}

	//圆柱切片
	float SdClipCylinder( float3 pos, float3 h)
	{
    pos.x += h.x*h.z*2. - h.x;
   	float cy = SdCylinder(pos,h.xy);
   	float bx = SdBox(pos-float3(h.x*(1.+2.*h.z),0.,0.),float3(h.x*2.,h.y+0.3,h.x*2.));
   	return OpS(cy,bx);
	}
上面列出基本几何图形和集合操作SDF，场景中将使用这些SDF绘制基本几何体和变体。   

**简单实践**    
除了需要用到SDF和更多光照计算外，其余部分与上篇大致相同，因此可以将重复部分抽取，使用时直接引入：           
![](https://i.imgur.com/gcAJrdy.png)         
SDF.cginc里就是各种距离计算函数，集合操作等，还有像变形，镜像等都可以后续添加   
BaseFrame.cginc中是顶点着色器和片元着色器的处理过程，因为同上一篇中一样，顶点着色器主要是传递摄像机4个角向量供片元着色器进行插值计算射线方向rd,而片元着色器里需要处理的过程主要是对深度纹理采样，方便后续的深度值比较，同时对每个像素做步进处理，计算碰撞点的光照，光照部分可以单独实现，因此这两部分可以作为统一的流程抽出，另外可以抽出的部分是常用的几个步进函数，包括法线计算、阴影计算、光线步进计算：     

	//法线计算
	#define CALC_NORMAL(pos,rz,MAP_FUNC)\
	float2 e=float2(1.0,-1.0)*0.5773*0.002*rz;\
	return normalize(e.xyy*MAP_FUNC(pos+e.xyy).x+\
					 e.yyx*MAP_FUNC(pos+e.yyx).x+\
					 e.yxy*MAP_FUNC(pos+e.yxy).x+\
					 e.xxx*MAP_FUNC(pos+e.xxx).x);     

	
	//阴影计算					 
	#define SOFT_SHADOW(ro,rd,maxH,MAP_FUNC)\
	float res=1.0;\
	float t=0.001;\
	for(int i=0;i<80;i++){\
		float3 p=ro+t*rd;\
		float h=MAP_FUNC(p).x;\
		res=min(res,16.0*h/t);\
		t+=h;\
		if(res<0.001||p.y>maxH) break;\
	}\
	return clamp(res,0.0,1.0);

    
	//光线步进计算
	#define _MACRO_RAY_CAST(ro,rd,tmax,MAP_FUNC)\
	float t=0.1;\
	float m=-1.0;\
	for(int i=0;i<256;i++){\
		float precis=0.0005*t;\
		float2 res=MAP_FUNC(ro+t*rd);\
		if(res.x<precis||t>tmax)break;\
		t+=0.5*res.x;\
		m=res.y;\
	}\
	if(t>tmax) m=-1.0;\
	return float2(t,m);
同时在BaseFrame.cginc中声明 ProcessRayMarch 函数：   

	float4 ProcessRayMarch(float2 uv,float3 ro,float3 rd,inout float sceneDep,float4 sceneCol);	   
由于光照的计算希望通过自行定义实现，因此这里只做声明，实现放到后面   

DefaultRedner.cginc部分是关于光照计算的默认实现方式：      

	float3 MatCol(float matID,float3 pos,float3 nor);
	float2 Map(in float3 pos);
	float3 Render(in float3 ro,in float3 rd);

	float2 RayCast(float3 ro,float3 rd){
	_MACRO_RAY_CAST(ro,rd,1000.0,Map);
	}

	float3 CalcNormal(in float3 pos,float rz){
	_MACRO_CALC_NORMAL(pos,rz,Map);
	}

	float SoftShadow(in float3 ro,in float3 rd,float tmax){
	_MACRO_SOFT_SHADOW(ro,rd,tmax,Map);
	}

	// Ambient Occlusion: 环境光吸收/遮蔽
	float CalcAO(in float3 pos,in float3 nor){
	float occ = 0.0;
    float sca = 1.0;
    for( int i=0; i<5; i++ )
    {
        float hr = 0.01 + 0.12*float(i)/4.0;
        float3 aopos =  nor * hr + pos;
        float dd = Map( aopos ).x;
        occ += -(dd-hr)*sca;
        sca *= 0.95;
    }
    return clamp( 1.0 - 3.0*occ, 0.0, 1.0 );
	}

	#ifdef DEFAULT_RENDER
	float3 Render(in float3 ro,in float3 rd){
	float3 col=float3(0.7,0.8,1.0)+rd.y*0.8;
	float2 res=RayCast(ro,rd);
	float t=res.x;
	float m=res.y;
	//碰撞到场景中的物体
	if(m>-0.5){
		float3 pos=ro+t*rd;
		float3 nor=CalcNormal(pos,0.01);
		float3 ref=reflect(rd,nor);
		col=MatCol(m,pos,nor);

		//Lighting 光照着色计算
		float occ=CalcAO(pos,nor); //环境光遮蔽
		float3 lig=normalize(float3(-0.4,0.7,-0.6));
		float3 hal=normalize(lig-rd); //halfLitDir
		float amb=clamp(0.5+0.5*nor.y,0.0,1.0);
		float dif=clamp(dot(nor,lig),0.0,1.0);
		float bac=clamp(dot(nor,normalize(float3(-lig.x,0.0,-lig.z))),0.0,1.0)*clamp(1.0-pos.y,0.0,1.0);//计算背光
		float dom=smoothstep(-0.01,0.01,ref.y);//计算反射方向上的影响因子
		float fre=pow(clamp(1.0+dot(nor,rd),0.0,1.0),2.0);//菲涅尔边缘高光

		dif*=SoftShadow(pos,lig,2.5);//沿着光线方向上指定范围内有物体，添加阴影
		dom*=SoftShadow(pos,ref,2.5);//沿着光线反射方向上指定范围内若有物体，添加阴影

		float spe=pow(clamp(dot(nor,hal),0.0,1.0),16.0)*dif*(0.04+0.96*pow(clamp(1.0+dot(hal,rd),0.0,1.0),5.0));//更复杂的高光计算模型

		float3 lin=float3(0.0,0.0,0.0);
		lin+=1.30*dif*float3(1.00,0.80,0.55);
		lin+=0.40*amb*float3(0.40,0.60,1.00)*occ;
		lin+=0.50*dom*float3(0.30,0.70,1.00)*occ;
		lin+=0.50*bac*float3(0.25,0.25,0.25)*occ;
		lin+=0.25*fre*float3(1.00,1.00,1.00)*occ;
		col=col*lin;
		col+=10.00*spe*float3(1.00,0,0);

		//根据光线检测距离非线性插值背景
		col=lerp(col,float3(0.8,0.9,1.0),1.0-exp(-0.0002*t*t*t));	
		}
		return float3(clamp(col,0.0,1.0));
	}

	#ifdef DEFAULT_MAT_COL
	float3 MatCol(float matID,float3 pos,float3 nor){
	float3 col=0.45+0.35*sin(float3(0.05,0.08,0.10)*(matID-1.0));
	if(matID<1.5){
		//CheckersGradBox 棋盘纹理绘制
		float f=CheckersGradBox(5.0*pos.xz);
		col=0.3+f*float3(0.1,0.1,0.1);
	}
	return col;
	}
	#endif

	#ifdef DEFAULT_PROCESS_FRAG
	float4 ProcessRayMarch(float2 uv,float3 ro,float3 rd,inout float sceneDep,float4 sceneCol){
	//render
	float3 col=Render(ro,rd);
	//gamma矫正
	col=pow(col,float3(0.4545,0.4545,0.4545));
	sceneCol.xyz=col;
	return sceneCol;
	}
	#endif	  
这里默认的光照计算包括 环境光遮蔽，漫反射，高光，反射，背光和边缘高光等部分，相较于上一篇的漫反射丰富了一些。除了实现了默认的光照计算，这里也实现了ProcessRayMarch。Map函数只做了声明，实现部分放在最终的shader中。    
SDFScene.shader中引入之前这些.cginc文件，并实现Map函数：     

	Pass
		{
			ZTest Always Cull off ZWrite off
			CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag

			#define DEFAULT_RENDER
			#define DEFAULT_MAT_COL
			#define DEFAULT_PROCESS_FRAG
			#include "DefaultRender.cginc"

			float3 SelfRotXZ(float3 pos,float speedSize,float phase=0.0){
				return float3(
					mul(Rot2DRad(_Time.y*speedSize+phase),pos.xz).x,
					pos.y,
					mul(Rot2DRad(_Time.y*speedSize+phase),pos.xz).y);
			}

			float3 SelfRotYZ(float3 pos,float speedSize,float phase=0.0){
				return float3(
				    pos.x,
					mul(Rot2DRad(_Time.y*speedSize+phase),pos.yz).x,
					mul(Rot2DRad(_Time.y*speedSize+phase),pos.yz).y);
			}

			float2 Map( in float3 pos )
			{
				float2 res = OpU( float2( SdPlane(pos), 1.0 ),float2( SdSphere(SelfRotXZ((pos-float3(0.0,0.30, 0.0)),1.0), 0.25 ), 46.9 ) );
				res = OpU( res, float2( SdBox(SelfRotXZ((pos-float3( 1.0,0.30, 0.0)),1.0), float3(0.25,0.25,0.25) ), 3.0 ) );
				res = OpU( res, float2( SdRoundBox(SelfRotXZ((pos-float3( 1.0,0.30, 1.0)),1.0,0.5), float3(0.15,0.15,0.15), 0.1 ), 41.0 ) );
				res = OpU( res, float2( SdTorus(SelfRotYZ((pos-float3( 0.0,0.25, 1.0)),1.0), float2(0.20,0.05) ), 25.0 ) );
				res = OpU( res, float2( SdCapsule(SelfRotXZ((pos-(float3(-1.3,0.10,-0.1)+float3(-0.8,0.50,0.2))/2.0),1.0),float3(-1.3,0.10,-0.1), float3(-0.8,0.50,0.2), 0.1  ), 31.9 ) );
				res = OpU( res, float2( SdTriPrism(SelfRotXZ((pos-float3(-1.0,0.30,-1.0)),1.0), float2(0.25,0.05) ),43.5 ) );
				res = OpU( res, float2( SdCylinder(SelfRotYZ((pos-float3( 1.0,0.30,-1.0)),1.0), float2(0.1,0.2) ), 8.0 ) );
				res = OpU( res, float2( SdCone(SelfRotXZ((pos-float3( 0.0,0.50,-1.0)),1.0), float3(0.8,0.6,0.3) ), 55.0 ) );
				res = OpU( res, float2( SdTorus82(SelfRotYZ((pos-float3( 0.0,0.25,2.0)),-1.0,-0.3), float2(0.20,0.05) ),50.0 ) );
				res = OpU( res, float2( SdTorus88(SelfRotYZ((pos-float3( -1.0,0.25,2.0)),1.0,0.2), float2(0.20,0.05) ),43.0 ) );
				res = OpU( res, float2( SdCylinder6(SelfRotXZ((pos-float3( 1.0,0.30,2.0)),1.0), float2(0.1,0.2) ), 12.0 ) );
				res = OpU( res, float2( SdHexPrism(SelfRotXZ((pos-float3( -1.0,0.30,1.0)),1.0), float2(0.25,0.05) ),17.0 ) );
				res = OpU( res, float2( SdPryamid4(SelfRotXZ((pos-float3( -1.0,0.50,-2.0)),1.0), float3(0.8,0.6,0.25) ),37.0 ) );
				res = OpU( res, float2( OpS(SdRoundBox(SelfRotXZ((pos-float3( -2.0,0.30,1.0)),1.0), float3(0.15,0.15,0.15),0.05), SdSphere(pos-float3(-2.0,0.3, 1.0), 0.25-0.02*sin(_Time.y))), 13.0 ) );
				res = OpU( res, float2( OpS( SdTorus82(pos-float3(-2.0,0.2, 0.0), float2(0.20,0.1)),SdCylinder(OpRep( float3(atan2(abs(pos.x+2.0),abs(pos.z))/6.2831, pos.y, 0.02+0.5*length(pos-float3(-2.0,0.2, 0.0))), float3(0.05,1.0,0.05)), float2(0.02,0.6))), 51.0 ) );
				res = OpU( res, float2( 0.5*SdSphere(pos-float3(-2.0,0.25,-1.0), 0.2) + 0.03*sin(50.0*pos.x+_Time.y)*sin(50.0*pos.y+_Time.y)*sin(50.0*pos.z+_Time.y), 65.0 ) );
				res = OpU( res, float2( 0.5*SdTorus(OpTwist(SelfRotXZ((pos-float3( -2.0,0.30,2.0)),1.0)),float2(0.20+0.05*sin(_Time.y),0.05)), 46.7 ) );
				res = OpU( res, float2( SdConeSection(pos-float3( 0.0,0.35,-2.0), 0.15, 0.2, 0.1 ), 13.67 ) );
				res = OpU( res, float2( SdEllipsoid(SelfRotXZ((pos-float3( 1.0,0.35,-2.0)),1.0), float3(0.15, 0.2, 0.1) ), 43.17 ) );
                res = OpU( res, float2( OpSmoothUnion(SdBox(SelfRotXZ((pos-float3( -2.0,0.30,-2.0)),0.5), float3(0.25,0.25,0.25) ), SdSphere(pos-float3( -2.0,0.6+0.35*sin(_Time.y),-2.0), 0.15 ),0.5),42.0));
				return res;
			}
		ENDCG
		}
通过抽取流程化的部分，Pass中只需实现需要自定义的部分就可以了，比如这里的Map函数，Map函数里主要使用SDF中的集合和基本集合体操作绘制几何体及变体，同时增加了两个自转函数，使这些几何体动起来。    
最终实现效果：        
![](https://i.imgur.com/ZRoTrYh.png)    

参考：        
[https://www.zhihu.com/search?q=%E5%85%89%E7%BA%BF%E8%BF%BD%E8%B8%AA&type=content](https://www.zhihu.com/search?q=%E5%85%89%E7%BA%BF%E8%BF%BD%E8%B8%AA&type=content)       
[https://blog.csdn.net/qq_16756235/article/details/75195994](https://blog.csdn.net/qq_16756235/article/details/75195994)   

	
