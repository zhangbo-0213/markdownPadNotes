Python基本语法：      
1. 字符串可以同时使用单引号''或同时使用双引号" "，不能一边单引号一边双引号               
2. 行尾没有分号        
3. 变量名命名即使用，没有类型声明         
4. 条件判断时，判断语句可以不使用括号，**判断句尾需要加冒号：** 语句之块之间的从属关系通过缩进进行区别，elif表示 else if            
5. print输出默认转行      
6. import关键字引入所需模块，就和C中包含头文件一样，例如引入画图模块，import turtle      
7. Python中不支持自增自减操作符，类似操作通过赋值完成 i+=1         

关于模块：      
**turtle**           

	turtle.forward()   
	turtle.right()
	turtle.penup()/pendown()/goto()   
 	turtle.fillcolor('color')/begin_fill()/end_fill()      
	// 起始点到结束点连成线构成的封闭区域上色       
	turtle.color('color') //画笔变为指定颜色，不会填充区域
	turtle.done()命令，结束等待 