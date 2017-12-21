## 数据结构学习笔记 ##

### 数据结构相关概念 ###
数据结构的相关概念：         
![](https://i.imgur.com/OSvaWbm.png)        
![](https://i.imgur.com/iGXBKz0.png)         

### 初识算法及相关概念 ###

- **算法**      
解决特定问题的求解步骤及描述，在计算机中为指令的有限序列，每条指令表示一个或多个操作。

- **算法特性**     
有穷性、确定性、可行性、输入、输出      

- **算法设计要求**     
正确性、可读性、健壮性、高效率和低存储量需求         

- **算法时间复杂度及大O阶推导**         
大O阶推导：     
1.使用常数1代替运行时间中的所有常数加法     
2.修改后的运行次数函数中，只保留最高阶项         
3.如果最高阶项存在且系数不为1，去除该项的系数        
常见的时间复杂度耗时排列：          
O(1)<O(logn)<O(n)<O(nlogn)<O(n^2)<O(n^3)<O(2^n)<O(!n)<O(n^n)          

一般而言，所指的时间复杂度通常是指最坏情况的耗时时间。      

### 线性表 ###
线性表：零个或多个数据元素的有限序列          
线性表的两种存储结构：**顺序存储&链式存储**         

**单链表结构&顺序存储结构对比**        
![](https://i.imgur.com/IWJzp1A.png)        

- 若线性表需要频繁的读取而插入和删除操作较少时，顺序存储结构更加合适，若有频繁的插入和删除操作，则单链表结构更加合适。
- 当线性表元素变化较大或者不知道有多大时，单链表结构更加合适，不需要考虑内存预先存储空间的大小问题，而如果已知具体长度，则使用顺序存储效率较高       

**静态链表**        
用数组描述的链表叫做静态链表     
静态链表的优缺点：       
![](https://i.imgur.com/Wn4HsVV.png)               
静态链表实际上是给没有指针的高级语言设计的一种实现单链表的方法，尽管存在一定缺陷，其设计思想十分巧妙。     

**栈与队列**     
**栈**是限定仅在表尾进行插入和删除操作的线性表      
**队列**是只允许在一端插入数据在另一端删除数据的线性表  

**顺序栈与链栈对比**           

- 插入删除时间复杂度均为O[1]  
- 对于空间复杂度，顺序栈需要事先确定长度，会存在内存空间浪费问题 ；链栈存取定位方便，但需要指针域增大存储开销。如果栈的长度不确定，使用链栈，反之使用顺序栈比较合适。   

**串**       
串是指零个或多个字符组成的有限序列，又叫字符串。      

串的顺序存储一般使用定长数组进行定义，对于字符串操作存在的溢出问题，串值的存储空间在执行过程中动态分配**堆内存**，由动态分配函数malloc()和free()来管理      

串的链式存储结构除了在串的连接操作会方便一些，总体不如顺序结构灵活，性能也不如顺序存储结构      

### 树 ###
树是n个结点的有限集，当n=0时，为空树。在任意一个非空树中有且仅有一个特定的称为**根**的结点；当n>1时，其余结点可以分为m个互不相交的有限集，每一个集合本身又是一棵树，称为**根的子树**     

数的结点包含一个数据元素&**若干个**指向其子树的分支。结点拥有的子树的数目称为 **结点的度**。度为0的结点称为叶节点或终端结点，度不为0的结点，称为分支结点，分支结点中除了根节点，其他结点也称为内部节点。数的度为树内所有结点的最大值。   

结点的层次从根结点开始定义，根为第一层，根的孩子为第二层。树中结点的最大层次称为树的深度或高度。    

**线性表与树结构的差异**           
![](https://i.imgur.com/cc8RHTR.png)       

**树的抽象数据类型ADT**     
![](https://i.imgur.com/aZUXbhq.png)      
树的存储结构       
对于树这种存在一对多的情况，单纯使用顺序存储无法满足其逻辑关系。结合顺序存储和链式存储可以实现对树的存储结构的要求。   
常见的三种不同的表示法：双亲表示法、孩子表示法（双亲-孩子结合的表示法）、孩子兄弟表示法。   

![](https://i.imgur.com/kZ3Un8A.png)        
双亲表示法： 
通过一定长度的结点数组存储结点（结点存储结点数据和双亲下标）    
![](https://i.imgur.com/NcKfbG8.png)      
双亲表示法中根据结点的parent可以找到其双亲，但是无法找到结点的子结点，除非是遍历整个树。可以对结点的数据域进行扩展，增加长子索引。可以根据需求继续扩展结点数据域。          
![](https://i.imgur.com/0kcbdwO.png)       

孩子表示法：         
孩子表示法为在结点数组中，每个结点形成一个单链表的结构，链表中的下一个元素为该结点的兄弟。        
![](https://i.imgur.com/0ONs0Bp.png)        

兄弟表示法      
兄弟表示法中，每个结点如果存在长子结点，有且只有一个，而结点紧邻右侧的兄弟若存在，有且只有一个。这样在结点数组中，每个结点的数据域中有两个指针，分别指向第一个长子和它右侧的兄弟。    
![](https://i.imgur.com/i0s5hrE.png)        
这种表示法如果要找到双亲还需要再添加指针域指向双亲，不过这种表示法将复杂树转为了二叉树，这样可以利用二叉树的特性和算法来处理相应的操作。     

**二叉树**        
由n个结点构成的有限集，当n=0时，为空树，当n>0时，由一个根结点和两颗互不相交的，分别称为根结点的左子树和右子树的二叉树构成。    

二叉树特点：         


- 每个结点**最多**只有两颗子树，二叉树中不存在度大于2的结点，可以是两颗子树，也可以是一颗或者没有子树。      
- 左子树和右子树有顺序，次序不能任意颠倒。        
- 即使树中某结点只有一棵子树，也要区分是左子树还是右子树。      

二叉树的五种基本形态：      


- 空二叉树
- 只有一个根结点
- 根结点只有左子树
- 根结点只有右子树
- 根结点既有左子树又有右子树       

特殊二叉树：       


- 斜树        
所有结点都只有左子树的二叉树称为左斜树，所有结点都有右子树的二叉树称为右斜树。斜树的结点数即为该树的深度。      


- 满二叉树  
一颗二叉树中，所有分支的结点都存在左子树和右子树，并且所有的叶子都在同一层上，这样的二叉树称为满二叉树。     
**满二叉树特点**     
叶子只能出现在最下一层        
非叶子的结点的度一定是2
同样深度的树中，满二叉树的结点个数最多，叶子数最多。    

- 完全二叉树       
对一颗n各结点的二叉树按层序编号，若编号为i的结点与对应深度的满二叉树中的编号一致，则这样的二叉树称为完全二叉树。即满二叉树一定是完全二叉树，而完全二叉树不一定是满二叉树。完全二叉树是满二叉树的子集。       
**完全二叉树特点**       
叶子结点只能是最下两层        
最下层的叶子一定集中在左部连续位置      
倒数第二层，若有叶子结点，一定都在右部连续位置     
如果结点的度为1，该结点只有左孩子，不存在只有右子树的情况        
同样结点数的二叉树，完全二叉树的深度最小    

**二叉树的性质**      


- 在二叉树的第i层上至多有2^(i-1)个结点        
- 在深度为k的二叉树中，结点数最多为2^k - 1   
- 对于任何一个二叉树，终端结点数N0=N2+1(N2:度为2的结点)       
- 对于一颗**完全二叉树**，树的深度值为[log2N]+1 (N为结点数，[]为不大于该值的最大整数)         
- 对于一颗有n个结点的**完全二叉树**（深度为[log2N]+1）的结点按层序编号，对于任一结点i:   
1.若i=1,则结点i为根结点，无双亲；若i>1，双亲结点的编号为[i/2]     
2.若2i>n，则结点无左孩子，否则左孩子结点为2i        
3.若2i+1>n，则结点无右孩子，否则右孩子结点为2i+1           

**二叉树的存储结构**        
二叉树的特殊性可以使用顺序结构存储按层序编号的结点，不存在的结点需要在数组对应编号处空缺。极端情况下，一颗深度为k的右斜树结点数为k，但需要 2^k - 1个存储单元。一般顺序存储结构适用于完全二叉树。        
使用链式存储结构，设计一个数据域和两个指针域的结点链表来存储二叉树，这样的链表叫做二叉链表，如有必要可以再添加指向双亲的指针域，为三叉链表。     

**二叉树的遍历**        
二叉树的遍历是指从根结点出发，按照某种次序依次访问二叉树中的所有结点，使得每个结点被访问一次且仅被访问一次。        
如果限制遍历方向从左向右，主要分为四种遍历方法：          
![](https://i.imgur.com/BGX5deh.png)          
**1.前根序遍历**                    
 规则：二叉树为空，则空操作返回，否则先遍历根结点，然后遍历左子树，最后遍历右子树。   
ABDHECFG           
**2.中根序遍历**       
规则：二叉树为空，则空操作返回，否则先遍历左子树，然后遍历根结点，最后遍历右子树。         
HDBEAFCG            
**3.后根序遍历**     
规则：二叉树为空，则空操作返回，否则先遍历左子树，然后遍历右子树，最后遍历根结点。       
HDEBFCGA       
**4.层序遍历**       
规则：二叉树为空，则空操作返回，否则按照层序从左至右依次访问结点。         

二叉树的定义和遍历采用递归的方式 
二叉树链表结构及遍历实现：        

	#define OK 1
	#define ERROR 0
	#define TRUE 1
	#define FALSE 0

	#define MAXSIZE 100

	typedef int Status;

	//用于构造二叉树的全局变量
	int index = 1;
	typedef char String[24];
	String str; //字符数组的别名    

	Status StrAssign(String T, char *chars) {
	int i;
	if (strlen(chars) > MAXSIZE)
		return ERROR;
	else {
		T[0] = strlen(chars);
		for (i = 1; i <= T[0]; i++)
			T[i] = *(chars + i - 1);
		return OK;
	}
	}

	typedef char TElemType;
	TElemType Nil = ' ';

	Status visit(TElemType e) {
	printf("%c", e);
	return OK;
	}

	typedef struct BiTNode {
	TElemType data;
	struct BiTNode *lchild, *rchild;
	}BiTNode, *BiTree;

	Status InitBiTree(BiTree *T) {
	*T = NULL;
	return OK;
	}

	//销毁二叉树
	void DestroyBiTree(BiTree *T) {
	if (*T) {
		if ((*T)->lchild)
			DestroyBiTree(&(*T)->lchild);
		if ((*T)->rchild)
			DestroyBiTree(&(*T)->rchild);
		free(*T);//释放结点空间
		*T = NULL;
	}
	}

	void CreatBiTree(BiTree *T) {
	TElemType ch;
	ch = str[index++];
	if (ch == '#')
		*T = NULL;
	else {
		*T = (BiTree)malloc(sizeof(BiTNode));
		if (!T)
			exit(OVERFLOW);
		//使用前根序的次序创建二叉树
		(*T)->data = ch;
		CreatBiTree(&(*T)->lchild);   //构造左子树
		CreatBiTree(&(*T)->rchild);   //构造右子树
	}
	}

	Status BiTreeEmpty(BiTree T) {
	if (T)
		return FALSE;
	else
		return TRUE;
	}

	int BiTreeDepth(BiTree T) {
	int i, j;
	if (!T)
		return 0;
	if (T->lchild)
		i = BiTreeDepth(T->lchild);
	else
		i = 0;
	if (T->rchild)
		j = BiTreeDepth(T->rchild);
	else
		j = 0;
	return i > j ? i + 1 : j + 1;
	}

	TElemType Root(BiTree T) {
	if (BiTreeEmpty(T))
		return Nil;
	else
		return T->data;
	}

	TElemType Value(BiTree p) {
	return p->data;
	}

	void Assign(BiTree p, TElemType e) {
	p->data = e;
	}


	//二叉树前根序遍历
	void PreOrderTraverse(BiTree T) {
	if (T == NULL)
		return;
	printf("%c", T->data);                    //先显示结点数据
	PreOrderTraverse(T->lchild);       //再遍历左子树
	PreOrderTraverse(T->rchild);      //再遍历右子树
	}

	//二叉树中根序遍历
	void InOrderTraverse(BiTree T) {
	if (T == NULL)
		return;
	InOrderTraverse(T->lchild);       //先遍历左子树
	printf("%c", T->data);                //再显示结点数据
	InOrderTraverse(T->rchild);      //再遍历右子树
	}

	//二叉树后根序遍历
	void PostOrderTraverse(BiTree T) {
	if (T == NULL)
		return;
	PostOrderTraverse(T->lchild);  //先遍历左子树
	PostOrderTraverse(T->rchild);  //再遍历右子树
	printf("%c", T->data);                //显示结点数据
	}


	int main()
	{
	int i;
	BiTree T;
	TElemType e1;
	InitBiTree(&T);

	StrAssign(str, "ABDH#K###E##CFI###G#J##");

	CreatBiTree(&T);

	printf("构造空二叉树后，树空否？%d(0:否,1:是) 树的深度=%d\n", BiTreeEmpty(T), BiTreeDepth(T));
	e1 = Root(T);
	printf("二叉树的根结点:%c\n",e1);

	printf("二叉树前根序排列:\n");
	PreOrderTraverse(T);
	printf("\n二叉树中根序排列:\n");
	InOrderTraverse(T);
	printf("\n二叉树前跟序排列:\n");
	PostOrderTraverse(T);
	
	getchar();
	return 0;
	}
      
**二叉树遍历性质**        

- 已知前根序和中根序，可以唯一确定一颗二叉树
- 已知后根序和中根序，可以唯一确定一颗二叉树            

**线索二叉树**         
利用二叉链表中的空指针域存放指向结点在某种次序下的前驱和后继结点的地址，将指向前驱和后继的指针称为线索，加上线索的二叉链表称为线索链表，相应的二叉树称为线索二叉树。      
线索二叉树中需要解决的一个问题是，如何知道一个结点的左指针域是指向其左孩子还是指向该结点的前驱，或者右指针域指向其右孩子还是该结点的后继，因此线索二叉树需要对原结点添加两个标志位，ltag和rtag，标志位只存储0和1,0表示指向其对应的孩子，1表示为对应前驱或后继。    

线索二叉树实现：   

	typedef int Status;	/* Status是函数的类型,其值是函数结果状态代码,如OK等 */
	typedef char TElemType;
	typedef enum {Link,Thread} PointerTag;	/* Link==0表示指向左右孩子指针, */
										/* Thread==1表示指向前驱或后继的线索 */
	typedef  struct BiThrNode	/* 二叉线索存储结点结构 */
	{
	TElemType data;	/* 结点数据 */
	struct BiThrNode *lchild, *rchild;	/* 左右孩子指针 */
	PointerTag LTag;
	PointerTag RTag;		/* 左右标志 */
	} BiThrNode, *BiThrTree;

	TElemType Nil='#'; /* 字符型以空格符为空 */

	Status visit(TElemType e)
	{
	printf("%c ",e);
	return OK;
	}

	/* 按前序输入二叉线索树中结点的值,构造二叉线索树T */
	/* 0(整型)/空格(字符型)表示空结点 */
	Status CreateBiThrTree(BiThrTree *T)
	{ 
	TElemType h;
	scanf("%c",&h);

	if(h==Nil)
		*T=NULL;
	else
	{
		*T=(BiThrTree)malloc(sizeof(BiThrNode));
		if(!*T)
			exit(OVERFLOW);
		(*T)->data=h; /* 生成根结点(前序) */
		CreateBiThrTree(&(*T)->lchild); /* 递归构造左子树 */
		if((*T)->lchild) /* 有左孩子 */
			(*T)->LTag=Link;
		CreateBiThrTree(&(*T)->rchild); /* 递归构造右子树 */
		if((*T)->rchild) /* 有右孩子 */
			(*T)->RTag=Link;
	}
	return OK;
	}

	BiThrTree pre; /* 全局变量,始终指向刚刚访问过的结点 */
	/* 中序遍历进行中序线索化 */
	void InThreading(BiThrTree p)
	{ 
	if(p)
	{
		InThreading(p->lchild); /* 递归左子树线索化 */
		if(!p->lchild) /* 没有左孩子 */
		{
			p->LTag=Thread; /* 前驱线索 */
			p->lchild=pre; /* 左孩子指针指向前驱 */
		}
		if(!pre->rchild) /* 前驱没有右孩子 */
		{
			pre->RTag=Thread; /* 后继线索 */
			pre->rchild=p; /* 前驱右孩子指针指向后继(当前结点p) */
		}
		pre=p; /* 保持pre指向p的前驱 */
		InThreading(p->rchild); /* 递归右子树线索化 */
	}
	}

	/* 中序遍历二叉树T,并将其中序线索化,Thrt指向头结点 */
	Status InOrderThreading(BiThrTree *Thrt,BiThrTree T)
	{ 
	*Thrt=(BiThrTree)malloc(sizeof(BiThrNode));
	if(!*Thrt)
		exit(OVERFLOW);
	(*Thrt)->LTag=Link; /* 建头结点 */
	(*Thrt)->RTag=Thread;
	(*Thrt)->rchild=(*Thrt); /* 右指针回指 */
	if(!T) /* 若二叉树空,则左指针回指 */
		(*Thrt)->lchild=*Thrt;
	else
	{
		(*Thrt)->lchild=T;
		pre=(*Thrt);
		InThreading(T); /* 中序遍历进行中序线索化 */
		pre->rchild=*Thrt;
		pre->RTag=Thread; /* 最后一个结点线索化 */
		(*Thrt)->rchild=pre;
	}
	return OK;
	}

	/* 中序遍历二叉线索树T(头结点)的非递归算法 */
	Status InOrderTraverse_Thr(BiThrTree T)
	{ 
	BiThrTree p;
	p=T->lchild; /* p指向根结点 */
	while(p!=T)
	{ /* 空树或遍历结束时,p==T */
		while(p->LTag==Link)
			p=p->lchild;
		if(!visit(p->data)) /* 访问其左子树为空的结点 */
			return ERROR;
		while(p->RTag==Thread&&p->rchild!=T)
		{
			p=p->rchild;
			visit(p->data); /* 访问后继结点 */
		}
		p=p->rchild;
	}
	return OK;
	 }	

	 int main()
	{
	BiThrTree H,T;
	printf("请按前序输入二叉树(如:'ABDH##I##EJ###CF##G##')\n");
 	CreateBiThrTree(&T); /* 按前序产生二叉树 */
	InOrderThreading(&H,T); /* 中序遍历,并中序线索化二叉树 */
	printf("中序遍历(输出)二叉线索树:\n");
	InOrderTraverse_Thr(H); /* 中序遍历(输出)二叉线索树 */
	printf("\n");
	
	return 0;
	}		

如果所用二叉树需经常遍历或查找结点时需要某种遍历序列中的前驱和后继，可以采用线索二叉链表的存储结构




 
