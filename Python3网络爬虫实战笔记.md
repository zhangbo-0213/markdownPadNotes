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

