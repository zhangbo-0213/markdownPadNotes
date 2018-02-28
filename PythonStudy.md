### Python基本语法 
1. 字符串可以同时使用单引号''或同时使用双引号" "，不能一边单引号一边双引号               
2. 行尾没有分号        
3. 变量名命名即使用，没有类型声明         
4. 条件判断时，判断语句可以不使用括号，**条件判断句尾需要加冒号：** 语句之块之间的从属关系通过缩进进行区别，elif表示 else if ，有一个规律是：在C语言中需要{}的地方，在Python中句尾换成：，同时下一行自动缩进           
5. print输出默认转行，如果不想转行，句尾加 ,      
6. import关键字引入所需模块，就和C中包含头文件一样，例如引入画图模块，import turtle      
7. Python中不支持自增自减操作符，类似操作通过赋值完成 i+=1         
8. 通过 import module as othername给模块别名，调用模块函数时可以使用别名        
9. 常见基本数据类型：   
    字符串 ‘str’   
    整数 123    
    浮点数 3.14      
    bool     true/false       
10. 单行注释：  #单行注释内容        
      多行注释   ''' 多行注释内容 '''  或  """  多行注释内容"""         
11. 关于字符串     
	\\'表示单引号，\\"表示双引号       
	'I\'m a \"good\" teacher'       
	\被称作转译字符，除了用来表示引号，还有比如用              
	\\\表示字符串中的\               
	\n表示字符串中的换行
	\还有个用处，就是用来在代码中换行，而不影响输出的结果：
	"this is the\
	same line"
	这个字符串仍然只有一行，和
	"this is thesame line"一样          
	在三个引号中，你可以方便地使用单引号和双引号，并且可以直接换行        
	'''      
	"What's your name?" I asked.     
	"I'm Han Meimei."       
	'''            
	使用  r"字符串",则字符串内的所有符号保持原有样式输出，不存在转义       

12. 格式化输出        
	print 'My age is %d' % num      
	print ‘Price is %.2f’ % 4.99    (.2f要保留的小数位数，四舍五入保留)     
	多个值的格式化输出：    
	print "%s's age is %d "  %('Mike',24)        

13. 关于bool的类型转换          
	在python中，其他类型转成 bool 类型时，以下数值会被认为是False：      
   	为0的数字，包括0，0.0       
       空字符串，包括''，""       
   	表示空值的None      
    	空集合，包括()，[]，{}    
	其他的值都认为是True。         

14. list：列表，用来处理一组有序项目的数据结构，列表定义 listName=[item1,item2,item3,...item n];   列表中的数据元素可以是不同的数据类型。列表中的元素访问通过下标形式listName[index]，listName.appand[index]成员方法可以为列表添加成员。通过 del listName[index]删除列表中的成员。       
list常用操作：       
**索引**     
Python中list可以使用负数索引：    
list[-1]表示最后一个元素，list[-3]表示倒数第三个元素       
**切片**      
list[num1:num2]  该操作即为对list进行切片，冒号前的数字表示切片开始位置，冒号后的数字表示切片终止位置 ，切片后返回一个列表，包含开始位置下标的元素，不包含原切片的终止位置下标元素。         
如果不指定第一个数，切片就从列表第一个元素开始。    
如果不指定第二个数，就一直到最后一个元素结束。    
都不指定，则返回整个列表的一个拷贝。      
l[:3]     
l[1:]     
l[:]      
切片中的索引也可以是负数，规则与索引一样     

15. 字符串分割split()        
str.split()默认按照空格将字符串分割，返回分割后的list，可以传入要分割的**字符**     

16. 字符串连接join()         
字符串的连接使用join(),将list内的成员连接成一个完整字符串,join()是字符串的方法，list(字符串也可以)是其参数，一般是用于连接的字符s调用该方法连接list，即  s.join(list)     

17. 字符串首尾删除    
s.strip(rm)        删除s字符串中开头、结尾处，位于 rm删除序列的字符  
s.lstrip(rm)       删除s字符串中开头处，位于 rm删除序列的字符       
s.rstrip(rm)      删除s字符串中结尾处，位于 rm删除序列的字符    
注意：    
 当rm为空时，默认删除空白符（包括'\n', '\r',  '\t',  ' ')        

17. 文件读取操作:        

	f = open('data.txt')       //参数可以是绝对路径，也可以是相对路径
	data = f.read()         
	//readline() #读取一行内容
	//readlines() #把内容按行读取至一个list中
	print data     
	f.close()        
18. 文件写入操作

	f=open('data.txt','w')  //第二个参数默认是读取模式，w即为写模式，这种模式会覆盖原有文件内容，没有改文件时，会创建文件，a为添加模式，会在已有内容基础上添加，而不是覆盖
19. 字典。字典是Python的基本数据类型，字典一对对的键值对保存数据。键值对之间用逗号分割，键与值之间使用冒号分割，所有的键值对包括在花括号内，字典是这些键值对的集合。      

	d={key1:value,key2:value}        
键唯一，且只能是简单对象，如字符串，整数，浮点数，bool值      
值可以有相同的，list列表可以作为值       
Python中的键值对是没有顺序的，不能通过下标索引访问，通过d['键']进行访问    
可以通过for in 遍历字典，遍历的参数是键,即：   
	
	for name in d:
		print d[name]      
添加一个新的键值对，直接赋值     

	d[newname]=newvalue    
删除一个键值对      

	del d[name]  #前提是该键值对在字典中     
建立一个空的字典    
	d={}     
20. 函数的默认参数，在定义函数时可以给函数形参一个默认值，当函数调用时给出了具体参数，就使用具体参数，否则使用默认参数：     
	
		def hello(name='world')      
			print 'hello'+name
如果函数定义时有多个形参，那么设置默认参数应放在参数列表的末尾      
		
		def func(a,b=5):         
21. 关于面向对象      
		
		class className:
			pass      
定义一个类，使用object=className()创建类的实例，即对象。       
类方法和之前定义的函数区别在于，第一个参数必须为self。而在调用类方法的时候，通过“对象.方法名()”格式进行调用，而不需要额外提供self这个参数的值。self在类方法中的值，就是调用的这个对象本身。      

22. and or使用技巧         
and or 在连着用时，不加括号时，and 优先级高于 or     
使用规则：    
a and b    如果a为True 返回结果为b 否则为a    
a or b       如果a为True 返回结果为a 否则为b      

23. 元组  
元组是一种序列，和list类似，只是元组成员中的元素在创建之后不能被修改
元组： (item1,item2,item3...)
都是元组的实例，和list一样具有索引，切片和遍历操作    

24. 正则表达式     
正则表达式：记录文本规则的代码        
默认情况下正则表达式严格区分大小写          
最简单的正则表达式，没有特殊符号，只有基本数字或字母，表示完全匹配。    

	\b占位符号（表示空格、标点、换行的占用位置）   
	如：  r'\bhi'    表示匹配所有满足 hi 前有空格，标点或换行的结果，即以hi开头的单词   
		  r'\bhi\b'    表示匹配以 hi 为单词的结果，因为只有单词前后才会是空格或标点或换行        
	  
	[]符号，表示只要满足[]内的任意一个字符即可   
	[hi]i  表示只要匹配到 hi或ii 的结果均可       
	“.” 在正则表达式中表示除了换行符以外的任一字符       
	 "\S" 表示除了空白符以外的任一字符           
	
	
		 '*'表示任意数量连续字符，被称为通配符        
		在很多搜索中，会用'.'表示任意一个字符，'*' 表示任意数量连续字符，这种被称为通配符。但在                
		正则表达式中，任意字符是用“.”表示，而“*”则不是表示字符，而是表示数量：
		它表示前面的字符可以重复任意多次（包括0次），只要满足这样的条件，都会被表达式匹配上。

		结合前面的“.*”，用“I.*e”去匹配，会得到结果:
		['I am Shirley Hilton. I am his wife']

		也许你会以为是
		['I am Shirle', 'I am his wife']

		这是因为“*”在匹配时，会匹配尽可能长的结果。如果你想让他匹配到最短的就停止，需要
		用“.*?”。如“I.*?e”，就会得到第二种结果。这种匹配方式被称为懒惰匹配，
		而原本尽可能长的方式被称为贪婪匹配。         

		text2='site sea sue sweet see case sse ssee loses'
		print text2+' 中所有以s开头 和 e结尾的单词'
		m=re.findall(r'\bs\S*?e\b',text2)
数字匹配的几种方法：    

		1.[0123456789] （[]表示匹配其中的任意一个字符）
		简写成	[0-9]
		类似匹配字母 [a-zA-Z]
		2.\d     
		表示任意长度的数字：
		[0-9]*
		\d*         
		这里值得注意的是：*表示任意长度，也包括0 
		而 + 同样表示任意长度，但不包括 0 ，从1开始计数
		因此，要表示任意长度的数字，从1开始：    
		[0-9]+
		\d+   
		如果要表示手机号的匹配规则：     
		\d{11}   后面的+换成{} ，并在里面填充数字位数      
		以1开头：    
		1\d{10}        
正则表达式中其他的符号：
		
		\w 匹配字母或数字或下划线或汉字  （3.x版本可以匹配汉字，2.x版本不匹配）    
		\s 匹配任意的空白符 （与\S相反）
		^ 匹配字符串的开始（某一行）       
		$ 匹配字符串的结束（某一行）           
关于正则表达式中的反义：       

		\S是\s的反义，任意非空白符的字符      
		\W是\w的反义，任意非字母、数字、汉字、下划线的字符          
		\D是\d的反义，任意非数字字符    
		\B是\b的反义，任意非字母开头或结束的位置       

		[abcd]的反义：  [^abcd]表示除abcd以外的任意字符    
		^[abcd] 则是以abcd任意一个开头的字符串       
		这里^的位置不同，代表意义不同      
关于表示重复数量的字符：     

		* 任意次数的重复 包括 0
		+ 任意次数的重复 不包括 0    
		{} 指定数量的重复    
 		？ 重复0次或1次      
		{n,} 重复n次或更多次      
		{n,m} 重复n到m次         
正则表达式不只是用来从大段文字中抓取信息，很多时候被用来判断输入的文本是否符合要求        

		^\w{4,12}$     :表示4-12位字符，以数字，字母，下划线组成，可以用来检测用户作为用户名是否符合要求
		
		\d{15,18}	：表示一段15-18位的数字，用来检测一串号码       

		^1\d*x?         :以1开头的数字，结尾可以是x或没有x   
关于转义字符 \
		
		如果要匹配的内容中包含.或者*这样的元字符，就需要用到转义字符\，即\.或者\*，    
		如果也包含 \ 需要 \\    
		\d+\.\d+    可以匹配出： 1234.456        
关于匹配合并符 |    
		
		如果需要将两种格式进行匹配的话，可以使用|符号，如，匹配出多种格式的电话号码：   
		(021)88776543
		010-55667890
		02584453362
		0571 66345673     

		通用的匹配规则：   
		\(?0\d{2,3}[( -]?\d{7,8}      
		这种可能会匹配到（）未成对出现的情况，如：   
		(02188776543       

		因此针对有()的情况单独写规则，    
		\(0\d{2,3}\)\d{7,8}
		其他情况：
		0\d{2,3}[ -]?\d{7,8}
		合并：   
		\(0\d{2,3}\)\d{7,8}|0\d{2,3}[ -]?\d{7,8}

25. 列表解析       
		
		list1=[1,3,4,6,9,12,23,34,38]
		#获得list1中的偶数并构建成新的list
		#使用传统方法：
		list2=[]
		for i in list1:
    		if i%2==0:
        	list2.append(i)

		print list2

		#使用列表解析
		list3=[i for i in list1 if i%2==0]
		print list3
		#对元素的操作同样可以放在构建过程中，如对选出的偶数/2操作
		list4=[i/2 for i in list1 if i%2==0]
		print list4

		print ';'.join([str(i) for i in range(1,100) if i%2==0&i%3==0&i%5==0])       

26. 函数参数传递   
	1.默认参数设置     
 	
		func(a=1,b=2,c=3)
		#调用不指定对应位置参数，则使用默认参数  
		func(5)     a=5,b=2,c=3
		#调用可以显示指定具体参数   
		func(a=2,3,4) a=2,b=3,c=4   
		func(a=4,c=5)  a=4,b=2,c=5  
		#调用时没有指定参数名的参数在前，指定参数名的在后，且不可重复     
		func(4,b=5)  允许
		func(a=5,4)   不允许
		func(5,a=5)    不允许 a参数被重复指定     
	2.接受任意数量的参数    
		
		func(*args)      
		#调用时可以传递多个参数(* 调用的参数存储在一个元组中)
		func(1,2,3,4,5)
		func(2,3,5)
	3.将参数以键值对字典形式传入     

		func(**kargs)
		#调用时按照键值进行传参    
		func(a=1,b=3,c=5)    
	4.组合使用   
		
		def func(x,y=5,*args,**krgs):
   		 	print x,y,args,krgs

		func(3)
		func(3,4)
		func(3,4,5)
		func(3,4,5,8)
		func(x=1)
		func(x=1,y=1)
		func(x=1,y=1,a=1)
		func(x=1,y=1,a=1,b=1)
		func(1,y=1)
		func(1,2,3,4,a=1)
		func(1,2,3,4,a=1,b=2,c=3,k=4)
		
		#输出
		3 5 () {}
		3 4 () {}
		3 4 (5,) {}
		3 4 (5, 8) {}
		1 5 () {}
		1 1 () {}
		1 1 () {'a': 1}
		1 1 () {'a': 1, 'b': 1}
		1 1 () {}
		1 2 (3, 4) {'a': 1}
		1 2 (3, 4) {'a': 1, 'c': 3, 'b': 2, 'k': 4}  

		在混合使用时，首先要注意函数的写法，必须遵守：
		带有默认值的形参(arg=)须在无默认值的形参(arg)之后；
		元组参数(*args)须在带有默认值的形参(arg=)之后；
		字典参数(**kargs)须在元组参数(*args)之后。

		可以省略某种类型的参数，但仍需保证此顺序规则。

		调用时也需要遵守：
		指定参数名称的参数要在无指定参数名称的参数之后；
		不可以重复传递，即按顺序提供某参数之后，又指定名称传递。

		而在函数被调用时，参数的传递过程为：
		1.按顺序把无指定参数的实参赋值给形参；
		2.把指定参数名称(arg=v)的实参赋值给对应的形参；
		3.将多余的无指定参数的实参打包成一个 tuple 传递给元组参数(*args)；
		4.将多余的指定参数名的实参打包成一个 dict 传递给字典参数(**kargs)。   

27. lambda表达式        
lambda表达式是定义及调用函数的一种简写方式 （或叫匿名函数）   

	 	lambda 表达式的语法格式：
		#参数列表无()   表达式前无 return
		lambda 参数列表: 表达式    

	 	#这里sum1是一个函数对象
		sum1=lambda a,b,c:a+b+c             

		#它的主体只能是一个表达式，不可以是代码块，甚至不能是命
		令（print 不能用在 lambda 表达式中）。所以 lambda 表达
		式能表达的逻辑很有限。      

28. map函数  

		#使用map函数
		#map为Python自带函数，其作用是将某个函数的功能全部应用到某一个序列中
		def double_func(x):
    			return 2*x
		list2=map(double_func,list1)
		print list2

		#两个列表的元素相加
		list3=[2,3,4,5,6]
		list4=[4,5,6,7,8]
		list5=map(lambda x,y:x+y,list3,list4)
		print list5

		#map 中的函数会从对应的列表中依次取出元素，
		#作为参数使用，同样将结果以列表的形式返回。
		#所以要注意的是，函数的参数个数要与 map 中提供的序列组数相同，
		#即函数有几个参数，就得有几组数据。   

29. reduce函数    
map函数将某个规则映射到列表的每一个元素并生成的新的序列，而ruduce函数则是将某个规则应用到列表中的元素，并将所得结果与下一下元素继续应用规则，直到应用到列表的最后一个元素，最后得到输出。     
		
		#求一个数列的和
		list1=range(1,101)
		def add(x,y):
    		return x+y

		sum1=reduce(add,list1)
		print sum1

		#使用lambda表达式
		sum2=reduce(lambda x,y:x+y,range(1,101))
		print sum2      

30. Python中的多线程     
Python中提供多线程模块,thread模块 ,其中提供函数  :      

		start_new_thread(function,args[,kwargs])  
		#function为开发者定义的线程函数
		#args是传递给线程函数的参数，必须是元组类型
		#kwargs是可选参数       
32. 生成器     
通过列表解析可以直接生成一个列表，由于受到内存限制，列表容量有限。如果列表元素可以通过某种算法推算出来，在循环过程中不断推出后续元素，就不必创建完整的列表，从而节省空间。Python中边循环边计算的机制，称为生成器。   

	
		#创建生成器的方法一，将列表解析中的[]改为()
		L=(x*x for x in range(6))
		#生成器的元素通过next（）函数得到
		print next(L)
		print next(L)
		print next(L)
		print next(L)
		print next(L)
		print next(L)
		#生成器保存算法，每次调用next（）函数得到下一个元素的值，直到最后一个元素，没有更多元素时
		#抛出StopIteration错误
		#生成器也属于可迭代对象，可以通过for进行遍历
		g=(2*x for x in range(10))
		for n in g:
    			print n

		#创建生成器的方法二，使用yield关键字

		#打印斐波那契数列的函数
		def fbb(max):
    			n,a,b=0,0,1
   			while n<max:
        			print b
        			a,b=b,a+b
        			n=n+1
    			return 'done'
		print fbb(6)

		#如果想要保存斐波那契数列结果，一般做法需要通过列表保存下来
		def fbb2(max):
    			L=[]
    			n,a,b=0,0,1
    			while n<max:
        			L.append(b)
        			a,b=b,a+b
        			n=n+1
    			return L

		print [x for x in fbb2(6)]

		#而当这个数列需要返回的元素个数很多时，使用列表就会占用大量的空间，因此这里使用yield
		#yield使普通函数作为一个生成器，每次只返回当前迭代的元素结果
		def fbb3(max):
    		n,a,b=0,0,1
   		 		while n<max:
        			yield b
        			a,b=b,a+b
        			n=n+1
		#普通函数是顺序执行，而generator生成器是在调用next（）执行，遇到yield返回，yield后的结果，
		#再次遇到next（）继续上一次的进行，直到没有更多元素，抛StopIteration异常结束
		#生成器可迭代，因此通过for循环自动执行next()直到结束，不会抛出StopIteration异常
		for x in fbb3(10):
    		print x
         
		#计算杨辉三角
		def YangAngles():
    			L=[1]
    			while True:
        			yield L
        			L=[1]+[L[n]+L[n+1] for n in range(len(L)-1)]+[1]
	
33. with ...as语法  
【转载记录】           	  	
先说明一个常见问题，文件打开：
			
			try:
    				f = open('xxx')
    				do something
			except:
    				do something
			finally:
    				f.close()
其实不止一次在网上看到有这么写的了，这个是错的。 
首先正确的如下：

			try:
   				 f = open('xxx')
			except:
    				print 'fail to open'
    				exit(-1)
			try:
    				do something
			except:
    				do something
			finally:
   			f.close()
很麻烦不是么，但正确的方法就是这么写。     
我们为什么要写finally，是因为防止程序抛出异常最后不能关闭文件，但是需要关闭文件有一个前提就是文件已经打开了。      
在第一段错误代码中，如果异常发生在f=open(‘xxx’)的时候，比如文件不存在，立马就可以知道执行f.close()是没有意义的。改正后的解决方案就是第二段代码。      
开始讨论with语法。       
首先从try-finally的语法结构说起：

		set things up
		try:
    			do something
		finally:
    			tear things down
这东西是个常见结构，比如文件打开，set things up就表示f=open(‘xxx’)，tear things down就表示f.close()。在比如像多线程锁，资源请求，最终都有一个释放的需求。Try…finally结构保证了tear things down这一段永远都会执行，即使上面do something得工作没有完全执行。                                 
如果经常用这种结构，我们首先可以采取一个较为优雅的办法，封装！       
	
		def controlled_execution(callback):
    		set things up
    		try:
        		callback(thing)
   	 	finally:
        		tear things down

		def my_function(thing):
    		do something

		controlled_execution(my_function)
封装是一个支持代码重用的好办法，但是这个办法很dirty，特别是当do something中有修改一些local variables的时候（变成函数调用，少不了带来变量作用域上的麻烦）。                       
另一个办法是使用生成器，但是只需要生成一次数据，我们用for-in结构去调用他：   
	
		def controlled_execution():
    			set things up
   			try:
        			yield thing
    			finally:
        			tear things down
		for thing in controlled_execution():
   	 		do something with thing
因为thing只有一个，所以yield语句只需要执行一次。当然，从代码可读性也就是优雅的角度来说这简直是糟糕透了。我们在确定for循环只执行一次的情况下依然使用了for循环，这代码给不知道的人看一定很难理解这里的循环是什么个道理。
最终的python-dev团队的解决方案。（python 2.5以后增加了with表达式的语法）

		class controlled_execution:
    			def __enter__(self):
        			set things up
        			return thing
    			def __exit__(self, type, value, traceback):
        			tear things down

		with controlled_execution() as thing:
        		do something
在这里，python使用了with-as的语法。当python执行这一句时，会调用__enter__函数，然后把该函数return的值传给as后指定的变量。之后，python会执行下面do something的语句块。最后不论在该语句块出现了什么异常，都会在离开时执行__exit__。                       
另外，__exit__除了用于tear things down，还可以进行异常的监控和处理，注意后几个参数。要跳过一个异常，只需要返回该函数True即可。下面的样例代码跳过了所有的TypeError，而让其他异常正常抛出。          

		def __exit__(self, type, value, traceback):
    			return isinstance(value, TypeError)
在python2.5及以后，file对象已经写好了enter和exit函数，我们可以这样测试：

		>>> f = open("x.txt")
		>>> f
		<open file 'x.txt', mode 'r' at 0x00AE82F0>
		>>> f.__enter__()
		<open file 'x.txt', mode 'r' at 0x00AE82F0>
		>>> f.read(1)
		'X'
		>>> f.__exit__(None, None, None)
		>>> f.read(1)
		Traceback (most recent call last):
    		File "<stdin>", line 1, in <module>
		ValueError: I/O operation on closed file
之后，我们如果要打开文件并保证最后关闭他，只需要这么做：

		with open("x.txt") as f:
    			data = f.read()
    			do something with data
如果有多个项，我们可以这么写：

		with open("x.txt") as f1, open('xxx.txt') as f2:
    			do something with f1,f2
上文说了__exit__函数可以进行部分异常的处理，如果我们不在这个函数中处理异常，他会正常抛出，这时候我们可以这样写（python 2.7及以上版本，之前的版本参考使用contextlib.nested这个库函数）：

		try:
    			with open( "a.txt" ) as f :
        		do something
		except xxxError:
    		do something about exception
总之，with-as表达式极大的简化了每次写finally的工作，这对保持代码的优雅性是有极大帮助的。

		with-block等价于

		try:  
      			执行 __enter__的内容  
      			执行 with_block.  
		finally:  
      			执行 __exit__内容 

### 关于函数 ###
**1.input()&raw_input()**         
在Python2中，input()和raw-input()均可以接受用户输入，区别在于:        
input() 获取到的内容会转化为相应的类型并返回，例如输入 123 会以整数的形式返回，因此如果要想接收输入的字符串  必须输入时 带引号，不带引号建会返回一个未声明的变量名         
而raw-input()则是将所有的输入均转化为字符串，因此不需要加引号      

**2.range()**       
接收两个参数整形参数a,b,得到一个整数的集合，其中n,a<=n<b,如果只传递一个整形参数，a默认为0，如果a>b，得到空集      

**3.str()等类型转换**     
将参数转化为字符串并返回,类似的类型转换还有：

	int('123')=123
	float('3.3')=3.3
	bool(0)=False

### 关于模块  ###
**turtle**           

	turtle.forward()   
	turtle.right()
	turtle.penup()/pendown()/goto()   
 	turtle.fillcolor('color')/begin_fill()/end_fill()      
	// 起始点到结束点连成线构成的封闭区域上色       
	turtle.color('color') //画笔变为指定颜色，不会填充区域
	turtle.done()命令，结束等待           

**Tkinter**        
  
	Tkinter.Tk()     //创建一个窗口对象
	Tkinter.Entry()   //参数为Tkinter窗口对象，函数返回一个输入对象entry，使用entry.pack()，使输入框对象悬挂在窗口      
	Tkinter.button() //创建一个button对象，参数为Tkinter窗口对象,需要.pack()悬挂      

**random**          

	print 'random.uniform'              #输出范围内的随机小数
	print random.uniform(10,20)      #17.749276658
	print random.uniform(20,10)      #11.9747960764


	print 'random.random'              #输出0-1之间的随机小数
	print random.random()             #0.849376425391

	print 'random.range'                      #输出指定递增的范围内(不包括上限)的随机整数
	print random.randrange(0,10,2)    #(0,2,4,6,8) 6

	print 'random.randint'                  #输出范围内的整数，包括上下限
	print random.randint(0,5)            #3    

	random.choice(list)     #随机挑选列表中的元素并返回       
	
	random.sample(population,k)   #对population进行随机取样k个元素生成新的序列，sample不改变原来的序列    

	random.shuffle(x)  #将列表x中的顺序打乱，直接改变原来的序列。        

	随机数中有随机种子概念，通过一个真实的随机数，类似此刻时间
	或鼠标位置等，以此为基础产生伪随机数。Python中，默认使用系
	统时间作为随机种子，random.seed(x)可以手动指定随机种子。一
	般在调用其他随机函数前调用此方法，同一个种子下产生的随机数相同     
	random.seed(10)
	print 'Random number with seed 10:',random.random()
	#输出：Random number with seed 10: 0.57140259469

	random.seed(10)
	print 'Random number with seed 10:',random.random()
	#输出：Random number with seed 10: 0.57140259469

	random.seed(10)
	print 'Random number with seed 10:',random.random()     
	#输出：Random number with seed 10: 0.57140259469
**urllib**    

	#获取网络资源模块
	import urllib2
	web=urllib2.urlopen('http://www.baidu.com')
	content=web.read()
	print content

**time**     
	#提供与时间相关的方法      
	#epoch时间：表示的时间 1970-01-01 00:00:00 UTC       
	time.time()  #表示从epoch到当前的秒数，该值被称为unix时间戳     
	通过该方法记录程序运行的开始和结束时间，计算出运行时间       

	time.sleep(secs) #使程序暂停secs秒  
	

### python爬虫小实例 ###
#### Requests+ 正则表达式爬取 猫眼电影####
**step1-抓取单页内容**    
利用requests请求目标站点，得到单个网页的HTML代码，返回结果       
**step2-正则表达式分析**    
根据HTML代码分析得到电影的名称，主演，上映时间，评分，图片链接等信息         
**step3-保存至文件**    
通过文件的形式将结果保存，每一部电影一个结果一行JSON字符串         
**step4-开启循环及多线程**     
对多页面内容遍历，开启多线程提高抓取速度         



 

