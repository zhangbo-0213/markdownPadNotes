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