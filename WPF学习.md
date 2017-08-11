## WPF学习 ##
WPF(Windows Presentation Foundation)，是用来编写程序表示层的技术和工具。一般程序为多层架构，至少包含三层：  


- 数据层 
- 业务逻辑层 
- 表示层

WPF可以编写出独立于平台的应用程序，清晰定义设计和功能界面。       
WPF开发中设计与功能分离。  

WPF中，用户界面设计使用的语言是XAML，使用XML语法，允许以声明性的、带层次结构的方式把控件添加到用户界面上。 

WPF中一般使用XAML来布置用户界面上的控件。XAML是WPF技术中用于设计UI的语言。  

WPF项目基本组成：  

- **Properties**  
程序需要用到的一些资源（图标，图片，静态字符串）和配置信息。    

- **References**   
当前项目需要引用的项目。   

- **App.xaml**   
程序的主体。Windows系统中，一个程序是一个进程，一个GUI进程需要有一个窗体（Windows）作为“主窗体”。App.xaml文件声明程序的进程，指定程序的主窗体。该分支下的App.xaml.cs是App.xaml的后台代码。 

- **MainWindow.xaml**   
程序的主窗体。对于XAML文件，VS具有可视化编辑能力，其分支下的MainWindows.xaml.cs是其后台代码。   

###XAML###
XAML是一种由XML派生而来，许多XML中的概念在XAML是通用的:  

- 使用标签声明一个元素。通过起始标签和终止标签< Tag >  < /Tag >  
- 通过赋予标签特征值（Attribute）表示某个标签的不同。   
  非空标签：< Tag Attribute1=Value1 Attribute2=Value2 > Content < /Tag >   
  空标签： < Tag Attribute1=Value1 Attribute2=Value2 / > 
 
 Attribute（特征）属于编程语言文法层面的内容；  
 Property （属性）属于面向对象编程中抽象对象身上的性状；
 XAML是用来在UI上绘制控件的，控件本身是一个对象，因此标签中有一部分Attribute会和控件对象的Proerty相对应。当然XAML标签中的Attribute并不对应控件对象的Property。 

**MainWindow.xaml中的组成部分：**   

	<Window x:Class="P20_AboutXAML.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:P20_AboutXAML"
        mc:Ignorable="d"
        Title="MainWindow" Height="350" Width="525">
    <Grid>
        
    </Grid>
	</Window>   

主体结构：  

	<Window>
		<Grid>
		</Grid>
	</Window>
窗体对象内嵌套一个Grid对象。  
Attribute部分：  
XML文档标签使用xmlns特征定义命名空间（xml-namespace）,xmlns特征语法格式：  

	xmlns:可选的映射前缀="命名空间"  
如果没有写可选映射前缀，那么所有来自这个命名空间的标签不用加对应的命名前缀，例如，在主体结构中Window和Grid标签前没有加前缀，则说明来自默认的命名空间。  
	
	 xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"   

默认命名空间只能有一个，应选择被元素频繁使用的命名空间充当默认命名空间。  

第一行中：  

	x:Class="P20_AboutXAML.MainWindow"  
前的x:前缀，说明这个Class引用了带有:x映射标记的命名空间，即：  

	xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"	

XAML中引用外来程序集和其中.NET命名空间步骤：  
1. 在References部分添加对应程序集的引用  
2. 在XAML的根元素的起始标签将对应的程序集以  
   
	xmlns:可选的映射前缀="命名空间"    
形式添加到标签特征列表中。  

< Windows > < /Windows >标签中通过xmlns引用的命名空间类似一个网址地址，这里是XAML解析器的一个硬性编码，只要见到这些固定字符串，会把相应一系列的程序集以及程序集中包含的.NET名称空间引用进来。  

x:Class 这个Attribute的作用是当XAML解析器将包含它的标签解析成C#类后，这个类的类名是啥。上面的 < Window >标签被解析成C#的类后，这个类的类名即为 "P20_AboutXAML.MainWindow"  

在MainWindow.xaml中包含有MainWindow类的声明，在MainWindow.xaml.cs中同样有MainWindow类的声明。这两部分不会冲突，原因在于MainWindow.xaml.cs中MainWindow类的声明部分使用了partial关键字，显然，由XAML解析器在生成MainWindow类时也会使用partical关键字，这样两部分的类构成一个完整部分。也正是partial这种机制，使得后台代码中负责逻辑，XAML部分负责UI设计，使设计与分离分开。 

### XAML语法 ###
XAML使用标签来定义UI元素，每个标签对应.NET Framework类库中的一个控件类。通过给标签Attribute，可以对标签所对应的控件对象Property进行赋值，以及额外设置（声明命名空间，指定类名）。  

XAML使用树形逻辑描述UI。如图所示UI：  

![](http://i.imgur.com/uX5oVGW.png)  

对应的XAML语言描述：   

	<Window x:Class="P20_AboutXAML.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:P20_AboutXAML"
        mc:Ignorable="d"
        Title="MainWindow" Height="350" Width="525">
    <Grid>
        <StackPanel Background="LightBlue" Margin="0,0,2,0">
            <TextBox x:Name="textBox1" Margin="5"  Height="50"/>
            <TextBox x:Name="textBox2" Margin="5" Height="50"/>
            <StackPanel Orientation="Horizontal">
                <TextBox x:Name="textBox3" Margin="5" Height="50" Width="248"/>
                <TextBox x:Name="textBox4" Margin="5" Height="50" Width="248"/>
            </StackPanel>
            <Button x:Name="button1" Margin="5" Height="50">
            </Button>
        </StackPanel>
    </Grid>
	</Window>   
主体部分：  

	<Window>
		<Grid>
			<StackPanel>
				<TextBox>
				</TextBox>
				<TextBox>
				</TextBox>
				<StackPanel>
					<TextBox>
					</TextBox>
					<TextBox>
					</TextBox>
				</StackPanel>
				<Button>
				</Button>
			</StackPanel>
		</Grid>
	</Window>

**XAML中为对象属性赋值**  
XAML中为对象属性赋值有两种语法：  

- 使用字符串进行简单赋值   
- 使用属性元素进行复杂赋值   

上面代码中，为Window标签里的部分属性使用字符串赋值：   

	<Window Title="MainWindow" Height="350" Width="525">
	</Window>  

属性元素   
XAML中，非空标签具有自己的内容（Content）,即夹在起始标签和结束标签之间的一些子级标签。每个子级标签都是父级标签的一个元素。属性元素指的是，标签的一个元素对应这个标签的一个属性，以元素的形式表达一个属性的实例。使用属性元素来赋值的代码描述：

	<ClassName>
		<ClassName.PropertyName>
		<!--以对象形式为属性赋值-->
		</ClassName.PropertyName>
	</ClassName>  

属性元素绘制径向渐变效果：  

![](http://i.imgur.com/DmbvuVv.png)  

对应XAML代码：  

	<Grid VerticalAlignment="Center" HorizontalAlignment="Center">
        <Ellipse Width="200" Height="200">
            <Ellipse.Fill>
                <RadialGradientBrush GradientOrigin="0.25,0.25" RadiusX="0.75" RadiusY="0.75">
                    <RadialGradientBrush.GradientStops>
                        <GradientStop Color="White" Offset="0"/>
                        <GradientStop Color="Black" Offset="0.65"/>
                        <GradientStop Color="Gray" Offset="0.8"/>
                    </RadialGradientBrush.GradientStops>
                </RadialGradientBrush>
            </Ellipse.Fill>
            </Ellipse>
    </Grid> 

**事件处理器与代码后置**   
当XAML标签对应一个对象时，标签的一部分Attribute会对应这个对象的Property。还有一部分Attribute会对应对象的事件。例如 < Button >标签有一个名为Click的Attribute，对应的是Button类的Click事件。
.Net事件的处理机制中，可以为对象的某个事件指定一个能与该事件匹配的成员函数，当这个事件发生时，.Net运行时调用这个函数。这个函数称为“事件处理器”(EventHandler)。  
XAML中为对象的事件指定事件处理器，使用事件处理器的函数名为对应对象事件的Attribute赋值：  

	<ClassName.EventName="EventHandlerName"/>  

EventHandlerName的事件处理器的具体逻辑会写在MainWindow.xaml.cs的后台代码中，使业务逻辑和UI设计分离，通过partial的类关键字最后编译合并。