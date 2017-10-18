## Lua学习记录 ##
### 前言 ###
Lua是一种轻量小巧的脚本语言，几乎可以在所有的操作系统上运行。  
可以很方便的更新代码，代码更新后可以直接在手机上运行，不需要重新安装新的安装包。  
非常适合作为一种热更新方案。  
大部分主流的编程语言编写的程序需要在特定操作系统中进行编译得到dll文件，打包安装到其他平台上运行，  
移动平台上不能更新替换已有的dll文件，除非重新下载更新包。  

常用的IDE编辑器：**SciTE**   

----------
### 语法 ###
- 文件命名保存的后缀最好为.lua
- 每一行代码结尾 ; 可加可不加
- 注释       

		print("Hello World!")
		print("Hello Lua");
		--单行注释 print 向控制台输出字符串
		--[[
		这是多行注释
		这是多行注释
		--]]  
		--多行注释取消技巧 ：在多行注释的开头加 - 则开头和结尾均变为单行注释，中间的内容取消注释
		---[[
		--]]  
- 标示符命名规则   
开头以_和字母，中间可以有数字   
区分大小写   
不推荐下划线+大写字母进行命名  

- 全局变量  
全局变量不需要声明，给一个变量赋值即创建了一个全局变量。访问一个未初始化的变量并不会报错，只是得到的结果是nil  

		print(b)      --输出 nil
		b=10  
		print(b)      --输出10  
		b=nil     --对变量清空，清空变量所占的内存空间，该变量也不存在了  

- 数据类型     
Lua是动态类型语言，变量不要类型定义，只需要为类型赋值。值可以存储在变量中，作为参数进行传递。Lua中有8个基本的数据类型：  

>
- nil  nil值属于该类，表示一个不存在或者无效值   
- boolean 布尔值
- number  表示双精度类型的实浮点数    
- string  字符串，由一对引号或单引号表示   
- function 由C或Lua编写的函数   
- userdata  表示任意存储在变量中的C数据结构   
- thread 表示执行的独立线路，用于执行协同程序   
- table Lua中的表是一个关联数组，数组的索引可以是数字或者是字符串。在Lua中，table的创建是通过“构造表达式”完成，最简单的构造表达式{}，用于表示一个空表。  

通过Type函数测试数据类型    

	print(type("Hello World!"))    --string
	print(type(10.4*3))                  --number
	print(type(print))                    --function
	print(type(type))		  	      --function
	print(type(true))			      --boolean
	print(type(nil))                        --nil
	print(type(type(nil)))		      --string
**nil**类型表示一个无效值，对于全局变量和table，nil还有**清除的作用**，释放变量所占用的内存空间 

	name="bobo"
	print(name)
	name=nil
	print(name)  

	table1={key1="value1",key2="value2"}
	print(table1.key1)
	table1.key1=nil
	print(table1.key1)
	table1=nil
	print(table1)   

**boolean**类型的可选值为true或false，Lua将nil视为false，其他有效值视为true   

	print(type(true))
	print(type(false))
	print(type(nil))

	if true then
	print("判断条件为真")
	end

	if nil then
	print("nil 为真")
	else
	print("nil为假")
	end
输出：   

	boolean
	boolean
	nil
	判断条件为真
	nil为假  
**number**可以看作为double类型的数据，不区分整数和小数 

**string **字符串，可以用单引号或双引号来表示 
可以使用[[ 和]]表示多行的字符串  
	
	html=[[
	<html>
	<head></head>
	<body>
		<a herf="www.baidu.com">百度一下</a>
	</body>
	</html>
	]]

	print(html)
字符串的组拼使用 **..** **+**是用来做加法运算的，如果对 **数字字符串使用+**,Lua中会将字符串转化为数字进行运算，如果非数字字符串使用+，则会报错   

	print("2".."6")
	--输出26
	print("2"+"6")
	--输出8 
	print("2"+6)
	--输出8 
	print("2e2"*"6")
	--输出1200
使用#取得字符串的长度  
	print(#"html")  
	--输出4	

**Table**通过键值对存储数据,两种访问表中数据的值的方式 
	
	 tab1={}  --空表 {}构造表达式
 	 tab2={key1=100,key2="value"}  --初始化一个表
 	 print(tab2.key1)
 	 --输出100
 	 print(tab2["key2"])
 	 --输出value  
表赋值的另一种形式：   
 	
	tab3={"apple","pear","orange","grape"}  
	print(tab3[2])
	--输出pear
当表中只有值，而没有显示指定键的时候，默认每个值的键为number类型，索引值从1开始。 Lua中没有数组，表的这种用法相当于数组  

遍历一个表中的值  

	for key,val in pairs(tab3) do
		print(key..":"..val)
	end  
	--[[输出:
	1:apple
	2:pear
	3:orange
	4:grape
	--]]  
 Lua中的Table不指定固定长度，会根据数据使用的情况扩容或缩小。表中数据的添加，修改和删除：

	tab1.key1="value1"
	tab1["key2"]="value2"
	tab1.10=1000
	print(tab1["key1"])
	--输出 value1
	print(tab1.key2)
	--输出 value2
	print(tab1.10)
	--输出 1000  
	tab1["key2"]=20
	print(tab1.key2)
	--输出20
删除一个数据，将值设为nil，同时对应的键也会被删除  
	
	tab1.key2=nil
	for key,val in pairs(tab1) do 
		print(key..":"..val)
	end  
	--[[输出
	key1:value1
	10:1000
	--]]  

**function**语法：  

	--[[   c#中的函数定义
	int fact(int n){
		if(n==1)
			retrun n;
		else
			return n*fact(n-1)
	}
	--]]

	function fact(n)          --关键字function 无需指明返回值 函数名（形参），没有{}
		if (n==1) then   
			return n
		else                        
			return n*fact(n-1)
		end                      --if   then  end 对
	end                          --函数结尾 end

	print(fact(5))

	fact2=fact                    --函数是一种类型，可以给其他变量赋值，被赋值后的变量也是一个函数
	print(fact2(3))  

函数作为参数传递及匿名函数的使用  
	
	function func(tab,fun)
		for k,v in pairs(tab) do
			fun(k,v)
		end
	end

	tab1={key1="value1",key2="value2"}

	function f1(k,v)
		print(k..":"..v)
	end

	function f2(k,v)
		print(k.."&"..v)
	end

	func(tab1,f1)       --f1,f2作为参数传递，类似C#中的委托
	func(tab1,f2)


	func(tab1, function(k,v)     --匿名函数的使用
				print(k.."--"..v)
				end
	)  

**变量声明与使用**    

- 变量声明无需指定类型
- 变量可更改类型  

		a=5
		print(type(a))     --number
		a="hello"
		print(type(a))     --string  



- 变量声明后，默认为全局变量，当在声明变量之前加关键字local 则该变量为局部变量，在对应的声明范围内使用完后会被销毁    

		function test()
		c=30
		local d=40
		end

		test()
		print(c)            --30
		print(d)		    --nil  
- 语句块中全局变量会覆盖块外的同名变量，局部变量在语句块执行完后被销毁，变量的访问遵循就近原则
- 推荐尽可能使用局部变量，访问更快，更节约资源  
- 多变量赋值与交换赋值     

		a1,a2,a3=13,25,"hello"  
		print(a1,a2,a3)   --输出25,13,"hello" nil

		a1,a2=a2,a1
		print(a1,a2)   --输出25,13
		
		function test()
		return 10,20
		end

		b1,b2=test()
		print(b1,b2)    --输出10,20   

**循环**    

	--[[
	1.while循环
	2.for循环
	3.repeat循环   （do while）


	1.while循环

	while(condition) do
	statements
	end

	2.for循环	
	2.1 数值for循环

	for var=start,end,step do
		statements
	end
	2.2 泛型for循环


	3.repeat循环

	repeat
		statement
	until(condition)
	--]]


	a=1	
	while (a<=20) do
	if (a%2==1) then
		print(a)
	end
	a=a+1
	end

	for var=1,10,2 do
	print(var)
	end

	for var=20,10,-1 do
	print(var)
	var=var-1
	end

	tab1={key1="value1",key2="value2"}
	for k,v in pairs(tab1) do
	print(k.."&"..v)
	end

	tab2={"一","二","三","四"}
	for k,v in pairs(tab2) do
	print(k,v)
	end

	repeat
	print(a)
	a=a-2
	until(a<=0)  

	--循环嵌套
	for var=1,10,1 do
		for var1=1,var,1 do
			print(var)
		end
	end   

**流程控制**    
if语句      

	--[[

	if(condition) then
		statements
	else if
		statements
	else
		statements
	end

	--]]


	if(0) then
		print(0)
	end
		--这里需要注意的是，条件判断时，nil为false，其他确定类型则为true
		--0 类型为number 因此条件判断结果为true  输出0

	a=10
	if a>10 then
		print(a)
	else
		print(a.."<=10")
	end
	--输出 10<=10

	if a>10 then
		print("a>10")
	elseif a<10  then   --elseif中间无空格  后接then
		print("a<10")
	else
		print("a=10")
	end
	--输出a=10   

**函数修饰符**   
函数的修饰符 local 和变量一样,表示局部函数

	[local] functionName(arg1,arg2,arg3...)
	functionBody
	[retrun value1,value2,value3...] 
	
	local function max(num1,num2)
		if num1>num2 then
			return num1
		else
			return num2
		end
	end

函数可以作为数据赋值  作为参数传递  

	temp=max
	print(temp(2,5))

	myprint=function (param)
		print("这是我的打印函数："..param)
	end

	myprint(100)

	function add(num1,num2,printFunc)
		printFunc(num1+num2)
		end

	add(15,20,myprint)  
  
**可变参数**    
	
	--可变参数
	print(20,30,40)
	--可变参数函数定义

	function test(...)      --通过...符号表明函数接受的是可变参数
		local arg={...}     --通过local arg={...}访问可变参数的table，不包含元素数量
		print(arg)          --通过arg内置参数访问得到包含了可变参数的table，除了包含每个元素的值，还包括元素的个数值，
		print(arg[1])       --通过索引来访问table中的数据
	end

	test(1,2,3,4)        --输出table: 003E9768  

	function average(...)
		local arg={...}
		res=0
		for k,v in pairs(arg) do
		res=res+v
		end
		print("average:"..(res/#arg))   --#table/#字符串  取得其长度
	end  
	average(1,2,3,4)
	
 运算符  
	
	--关系运算符 ==  ~=不等于  > < >= <=
	if(a~=b)  then
		print("a~=b")
	else
		print("a==b")
	end

	--逻辑运算  and   or not
	if(1 and 2)  then
		print("条件为真")
	end

	print(10>3 and 10<3)
	print(10>3 or 10<3)
	print(not (10>3))     

字符串    

	--[[
	字符串
	1.定义
	--''  ""  [ [ ] ]多行字符串定义

	2.转义字符
	\n  换行
	\r  回车
	\\  \

	--]]
 	--print("hello\n\\world")

 	--print("my name is \"Micheal\"")


 	--字符串操作
 	str="My Name Is bobo"
 	str2=string.upper(str)   --对原字符串没有影响
 	str3=string.lower(str)

	print(str2)
	print(str3)

	--字符串替换
	str4=string.gsub(str,'b','Z',1)  --后面的数字表示可以替换的最多次数
	print(str4)

	--字符串查找
	index=string.find(str,"bobo",10)  --返回查找字符串的索引值，最后的参数表示开始查找的起始位置
	print(index)

	--字符串翻转
	str5=string.reverse(str)
	print(str5)

	--格式化字符串
	num1=5
	num2=10
	str6=string.format("加法运算： %d+%d=%d",num1,num2,(num1+num2))
	print(str6)

	date=2;month=1;year=2017
	print(string.format("日期格式化: %02d/%02d/%04d",date,month,year))
	
	--字符与整数之间转换
	print(string.char(97,98,99,100))
	--输出abcd
	print(string.byte('ABCD'))
	--输出65
	print(string.byte('ABCD',2))    --默认从第一个字符输出,也可以指定位置
	--输出66

	--取得字符串长度
	length=string.len('abc')
	length2=#'abc'
	print(length,length2)
	--输出  3  3

	--字符串拷贝
	str7=string.rep('abcd',2)
	print(str7)
	--输出abcdabcd 


数组     

	array={"lua","code"}

	for i=1,2 do
		print(array[i])
	end
	--输出lua code
	--Lua中索引值从1开始

	array2={}
	for i=-2,2 do
		array[i]=i*3
	end
	for i=-2,2 do
		print(array[i])
	end
	--输出 -6 -3 0 3 6
	--数组通过表实现，索引值可以为负数

	--多维数组
	array3={{1,2},{3,4},{5,6},{7,8}}
	print(#array3[1])
	--输出2

	--遍历多维数组
	for i=1,4 do
		for j=1,2 do
			print(array3[i][j])
		end
	end  

迭代函数   
	
	array={"Lua","C#","JAVA"}
	for k,v in pairs(array) do
		print(k,v)
	end

	array[2]=nil
	for k,v in ipairs(array) do
		print(k,v)
	end

	--pairs迭代table,遍历表中所有的元素
	--ipairs按照索引从1开始，连续遍历，遇到nil会停止遍历


	--[[
	for in 接收返回值的变量列表 迭代函数,状态变量,控制变量 do
	--执行体
	end
	--]]

	--1.调用迭代函数,(将控制变量和状态变量作为参数传入迭代函数)
	--2.如果迭代函数的返回值为 nil 退出循环
	--  如果返回值不为nil，将返回值赋值给变量列表，执行循环体
	--  状态变量在第一次调用时赋值，控制变量则在函数内控制赋值，并有结束条件

	function square(state,control)
		if(control>=state) then
			return nil
		else
			control=control+1
			return control,control*control
		end
	end

	for i,j in square,9,0 do
		print(i,j)
	end

关于表——引用类型

	--[[
	table类似于C#中的对象，是引用类型
	--]]
	
	mytable={}
	print(type(mytable))
	
	mytable[1]="Lua"
	mytable["name"]="bobo"
	
	newtable=mytable
	print(newtable[1])
	--输出 Lua

	mytable[1]="LuaCode"
	print(newtable[1])
	--输出 LuaCode 

	newtable[2]="JAVA"
	print(mytable[2])
	--输出 JAVA	

操作表的方法 

	-table.xxxmethod
	mytable={"Lua","C#","JAVA","C++","C"}

	--拼接（可以指定拼接连接符，开始索引和结束索引）
	print(table.concat(mytable))
	print(table.concat(mytable,','))
	print(table.concat(mytable,',',2,4))

	--插入移除数据
	mytable[6]="PHP"
	mytable[#mytable+1]="Python"
	--数据插入,默认插到末尾
	table.insert(mytable,"JavaScript")
	print(mytable[#mytable])
	--指定位置插入
	table.insert(mytable,2,"Go")
	for k,v in pairs(mytable) do
		print(k,v)
	end
	--输出   Lua Go C# JAVA C++ C PHP Python JavaScript

	--数据移除
	table.remove(mytable,2)
	for k,v in pairs(mytable) do
		print(k,v)
	end
	--输出   Lua C# JAVA C++ C PHP Python JavaScript

	--排序（字母按照ASCII码进行排序,大写字母排在前，小写字母排在后)
	print("排序前")
	for k,v in pairs(mytable) do
		print(k,v)
	end
	print("排序后")
	table.sort(mytable)
	for k,v in pairs(mytable) do
		print(k,v)
	end

	mytable2={23,45,678,39,47,482,37,49}
	table.sort(mytable2)
	for k,v in pairs(mytable2)  do
		print(k,v)
	end

	--取得最大值
	function getMax(mytable)
		local temp=0
		for k,v in pairs(mytable) do
			if v>=temp then
				temp=v
			end
		end
		return temp
	end

	print("最大值为 ："..getMax(mytable2))

模块  
18_module1.lua
	
	--[[
	Lua中的模块通过表来实现，将变量和函数作为表的成员
	--]]

	module1={}
	module1.var="bobo"
	module1.func1=function ()
		print("这是module的一个函数，对应键为 func1")
	end
	
	local function func3()
		print("这里是局部函数3")
	end
	--局部函数无法被模块外部进行调用

	function func2()
	print("这里是全局函数2")
	func3()
	end

	return module1

19_usingModule.lua

	--[[
	模块的使用 通过
	require "模块名"
	--]]
	require "18_module1"
	print(module1.var)
	
	module1.func1()
	func2()

元表  

		--[[
		Lua中 table可以通过对应的key访问到value ，但无法对两个table进行	操作
		因此Lua提供元表(Metatable),允许定义对表的自定义操作
		--]]
		
		--为普通表扩展元表
		mytable={"Lua","C#","C","C++"}    --普通表
		mymetatable={}                    --元表（空表）
		mytable=setmetatable(mytable,mymetatable)   --将mymetatable	设置成mytable的元表,返回普通表
		print(mytable[1])                 --输出Lua
		--另一种设置方式
		--mytable=setmetatable({},{})     普通表和元表均为空表

		--获得空表的元表

		print(getmetatable(mytable))   --输出table: 008E9918
		print(mymetatable)             --输出table: 008E9918

		--当元表中存在 __metatable键值时，使用getmetatable，直接返回__metatable键所对应的值，而不是整个元表
		--这样可以对元表进行保护，防止元表成员被修改

		--元表中的__index作用
		--[[
		元表中的__index作为特殊含义的一种键，有两个作用
		1.__index=function(tab,key) __index后跟一个匿名函数，当访问元表 对应的普通表的元素不存在时，调用该方法
	  	方法的参数为tab和key
		--]]
		mytable={"Lua","C#","Python","C++"}    --普通表
		mymetatable={
		__index=function (tab,key)
			return "C"
		end
		}
		mytable=setmetatable(mytable,mymetatable)
		
		print(mytable[7])     --输出C
		
		--[[
		2.__index={} __index后可以跟一个表，当访问元表对应的普通表不存	在时，就会在__index后的表中寻找（如果__index后跟的是表）
		--]]
		mytable2={"LUA","C#","Python","C++"}
		newtable={}
		newtable[6]="Go"
		newtable[7]="JavaScript"
		metatable2={
			__index=newtable
		}
		mytable2=setmetatable(mytable2,metatable2)
	
		print(mytable2[7])     --输出Javascript

		-元表中的__newindex作用
		--[[
		元表中的__newindex主要针对表的新索引赋值过程，后面可以跟一个匿名函数和一个表
		1.__newindex=function (tab,key,value)   __newindex后跟匿名函数
		--]]
		mytable3={"Lua","C#","Python","Go"}
		metatable3={
		__newindex=function (tab,key,value)   --当对应普通表的新索引赋值时会调用该方法
		print("赋值过程："..key..":"..value)
		rawset(tab,key,value)    --如果要完成对应新增赋值到元表，则需要调用该函数
		end
		}
		mytable3=setmetatable(mytable3,metatable3)

		mytable3[1]="C++"
		mytable3[5]="C"        --输出 赋值过程：5：C  只针对新索引
		print(mytable3[5])     --输出 C

		--[[
		2.元表中的__newindex后跟一个表，元表对应的普通表新增元素会放入__newindex后跟的表内
		--]]

		mytable4={"Lua","C#","Python","Go"}
		newtable2={}
		metatable4={
		__newindex=newtable2
		}
		mytable4=setmetatable(mytable4,metatable4)

		mytable4[5]="C++"
		print(mytable4[5])          --输出nil
		print(newtable2[5])         --输出C++

		--给表添加操作符[__add对应 两个表相加的操作]
		mytable5={"Lua","C#","Python","Go"}
		newtable5={"PHP","Java"}
		mymetatable5={
		__add=function(tab,newtab)
			local length=#tab
			for k,v in pairs(newtab) do
				tab[length+k]=v
			end
			return tab
		end
		}
		mytable5=setmetatable(mytable5,mymetatable5)
		v1=newtable5+mytable5
	
		for k,v in pairs(v1) do
			print(k,v)
		end
		--两个表使用+操作符，只要其中一个表的元表定义了__add的元方法就可以进行相加操作

		--__Call元方法，可以将表作为函数使用
		mytable6={"Lua","C#","Python","Go"}
		mymetatable6={
		__call=function(tab,arg)
			print(arg)
			return arg
		end
		}
		mytable6=setmetatable(mytable6,mymetatable6)
		print(mytable6("Java"))
		--[[输出
		Java   --__call调用输出一次
		Java
		--]] 

		
----------
参考资料：    
学习网址：[http://www.runoob.com/lua/lua-tutorial.html](http://www.runoob.com/lua/lua-tutorial.html)



