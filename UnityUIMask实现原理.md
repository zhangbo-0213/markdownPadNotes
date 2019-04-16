### Unity UI Mask实现原理 ###
Mask的实现思路：
与Image组件配合工作，根据Image的覆盖区域来定位显示范围，所有该Image的**子级UI元素**，超出此区域的部分会被隐藏（**包括UI的交互事件**）   

Mask的实现原理：

1. Mask会赋予Image一个特殊的材质，这个材质会给Image的每个像素点进行标记，将标记结果存放在一个缓存内（这个缓存叫做 Stencil Buffer）
2. 当子级UI进行渲染的时候会去检查这个 Stencil Buffer内的标记，如果当前覆盖的区域存在标记（即该区域在Image的覆盖范围内），进行渲染，否则不渲染    

UGUI自身实现的Mask组件实现遮罩，需要子级UI进行配合，Unity内置的UI组件继承自MaskableGraphic,该类是Mask的配合实现者：       
相关代码：      

		public virtual Material GetModifiedMaterial(Material baseMaterial)
    {
        var toUse = baseMaterial;
 
        if (m_ShouldRecalculateStencil)
        {
            var rootCanvas = MaskUtilities.FindRootSortOverrideCanvas(transform);
            m_StencilValue = maskable ? MaskUtilities.GetStencilDepth(transform, rootCanvas) : 0;
            m_ShouldRecalculateStencil = false;
        }
 
        // if we have a enabled Mask component then it will
        // generate the mask material. This is an optimisation
        // it adds some coupling between components though :(
        Mask maskComponent = GetComponent<Mask>();
        if (m_StencilValue > 0 && (maskComponent == null || !maskComponent.IsActive()))
        {
            var maskMat = StencilMaterial.Add(toUse, (1 << m_StencilValue) - 1, StencilOp.Keep, CompareFunction.Equal, ColorWriteMask.All, (1 << m_StencilValue) - 1, 0);
            StencilMaterial.Remove(m_MaskMaterial);
            m_MaskMaterial = maskMat;
            toUse = m_MaskMaterial;
        }
        return toUse;
    }   

这里的核心代码是为UI生成一个材质：   

	var maskMat = StencilMaterial.Add(toUse, (1 << m_StencilValue) - 1, StencilOp.Keep, CompareFunction.Equal, ColorWriteMask.All, (1 << m_StencilValue) - 1, 0);     
这个材质在渲染时会去取StencilBuffer的值，并通过CompareFunction这个枚举变量所指定的比较方法进行比较，符合比较结果会进行渲染，否则不进行渲染。   
因此可以通过改写此方法中的比较方式，来达到类似挖孔的效果，即超出Image区域的显示，Image内的子UI不显示：      

	public class HoleImage : Image {
    public override Material GetModifiedMaterial(Material baseMaterial)
    {
        var toUse = baseMaterial;
 
        if (m_ShouldRecalculateStencil)
        {
            var rootCanvas = MaskUtilities.FindRootSortOverrideCanvas(transform);
            m_StencilValue = maskable ? MaskUtilities.GetStencilDepth(transform, rootCanvas) : 0;
            m_ShouldRecalculateStencil = false;
        }
 
        // if we have a enabled Mask component then it will
        // generate the mask material. This is an optimisation
        // it adds some coupling between components though :(
        Mask maskComponent = GetComponent<Mask>();
        if (m_StencilValue > 0 && (maskComponent == null || !maskComponent.IsActive()))
        {
            var maskMat = StencilMaterial.Add(toUse, (1 << m_StencilValue) - 1, StencilOp.Keep, CompareFunction.NotEqual, ColorWriteMask.All, (1 << m_StencilValue) - 1, 0);
            StencilMaterial.Remove(m_MaskMaterial);
            m_MaskMaterial = maskMat;
            toUse = m_MaskMaterial;
        }
        return toUse;
    }
	}   
将 CompareFunction.Equal 改为 CompareFunction.NotEqual 实现遮罩的反效果，挖孔。 
参考： [UnityGUI扩展实例：图片挖洞效果 Mask的反向实现](https://www.cnblogs.com/j349900963/p/8340571.html "UnityGUI扩展实例：图片挖洞效果 Mask的反向实现")   

修改 CompareFunction 的比较方式也可以通过 UI 材质中的参数值进行：    

UI材质的默认参数：   
![](https://i.imgur.com/IF2TbE1.png)      

Stencil Comparison：比较值    
Stencil ID：UI ID值   
Stencil Operation：缓冲区操作值       
Stencil Write Mask：遮罩写入     
Stencil Read Mask：遮罩读取     

在渲染时，将stencil buffer的值与ReadMask与运算，然后与Ref值进行Comp比较，结果为true时进行Pass操作，否则进行Fail操作，操作值写入stencil buffer前先与WriteMask与运算。    
在UI Shader中 参数(属性)值的对应设置：    

Ref [ _Stencil]   
Comp [ _StencilComp]     
Pass [ _StencilOp]   
ReadMask [ _StencilReadMask]   
WriteMask [ _StencilWriteMask]  

**Ref**：   
用来设定参考值referenceValue，这个值将用来与模板缓冲中的值进行比较。referenceValue是一个取值范围位0-255的整数。        
**Comp**：   
是定义参考值（referenceValue）与缓冲值（stencilBufferValue）比较的操作函数，默认值：always       
ComparisonFunction比较操作通过Comp命令定义，公式左右两边的结果将通过它进行判断，其取值及其意义如下面列表所示：      
![](https://i.imgur.com/uYCPiRp.png)      
**Pass**：     
是定义当模板测试（和深度测试）通过时，则根据（stencilOperation值）对模板缓冲值（stencilBufferValue）进行处理，默认值：keep    
stencilOperation命令值列表：     
![](https://i.imgur.com/w8pCCHU.png)          
**ReadMask** ：    
从字面意思的理解就是读遮罩，readMask将和referenceValue以及stencilBufferValue进行按位与（&）操作，readMask取值范围也是0-255的整数，默认值为255，二进制位11111111，即读取的时候不对referenceValue和stencilBufferValue产生效果，读取的还是原始值。     
**WriteMask**：          
是当写入模板缓冲时进行掩码操作（按位与&），writeMask取值范围是0-255的整数，默认值也是255，即当修改stencilBufferValue值时，写入的仍然是原始值。     

和深度测试一样，在unity中，每个像素的模板测试也有它自己一套独立的依据，具体公式如下：


if（referenceValue&readMask comparisonFunction stencilBufferValue&readMask）

通过像素

else

抛弃像素

通过的像素操作则根据 stencilOperation命令值 来确定，并更新模板缓冲内的值：       

在更新模板缓冲值的时候，也有writeMask进行掩码操作，用来对特定的位进行写入和屏蔽，默认值为255（11111111），即所有位数全部写入，不进行屏蔽操作。

参考：[Stencil Buffer&Stencil Test](https://blog.csdn.net/u011047171/article/details/46928463)    

通过以上的内容，下面来解释 [浅谈Unity uGUI Mask组件实现原理](https://www.sohu.com/a/211665096_99940808) 这篇博客中的UI所表现出的挖孔和遮罩效果的原因（一开始就是没明白，所以找了上面一些文档来看）      

![](https://i.imgur.com/5fGJQF1.png)       
场景中的UI均在同一层级，不同的UI后缀，即所用的自定义UI材质：     
![](https://i.imgur.com/xttxdn3.png)           

蓝色Image 首先绘制，根据UIHole的材质参数：
Comp值为6，对应的比较方法为： NotEqual   
在UI被绘制前，对应像素点默认的Stencil值为0，因此蓝色UI通过模板绘制，被绘制；      
Stencil Operation值为0，对应操作为 Keep ，也就是蓝色UI对应像素点的Stencil值保持之前的值，为0      

然后绘制stencil-stencil的UI，Comp值为8，对应的比较方法为：Always       
也就是该UI无论如何都会通过测试，被绘制，那为啥看不见呢？注意这里有个ColorMask的值为0，它会屏蔽UI的颜色输出，实际上stencil-stencil是被绘制了，只是颜色没有输出         
Stencil Operation值为2，对应操作为Replace，也就是材质参数中的Stencil ID 的值被写入 StencilBuffer中 ，所以在绘制完stencil-stencil后，对应UI位置的像素点的值为2          

绘制white-hole的UI，Comp的值为6，对应的比较方法：NotEqual      
由于UIHole材质的Stencil ID值为2，而只有与StencilBuffer的值不相等的对应像素点才会被绘制，所以呈现出white-hole被stencil-stencil挖洞的效果，后面红色的red-hole也是同理    

绘制green-mask的UI，Comp的值为3，对应的比较方法为：Equal    
由于UIHole材质的Stencil ID值为2，而只有与StencilBuffer的值相等的对应像素点才会被绘制，所以呈现出只有在stencil-stencil区域内才显示的遮罩效果，后面的yellow-mask也是同理










	
