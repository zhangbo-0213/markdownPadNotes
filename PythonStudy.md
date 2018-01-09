### Python基本语法   ###
1. 字符串可以同时使用单引号''或同时使用双引号" "，不能一边单引号一边双引号               
2. 行尾没有分号        
3. 变量名命名即使用，没有类型声明         
4. 条件判断时，判断语句可以不使用括号，**判断句尾需要加冒号：** 语句之块之间的从属关系通过缩进进行区别，elif表示 else if            
5. print输出默认转行      
6. import关键字引入所需模块，就和C中包含头文件一样，例如引入画图模块，import turtle      
7. Python中不支持自增自减操作符，类似操作通过赋值完成 i+=1         
8. 通过 import module as othername给模块别名，调用模块函数时可以使用别名        
9. 常见基本数据类型：   
    字符串 ‘str’   
    整数 123
    浮点数 3.14
    bool     true/false
10. 单行注释：  #当行注释内容       
      多行注释   ''' 多行注释内容 '''  或  """  多行注释内容"""         
11. 
### 关于函数###
**1.input()&raw_input()**         
在Python2中，input()和raw-input()均可以接受用户输入，区别在于：       
input() 获取到的内容会转化为相应的类型并返回，例如输入 123 会以整数的形式返回，因此如果要想接收输入的字符串  必须输入时 带引号，不带引号建会返回一个未声明的变量名         
而raw-input()则是将所有的输入均转化为字符串，因此不需要加引号

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
