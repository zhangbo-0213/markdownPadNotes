## Python3网络爬虫实战笔记 ##
### 环境配置 ###
- **基础环境配置**    
Python3       
pip包管理工具     

- **数据库配置**     
MongoDB  非关系型数据库工具  [下载地址](https://www.mongodb.com/download-center?jmp=nav#community)   
robomongo MongoDB可视化管理工具 [下载地址](https://robomongo.org/download)     
Redis 非关系型数据库工具  [下载地址](https://github.com/MicrosoftArchive/redis/releases)    
RedisDesktopManager Redis可视化管理工具 [下载地址](https://github.com/uglide/RedisDesktopManager/releases)   
MySQL 关系型数据库工具 [下载地址](https://www.mysql.com/downloads/)   
MySQL-Front MySQL可视化管理工具 [下载地址](https://mysql-front.en.softonic.com/)     

- **常用库&工具**     
**内置库**     
*urllib* 	网络连接，请求库      
*re*        正则表达式解析库         
**外置库**     
*requests* 	网络请求库 （适用于静态加载的网页）  
*selenium* 	用于浏览器驱动（有的网站是通过JS渲染显示页面内容，使用requests库拿不到目标数据），做网站 自动化测试      
*lxml* 	提供xpath解析方式，相对原生的xml库，使用更加方便       
*beautifulsoup* 		网页解析库，依赖lxml库，因此在安装前先安装lxml库    
*pyquery*	网页解析库，使用更加方便，语法与Jquery基本一致       
*pymysql*	存储库操作    操作mysql数据库           
*pymongo*	存储库操作	操作mongodb数据库       
*redis* 	存储库操作	操作Redis数据库         
*flask*    处理代理操作        
*django*   web服务器框架，提供后台管理，模板引擎，接口，路由，通过django可以搭建网站，在爬虫应用中用于分布式爬虫管理        
*jupyter*	网页端运行的记事本， 可以对代码进行调试，支持markdown语法                              
 **工具**  
*Chromedriver* 浏览器驱动 （注意Chromedriver与chrome的版本对应）   [下载地址](http://chromedriver.storage.googleapis.com/index.html)   
*phantomjs* 无界面浏览器工具，做爬虫时无需弹出自动化测试的界面   [下载地址](http://phantomjs.org/download.html)          

### 库学习 ###

 **Urllib**   
Python内置的HTTP请求库     
urllib.request  	请求模块   
urllib.error	 异常处理模块    
urllib.parse 	url解析模块  
urllib.robotparser 	robots.txt解析模块

 **Requests**      

Requests库是使用Python编写,基于Urllib,比Urllib使用更加方便的HTTP第三方库

	import requests
	import json
	from requests import Timeout, ReadTimeout,ConnectionError,RequestException
实例引入

	response=requests.get('http://www.baidu.com/')
	print(type(response))
	print(response.status_code)
	print(type(response.text))
	print(response.text)
	print(response.cookies)      
各种请求方式

	#使用 http://httpbin.org 进行测试    
	requests.post('http://httpbin.org/post')
	requests.put('http://httpbin.org/put')
	requests.delete('http://httpbin.org/delete')
	requests.head('http://httpbin.org/get')
	requests.options('http://httpbin.org/get')
基本Get请求

	response=requests.get('http://httpbin.org')
	print(response.text)
带参数的Get请求

	data={
	    'user':'majortom',
	    'age':24
	}
	response=requests.get('http://httpbin.org/get',params=data)
	print(response.text)
josn解析
	
	response=requests.get('http://httpbin.org/get')
	print(type(response.text))
	print(response.json())
	print(json.loads(response.text))
	print(type(response.json()))

获取二进制数据(下载图片，视频信息时常用)

	#response.content
	response=requests.get('https://github.com/favicon.ico')
	print(type(response.text))
	print(response.text)
	print(type(response.content))
	print(response.content)
	#下载保存图片
	response=requests.get('http://github.com/favicon.ico')
	with open('favicon.ico','wb+') as f:
    		f.write(response.content)
    		f.close()
	print('下载完成')

添加Headers（实现对浏览器的伪装）
在爬虫中，如果对目标站点发起请求时，不添加任何信息，可能会被拒绝访问
例如：

	response=requests.get('http://www.zhihu.com/explore')
	print(response.text)
	#输出：
	#<html><body><h1>500 Server Error</h1>
	#An internal server error occured.
	#</body></html>

添加headers
	
	headers={'User-Agent':'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/64.0.3282.186 Safari/537.36'
         }
	response=requests.get('http://www.zhihu.com/explore',headers=headers)
	print(response.text)
	#正常获取知乎发现页面的html内容

基本POST请求    
Post请求需要在发出请求时提交表单(form-data)信息
	
	data={'name':'MajorTom',
      'age':25}
	headers={
    'User-Agent':'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/64.0.3282.186 Safari/537.36'
	}
	response=requests.post('http://httpbin.org/post',data=data,headers=headers)
	print(response.text)

响应
	
	response属性
	response=requests.get('http://www.jianshu.com')
	print(type(response.status_code),response.status_code)
	print(type(response.headers),response.headers)
	print(type(response.cookies),response.cookies)
	print(type(response.url),response.url)
	print(type(response.history),response.history)

状态码判断

	response=requests.get('http://www.jianshu.com')
	exit() if not response.status_code==200 else print('Request Successfully')

Requests高级操作
	
	文件上传
	file={'file':open('favicon.ico','rb')}
	#通过字典保存需要上传的文件
	response=requests.post('http://httpbin.org/post',files=file)
	print(response.text)

获取cookie

	response=requests.get('http://www.baidu.com')
	print(response.cookies)
	print(type(response.cookies.items()))
	for key,values in response.cookies.items():
	    print(key+'='+values)

会话维持
设置cookies,模拟登陆过程

	requests.get('http://httpbin.org/cookies/set/number/123456789')
	#使用httpbin/cookies/set 设置cookies信息
	response=requests.get('http://httpbin.org/cookies')
	#拿到当前请求的cookies
	print(response.text)
	#输出结果：  "cookies": {}
cookies为空,这是由于两次get请求是相互独立的，相当于使用一个浏览器访问一个cookies，
使用另外一个浏览器再去获取cookies,因此获取不到cookies信息

	#如何模拟使用一个浏览器去设置cookies,再去获取cookies信息？
	#使用Session对象，再用Session对象发起两次请求，分别设置和获取cookies
	s=requests.Session()
	s.get('http://httpbin.org/cookies/set/number/123456789')
	response=s.get('http://httpbin.org/cookies')
	print(response.text)
	#输出
	#"cookies": {
	#    "number": "123456789"
	#  }


证书验证

	#当使用requests.get请求一个https协议的站点，首先会检测证书是否合法
	#response=requests.get('https://www.12306.cn')
	#print(response.status_code)
	#输出：requests.exceptions.SSLError:
	# HTTPSConnectionPool(host='www.12306.cn', port=443):
	# Max retries exceeded with url:
	# / (Caused by SSLError(SSLError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed (_ssl.c:777)'),))
	# 当证书不合法时，会出现错误信息（SSL_ERROR）

	#可以设置不进行证书验证（默认开启验证）
	response=requests.get('https://www.12306.cn',verify=False)
	print(response.status_code)
	#输出：200
	#同时会有 未验证 警告

可以手动设置cert参数，指定本地证书

	#代理设置
	proxies={
	    'http':'http://127.0.0.1:9743',
	    'https':'https://127.0.0.1:9743'
	}

	#如果使用有密码的代理:
	proxies={'http':'http://user:password@127.0.0.1:9743/'}

	#response=requests.get("http://www.taobao.com",proxies=proxies)
	#requests中直接将代理 proxies 作为get的参数传递即可完成代理，而不必构造
	#ProxyHandler对象，opener对象，使用open()发起请求等

超时设置
	
	try:
	    response=requests.get('http://www.baidu.com',timeout=0.01)
	    print(response.status_code)
	except Timeout:
	    print('TIMEOUT')

	#认证设置
	#当某些站点请求时需要提供用户名密码认证
	#在get申请中提供 auth 参数
	#response=requests.get('http://120.37.34.24:9001',auth=('user','123'))
	#print(response.status_code)


异常处理

	try:
    		response=requests.get('http://www.baidu.com',timeout=0.05)
    		print(response.status_code)
	except ReadTimeout:
    		print('TimeOut')
	except ConnectionError:
    		print('ConnectionError')
	except RequestException:
    		print('RequestError')       


**re（正则库）**    

	import re
	#http://tool.oschina.net/regex  开源中国正则表达式在线测试工具（常用正则表达式）

re.match

	#re.match 尝试从字符串的起始位置(第一个字符)匹配一个模式，如果不是起始位置匹配成功的话，match()就返回none
	#re.match(pattern,string,flags=0)


	content='Hello 123 4567 World_This is a Regex Demo'

最常规匹配

	print(len(content))
	result=re.match('^Hello\s\d\d\d\s\d{4}\s\w{10}.*Demo$',content)
	print(result)
	print(result.group())
	print(result.span())
	#输出
	#41
	#<_sre.SRE_Match object; span=(0, 41), match='Hello 123 4567 World_This is a Regex Demo'>
	#Hello 123 4567 World_This is a Regex Demo
	#(0, 41)  span()匹配到的字符串长度

泛匹配
	
	result=re.match('^Hello.*Demo$',content)
	print(type(result))
	print(result.group())
	#输出
	#Hello 123 4567 World_This is a Regex Demo

目标匹配
	
	#使用()确定选取目标,通过group()中传入参数确定选取部分
	content='Hello 1234567 World_This is a Regex Demo'
	result=re.match('^Hello\s(\d+)\s.*Demo$',content)
	print(result.group(1))
	print(result.span())
	#输出
	#1234567
	#(0, 40)

贪婪匹配   .*

	content='Hello 1234567 World_This is a Regex Demo'
	result=re.match('^H.*(\d+).*Demo$',content)
	print(result)
	print(result.group(1))
	#输出
	#<_sre.SRE_Match object; span=(0, 40), match='Hello 1234567 World_This is a Regex Demo'>
	#7
	#输出结果并不是 1234567 是由于.*会尽可能保证覆盖尽可能多的字符内容，而\d+ 表示至少一个数字
	#因此前面的123456被.*所覆盖

非贪婪匹配  .*?
	
	result=re.match('^H.*?(\d+).*Demo$',content)
	print(result)
	print(result.group(1))
	#输出
	#<_sre.SRE_Match object; span=(0, 40), match='Hello 1234567 World_This is a Regex Demo'>
	#1234567

匹配模式

	content='Hello 1234567 World_This \nis a Regex Demo'
	result=re.match('^He.*?(\d+).*?Demo$',content)
	print(result)
	#输出结果
	#None   content中包含换行符，而 . 是不能匹配换行符，因此无法匹配出正确结果

	#使用 re.S 更换匹配模式，. 就可以匹配任一字符，包括换行符
	content='Hello 1234567 World_This \nis a Regex Demo'
	result=re.match('^He.*?(\d+).*?Demo$',content,re.S)
	print(result)
	print(result.group(1))
	#输出
	#<_sre.SRE_Match object; span=(0, 41), match='Hello 1234567 World_This \nis a Regex Demo'>
	#1234567

转义

	content='price is $5.00'
	result=re.match('price is $5.00',content)
	print(result)
	#输出
	#None
	#原因在于 $ . 在正则表达式中有特殊含义 如果要表示原本的字符意义，需要使用转义
	result=re.match('price is \$5\.00',content)
	print(result)
	#输出
	#<_sre.SRE_Match object; span=(0, 14), match='price is $5.00'>
	
re.match()是从给定的字符串的第一个字符开始匹配，如果第一个字符不符合正则表达式，
则无法得到正确目标


re.search           
re.search（）是从给定字符串扫描，然后返回第一个成功的匹配，      
也就是正则表达式不用从字符串的一开始就书写规则，可以只写筛选目标的局部规则      
	
	content='Extra stings Hello 1234567 World_This is a Regex Demo Extra stings'
	result=re.match('Hello.*?(\d+).*?Demo',content)
	print(result)
	#输出 None
	result=re.search('Hello.*?(\d+).*?Demo',content)
	print(type(result))
	print(result)
	print(result.group(1))
	#输出
	# <_sre.SRE_Match object; span=(13, 53), match='Hello 1234567 World_This is a Regex Demo'>
	# 1234567
	
匹配练习

	html='''<div id="songs-list">
		<h2 class="title">经典老歌</h2>
		<p class="introduction">
		</p>
		<ul id="list" class="list-group">
			<li data-view="2">一路上有你</li>
			<li data-view="7">
				<a herf="/2.mp3" singer="任贤齐">沧海一声笑</a>
			</li>
			<li data-view="4" class="active">
				<a herf="/3.mp3" singer="齐秦">往事随风</a>
			</li>
				<li data-view="6"><a herf="/4.mp3" singer="beyond">光辉岁月</a></li>
				<li data-view="5"><a herf="/5.mp3" singer="陈慧琳">记事本</a></li>
			</li>
			<li data-view="5">
				<a herf="/6.mp3" singer="邓丽君">但愿人长久</a>
			</li>
		</ul>
	</div>'''
	result=re.search('<a.*?singer="(.*?)">(.*?)</a>',html,re.S)
	print(result)
	#<_sre.SRE_Match object; span=(173, 212), match='<a herf="/2.mp3" singer="任贤齐">沧海一声笑</a>'>
re.search()只返回第一个符合要求的结果  

re.findall 查询所有符合条件的所有结果

	result=re.findall('<a.*?herf="(.*?)".*?singer="(.*?)">(.*?)</a>',html,re.S)
	print(result)
	#输出
	#[('/2.mp3', '任贤齐', '沧海一声笑'), ('/3.mp3', '齐秦', '往事随风'), ('/4.mp3', 'beyond', '光辉岁月'), ('/5.mp3', '陈慧琳', '记事本'), ('/6.mp3', '邓丽君', '但愿人长久')]
	#返回列表，每个元素为一个元组
	for items in result:
    		print(items[0],items[1],items[2])
	#输出
	#/2.mp3 任贤齐 沧海一声笑
	#/3.mp3 齐秦 往事随风
	#/4.mp3 beyond 光辉岁月
	#/5.mp3 陈慧琳 记事本
	#/6.mp3 邓丽君 但愿人长久

re.compile      
将正则字符串编译成为正则表达式对象,以便复用这个对象

	content='''Hello 1234567 World_This 
       	 is a Regex Demo'''
	pattern=re.compile('Hello.*?Demo',re.S)
	result=re.findall(pattern,content)
	print(result)
	#输出
	#['Hello 1234567 World_This \n        is a Regex Demo']


**BeautifulSoup**     
灵活高效的网页解析库，支持多种解析器        
利用它不用编写正则表达式即可方便地实现网页信息的提取    
安装 pip3 install beautifulsoup4    

	html='''<html><head><title>The Dormouse's story</title></head>
	<body>
	<p class="title" name="dromouse"><b>The Dormouse's story</b></p>
	<p class="story">Once upon a time there were three little sisters; and their names were
	<a href="http://example.com/elsie" class="sister" id="link1"><!-- Elsie --></a>,
	<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and 
	<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
	and they lived at the bottom of a well.</p>
	<p class="story">...</p>
	'''
 用法实例

	# 构建BeautifulSoup对象，传入指定的html文本和解析器（这里选择lxml解析器）
	soup=BeautifulSoup(html,'lxml')
	# soup.prettify() 方法将传入的beautifulsoup对象的html文本自动补全(标签对自动补全)，形成标准的html文本
	print(soup.prettify())
	# 提取soup对象的html文本中title标签内的内容
	print(soup.title.string)       

标签选择器   

1.选择元素（结果包含标签本身）   
	
	print(soup.title)
	print(type(soup.title))
	print(soup.head)
	print(soup.p) 
	#标签只返回第一个标签匹配结果
 2.获取名称

	print(soup.title.name)
	# 返回最外层标签名  
3.获取属性（两种获取标签内对应属性名的值）         
	
	print(soup.p.attrs['name'])
	print(soup.p['name'])      
4.获取标签内容

	print(soup.p.string)       

5.嵌套选择(标签内包含标签进一步选择)

	print(soup.head.title.string)            
print(soup.p.contents)    
 
6.子节点和子孙节点（标签之间包含层级关系，如何选择子结点）   

	print(soup.p.contents)
	# 返回结果为一个列表，列表元素为每一个闭合的子标签及其内容
	
	# 返回子节点的另一种方法
	print(soup.p.children)  
	#soup.p.children是list_iterator object迭代器类型
	for i,child in enumerate(soup.p.children):
    		print(i,child)

	# 返回子孙结点
	print(soup.p.descendants)
	for i,child in enumerate(soup.p.descendants):
    		print(i,child)

	# 返回父节点
	print(soup.a.parent)

	# 返回祖先节点
	print(list(enumerate(soup.a.parents)))

	# 返回兄弟结点
	print(list(enumerate(soup.a.next_siblings)))
	print(list(enumerate(soup.a.previous_siblings)))     

标准选择器            
	
	find_all(name,attrs,recursive,text,**kwargs)
	
		html='''<div class="pannel">
	    <div class="panel-heading">
	        <h4>Hello</h4>
	    </div>
	    <div class="panel-body">
	        <ul class="element" id="list-1" name="elements">
	            <li class="element">Foo</li>
	            <li class="element">Bar</li>
	            <li class="element">Jay</li>
	        </ul>
        	<ul class="list list-small" id="list-2">
            	<li class="element">Foo</li>
            	<li class="element">Bar</li>
        	</ul>
    	</div>
	</div>
	'''     
1.find_all    

	soup=BeautifulSoup(html,'lxml')
	print(soup.find_all('ul'))
	print(type(soup.find_all('ul')[0]))
	# 输出：<class 'bs4.element.Tag'>      

2.使用find_all嵌套查找目标标签的子标签
	
	soup=BeautifulSoup(html,'lxml')
	ul=soup.find_all('ul')
	for li in ul:
    		print(li.find_all('li'))       

3.attrs参数(标签内属性)

	soup=BeautifulSoup(html,'lxml')
	prin	t(soup.find_all(attrs={'id':'list-1'}))
	print(soup.find_all(id='list-1'))
	print(soup.find_all(attrs={'name':'elements'}))      


4.根据text(文本内容选择)

	soup=BeautifulSoup(html,'lxml')
	print(soup.find_all(text='Foo'))
	# 输出:['Foo', 'Foo']
	# 输出结果只包含文本，并不包括所在的标签   

5.find方法：返回单个元素，find_all方法：返回所有元素       
	
	#find(name,attrs,recurive,text,**kwargs)
	# find与find_all用法一致，只是返回结果不同
	soup=BeautifulSoup(html,'lxml')
	print(soup.find('ul'))
	print(type(soup.find('ul')))
	print(soup.find('page'))      

	#find_parent() find_parents()
	#返回父节点，返回祖先节点

	#find_next_sibling() find_next_siblings()
	#返回下一个兄弟节点，返回后面的所有兄弟节点

	#find_previous_sibling() find_previous_siblings()
	#返回前一个兄弟节点，返回前面所有兄弟节点

	#find_all_next find_next()
	#返回节点后所有符合条件的节点，返回第一个符合条件的节点

	#find_all_previous() find_previous()
	#返回节点前所有符合条件的节点 返回前面第一个符合条件的节点

CSS选择器       
通过select()直接传入CSS选择器即可完成选择

	html='''<div class="panel">
	    <div class="panel-heading">
	        <h4>Hello</h4>
	    </div>
	    <div class="panel-body">
	        <ul class="element" id="list-1" name="elements">
	            <li class="element">Foo</li>
	            <li class="element">Bar</li>
	            <li class="element">Jay</li>
	        </ul>
	        <ul class="list list-small" id="list-2">
	            <li class="element">Foo</li>
	            <li class="element">Bar</li>
	        </ul>
	    </div>
	</div>
	'''
	#使用select()方法选择在传入参数时，
	# 若选择class 需要在class名前加.
	# 若选择id 需要在id名前加#
	# 直接选择标签时，不需要加符号
	# 连续选择时，各个选择参数之间使用 空格
	soup=BeautifulSoup(html,'lxml')
	print(soup.select('.panel .panel-heading'))
	print(soup.select('ul li'))
	print(soup.select('#list-2 .element'))
	print(type(soup.select('ul')[0]))

1.CSS选择器嵌套选择

	soup=BeautifulSoup(html,'lxml')
	for ul in soup.select('ul'):
    		print(ul.select('li'))

2.获取属性(使用[])

	soup=BeautifulSoup(html,'lxml')
	for ul in soup.select('ul'):
    		print(ul['id'])
    		print(ul.attrs['id'])
	#输出：
	#list-1
	#list-1
	#list-2
	#list-2

3.获取内容(get_text)

	soup=BeautifulSoup(html,'lxml')
	for li in soup.select('li'):
    		print(li.get_text())   

**PyQuery**   
 PyQuery      
 强大灵活的网页解析库，与jQuery语法类似       
pip3 install pyquery   

	from pyquery import PyQuery as pq

**初始化**

	html='''
	<div class="warp">
	<div id="container">
	    <ul class="list">
	        <li class="item-0">first item</li>
	        <li class="item-1"><a href="link2.html">second item</a></li>
	        <li class="item-0 active"><a href="link3.html"><span class="bold">third item</span></a></li>
	        <li class="item1 active"><a href="link4.html">fourth item</a></li>
	        <li class="item-0"><a href="link5.html">fifth item</a></li>
	    </ul>
	</div>
	</div>
	'''
字符串初始化

	doc=pq(html)
	print(doc('li'))
	# PyQuery所使用的是CSS选择器，.选择class, #选择id 标签直接传入

URL初始化

	doc=pq(url='http://www.baidu.com')
	print(doc('head'))

文件初始化

	doc=pq(filename='pyquery_test.html')
	print(doc('li'))

**基本CSS选择器**

	doc=pq(html)
	print(doc('#container .list li'))

**查找元素**    
子元素

	doc=pq(html)
	items=doc('.list')
	print(type(items))
	# <class 'pyquery.pyquery.PyQuery'>
	print(items)
	lis=items.find('li')
	print(type(lis))
	# <class 'pyquery.pyquery.PyQuery'>
	print(lis)
	# 每一次选择都是PyQuery对象，因此可以通过层层嵌套选择

查找所有的直接子元素

	lis=items.children()
	print(type(lis))
	# <class 'pyquery.pyquery.PyQuery'>
	print(lis)

	lis=items.children('.active')
	print(lis)
	
父元素

	doc=pq(html)
	items=doc('.list')
	contanier=items.parent()
	print(type(contanier))
	# <class 'pyquery.pyquery.PyQuery'>
	print(contanier)
	# 返回父标签里的所有内容

所有的父级及以上的标签

	contaniers=items.parents()
	print(type(contaniers))
	# <class 'pyquery.pyquery.PyQuery'>
	print(contaniers)
	# 再进行一次筛选
	parent=contaniers('.warp')
	print(parent)

兄弟元素

	doc=pq(html)
	li=doc('.list .item-0.active')
	# 在使用选择器连续选择时，条件之间空格表示条件之间的层级关系 没有空格则表示条件之间的并列关系
	# 上述选择器选择class为list的子标签下同时满足class为item-0和active的标签
	print(li.siblings())
	# 继续筛选
	print(li.siblings('.item1'))

遍历 .items()方法 将多个匹配结果转化为可枚举类型

	doc=pq(html)
	lis=doc('li').items()
	print(type(lis))
	# <class 'generator'>
	for li in lis:
	    print(type(li))
    # 单个元素类型还是<class 'pyquery.pyquery.PyQuery'>
    print(li)

**获取信息**     
获取属性

	doc=pq(html)
	a=doc('.item-0.active a')
	print(a)
	# <a href="link3.html"><span class="bold">third item</span></a>
	print(a.attr('href'))
	# link3.html
	print(a.attr.href)
	# link3.html

获取文本(被标签包含的文本内容) .text()

	a=doc('.item-0.active a')
	print(a)
	# <a href="link3.html"><span class="bold">third item</span></a>
	print(a.text())
	# third item

获取HTML .html()

	li=doc('.item-0.active')
	print(li)
	# <li class="item-0 active"><a href="link3.html"><span class="bold">third item</span></a></li>
	print(li.html())
	# <a href="link3.html"><span class="bold">third item</span></a>

DOM操作

 add_Class() remove_Class()

	doc=pq(html)
	li=doc('.item-0.active')
	print(li)
	# <li class="item-0 active"><a href="link3.html"><span class="bold">third item</span></a></li>
	li.remove_class('active')
	print(li)
	# <li class="item-0"><a href="link3.html"><span class="bold">third item</span></a></li>
	li.add_class('active')
	print(li)
	# <li class="item-0 active"><a href="link3.html"><span class="bold">third item</span></a></li>

attr,css 修改或增加属性，修改或增加 style

	doc=pq(html)
	li=doc('.item-0.active')
	print(li)
	# <li class="item-0 active"><a href="link3.html"><span class="bold">third item</span></a></li>
	li.attr('name','link')
	print(li)
	# <li class="item-0 active" name="link"><a href="link3.html"><span class="bold">third item</span></a></li>
	li.css('font-size','14px')
	print(li)
	# <li class="item-0 active" name="link" style="font-size: 14px"><a href="link3.html"><span class="bold">third item</span></a></li>

remove() 删除指定的标签

	html='''
	    <div class="warp">
	        Hello,world
	        <p>This is a paragraph.</p>
	        </div>
	'''
	doc=pq(html)
	div=doc('.warp')
	print(div.text())
	# Hello,world
	# This is a paragraph.
	div.remove('p')
	print(div)
	# <div class="warp">
	#        Hello,world
	#        </div>
	print(div.text())
	# Hello,world

 伪类选择器

	html='''
	<div class="warp">
	<div id="container">
	    <ul class="list">
	        <li class="item-0">first item</li>
	        <li class="item-1"><a href="link2.html">second item</a></li>
	        <li class="item-0 active"><a href="link3.html"><span class="bold">third item</span></a></li>
	        <li class="item1 active"><a href="link4.html">fourth item</a></li>
	        <li class="item-0"><a href="link5.html">fifth item</a></li>
	    </ul>
	</div>
	</div>
	'''
	doc=pq(html)
	li=doc('li:first-child')
	# 选取li标签中的 第一个
	print(li)
	# <li class="item-0">first item</li>

	li=doc('li:last-child')
	# 选取li标签中的 最后一个
	print(li)
	# <li class="item-0"><a href="link5.html">fifth item</a></li>
	
	li=doc('li:nth-child(2)')
	# 选取li标签中的 第二个
	print(li)
	# <li class="item-1"><a href="link2.html">second item</a></li>
	
	li=doc('li:gt(2)')
	# 选取li标签中的 序号比2大的
	print(li)
	# <li class="item1 active"><a href="link4.html">fourth item</a></li>
	# <li class="item-0"><a href="link5.html">fifth item</a></li>

	li=doc('li:nth-child(2n)')
	# 选取li标签中的 序号为偶数的
	print(li)
	# <li class="item-1"><a href="link2.html">second item</a></li>
	# <li class="item1 active"><a href="link4.html">fourth item</a></li>
	
	li=doc('li:contains(second)')
	# 选取li标签中的 文本中包含"second"的
	print(li)
	# <li class="item-1"><a href="link2.html">second item</a></li>

**Selenuim**       
selenium 自动化测试工具，支持多种浏览器        
爬虫中主要用来解决JavaScript渲染的问题，在使用Urllib或Requests获取请求无法解析       
目标站点可能是使用JavaScript进行渲染，可以使用Selenium库进行解析                  

pip3 install selenium


	import time
	from selenium import webdriver
	from selenium.common.exceptions import NoSuchElementException, TimeoutException
	from selenium.webdriver import ActionChains
	from selenium.webdriver.common.by import By
	from selenium.webdriver.common.keys import Keys
	from selenium.webdriver.support import expected_conditions as EC
	from selenium.webdriver.support.wait import WebDriverWait

**基本用法**       
  
 声明一个浏览器驱动对象     

	browser=webdriver.Chrome()
	try:
    	    	browser.get('http://www.baidu.com')
    		input=browser.find_element_by_id('kw')
    		# 输入关键字
    		input.send_keys('Python')
    		# 输入回车
    		input.send_keys(Keys.ENTER)
    		# 设置等待
   			wait=WebDriverWait(browser,10)
    		# 等待元素ID 为 ‘content_left’ 的元素被加载出来
    		wait.until(EC.presence_of_element_located((By.ID,'content_left')))

   		 print(browser.current_url)
   		 print(browser.get_cookies())
   		 print(browser.page_source)
	finally:
    		browser.close()


 声明浏览器对象      

	browser=webdriver.Chrome()
	# browser=webdriver.Firefox()
	# browser=webdriver.Edge()
	# browser=webdriver.PhantomJS()
	# browser=webdriver.Safari()
	browser.close()

访问页面      

	browser=webdriver.Chrome()
	browser.get('https://www.taobao.com')
	print(browser.page_source)
	browser.close()

**查找元素**    
通过选择器查找单个元素（如：找到对应的输入框输入内容或者找到按钮进行点击）

	browser=webdriver.Chrome()
	browser.get('https://www.taobao.com')
	input_1=browser.find_element_by_id('q')
	input_2=browser.find_element_by_css_selector('#q')
	input_3=browser.find_element_by_xpath('//*[@id="q"]')
	input_4=browser.find_element(By.ID,'q')
	print(input_1,input_2,input_3,input_4)
	browser.close()
	#  常用查找元素方法
	# browser.find_element_by_name()
	# browser.find_element_by_xpath()
	# browser.find_element_by_link_text()
	# browser.find_element_by_partial_link_text()
	# browser.find_element_by_tag_name()
	# browser.find_element_by_class_name()
	# browser.find_element_by_css_selector()

查找多个元素    

	browser=webdriver.Chrome()
	browser.get('https://www.taobao.com')
	lis=browser.find_elements(By.CSS_SELECTOR,'.service-bd li')
	print(lis)
	browser.close()

	# 常用查找多个元素方法
	# browser.find_elements_by_name()
	# browser.find_elements_by_xpath()
	# browser.find_elements_by_link_text()
	# browser.find_elements_by_partial_link_text()
	# browser.find_elements_by_tag_name()
	# browser.find_elements_by_class_name()
	# browser.find_elements_by_css_selector()

**元素交互操作**       

	# 对获取的元素调用交互方法（获取到文本框或按钮）
	browser=webdriver.Chrome()
	browser.get('https://www.taobao.com')
	# 找到输入框的元素 id 并选择该元素
	input=browser.find_element_by_id('q')
	# 对文本框元素输入 搜索内容
	input.send_keys('iPhone')
	time.sleep(1)
	input.clear()
	input.send_keys('iPad')
	button=browser.find_element_by_class_name('btn-search')
	button.click()


**交互动作**       
将动作附加到动作链中串行执行（并非单一点击动作，而是连续动作，例如拖拽，需要引入 ActionChains）      

	browser=webdriver.Chrome()
	url='http://www.runoob.com/try/try.php?filename=jqueryui-api-droppable'
	browser.get(url)
	browser.switch_to.frame('iframeResult')
	source=browser.find_element_by_id('draggable')
	target=browser.find_element_by_id('droppable')
	actions=ActionChains(browser)
	actions.drag_and_drop(source,target)
	actions.perform()
	# 更多操作参考：http://selenium-python.readthedocs.io/api.html#module-selenium.webdriver.common.action_chains

**执行JavaScript  browser.execute_script()**     
传入JS命令执行动作   
例如打开知乎网页，下拉到最下端，然后进行提示    

	browser=webdriver.Chrome()
	browser.get('https://www.zhihu.com/explore')
	browser.execute_script('window.scrollTo(0,document.body.scrollHeight)')
	browser.execute_script('alert("To Bottom")')

**获取元素信息**       
获取属性     

	browser=webdriver.Chrome()
	browser.get('http://www.zhihu.com/explore')
	logo=browser.find_element_by_id('zh-top-link-logo')
	print(logo)
	print(logo.get_attribute('class'))

获取文本值    

	browser=webdriver.Chrome()
	browser.get('http://www.zhihu.com/explore')
	input=browser.find_element_by_class_name('zu-top-add-question')
	print(input.text)

获取ID、位置、标签、大小    

	browser=webdriver.Chrome()
	browser.get('http://www.zhihu.com/explore')
	input=browser.find_element_by_class_name('zu-top-add-question')
	print(input.id)
	print(input.location)
	print(input.tag_name)
	print(input.size)

Frame 网页中独立的窗口，需要从父窗口切换到子窗口再进行元素选择和操作    

	browser=webdriver.Chrome()
	browser.get('http://www.runoob.com/try/try.php?filename=jqueryui-api-droppable')
	browser.switch_to.frame('iframeResult')
	source=browser.find_element_by_id('draggable')
	try:
    		logo=browser.find_element_by_class_name('logo')
	except NoSuchElementException:
    		print('No Logo')
	browser.switch_to.parent_frame()
	logo=browser.find_element_by_class_name('logo')
	prin	t(logo)
	print(logo.text)

**等待**    

 隐式等待(网络情况不良时可以进行等待后再查找)         
当使用了隐式等待执行测试地时候，如果WebDriver没有在DOM中找到目标元素，将继续等待，超出设定时间后则抛出找不到元素异常      
也就是说，当查找元素没有立即出现的时候，隐式等待会等待一段时间后再进行DOM元素的查找，默认时间是0       

	browser=webdriver.Chrome()
	browser.get('http://www.zhihu.com/explore')
	browser.implicitly_wait(10)
	input=browser.find_element_by_id('zh-top-nav-explore')
	print(input.text)

显式等待       
指定一个条件和显式等待时间，如果指定时间内条件不成立会一直等待，直到超时等待时间条件不成立会抛出异常，指定时间内成立返回内容       
确保交互动作前需要等待的操作能正常进行不中断   

	browser=webdriver.Chrome()
	browser.get('https://www.taobao.com')
	wait=WebDriverWait(browser,10)
	input=wait.until(EC.presence_of_element_located((By.ID,'q')))
	button=wait.until(EC.element_to_be_clickable((By.CSS_SELECTOR,'.btn-search')))
	browser.execute_script('alert("Alert")')
	alert=wait.until(EC.alert_is_present())
	print(input,button,alert)

	# 常用的判断条件：
	# title_is 标题是某内容
	# title_contains 标题包含某内容
	# presence_of_element_located 元素加载出，传入定位元组，如(By.ID, 'p')
	# visibility_of_element_located 元素可见，传入定位元组
	# visibility_of 可见，传入元素对象
	# pr	esence_of_all_elements_located 所有元素加载出
	# text_to_be_present_in_element 某个元素文本包含某文字
	# text_to_be_present_in_element_value 某个元素值包含某文字
	# frame_to_be_available_and_switch_to_it frame加载并切换
	# invisibility_of_element_located 元素不可见
	# element_to_be_clickable 元素可点击
	# staleness_of 判断一个元素是否仍在DOM，可判断页面是否已经刷新
	# element_to_be_selected 元素可选择，传元素对象
	# element_located_to_be_selected 元素可选择，传入定位元组
	# element_selection_state_to_be 传入元素对象以及状态，相等返回True，否则返回False
	# element_located_selection_state_to_be 传入定位元组以及状态，相等返回True，否则返回False
	# alert_is_present 是否出现Alert
	# 更多操作参考：http://selenium-python.readthedocs.io/api.html#module-selenium.webdriver.support.expected_conditions
 
**浏览器前进后退**     

	browser=webdriver.Chrome()
	browser.get('https://www.baidu.com/')
	browser.get('https://www.taobao.com/')
	browser.get('https://www.python.org/')
	browser.back()
	time.sleep(1)
	browser.forward()
	browser.close()

**Cookies**      

	browser=webdriver.Chrome()
	browser.get('https://www.zhihu.com')
	print(browser.get_cookies())
	browser.add_cookie({'name':'name','domain':'www.zhihu.com','value':'majortom'})
	print(browser.get_cookies())
	browser.delete_all_cookies()
	print(browser.get_cookies())

**浏览器多个选项卡管理**        
使用JS

	browser=webdriver.Chrome()
	browser.get('http://www.baidu.com')
	browser.execute_script('window.open()')
	print(browser.window_handles)
	browser.switch_to.window(browser.window_handles[1])
	browser.get('https://www.taobao.com')
	time.sleep(1)
	browser.switch_to.window(browser.window_handles[0])
	browser.get('https://python.org')

**异常处理**      
浏览器异常相对较复杂,种类比较多    

	browser=webdriver.Chrome()
	try:
    		browser.get('http://www.baidu.com')
	except TimeoutException:
    		print('Time Out Exception')
	try:
    		input=browser.find_element_by_id('hello')
	except NoSuchElementException:
    		print('No Such Element')
	# 官网的参考地址：http://selenium-python.readthedocs.io/api.html#module-selenium.common.exceptions


### 网络爬虫：###
请求网站并提取数据的自动化程序       

爬虫基本流程：     
**1.发起请求**      
Request      
请求：主要有GET,POST两种类型     
URL：统一资源定位符 （文档，图片，视频等由唯一URL确定）      
请求头：包含发出请求的配置信息   
请求体：请求时额外携带的数据 （GET形式不携带额外数据，POST形式有额外信息）       

**2.获取响应内容**     
Response      
响应状态：通过状态码来区分响应状态      
响应头：   包含响应的基本属性信息，字典形式结构，内容类型，内容长度等         
响应体： 包含响应资源的信息，也是最重要的信息             

**3.解析内容**       
直接处理     
Json解析    
正则表达式      
BeautifulSoup解析库       
PyQuery解析库    
XPath解析库     

**4.保存数据**      
爬虫可以抓取的信息：    
网页文本：HTML标签，文本       
图片：   图片的二进制文件格式      
视频： 视频的二进制文件格式      
其他格式：其他格式      

可以保存的数据：     
文本 ：  纯文本，Json 等      
关系型数据库：如MySQL、Oracle、SQL Server等结构化表结构形式存储         
非关系型数据库：如MongoDB、Redis等键值对形式存储      
二进制文件： 如图片、视频、音频等直接保存成特定格式      


