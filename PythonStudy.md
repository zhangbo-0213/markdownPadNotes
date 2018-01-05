Python基本语法：      
1. 字符串可以同时使用单引号''或同时使用双引号" "，不能一边单引号一边双引号               
2. 行尾没有分号        
3. 变量名命名即使用，没有类型声明         
4. 条件判断时，判断语句可以不使用括号，**判断句尾需要加冒号：** 语句之块之间的从属关系通过缩进进行区别，elif表示 else if   

	userInput=raw_input('Please input your answer Y or N')
	if (userInput=='Y'):
    		print 'right'
	elif userInput=='N':
   		print 'error'
	else:
   		print 'Invaild Input'	         