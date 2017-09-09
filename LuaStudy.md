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
nil类型表示一个无效值，对于全局变量和table，nil还有**清除的作用**，释放变量所占用的内存空间 

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
