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

### x名称空间 ###
x名称空间内的成员（x:Class、x:Name）是专门写给XAML编译器，引导XAML编译器将XAML代码编译成CLR代码的。  
包含XAML代码的WPF程序都需要通过语句： 

	xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"   
引入"http://schemas.microsoft.com/winfx/2006/xaml"名称空间。  
x名称空间内包含代码编写者能够与XAML编译器沟通的工具，这些工具告诉编译器重要信息，例如编译结果与哪个C#代码的编译结果合并、使用XAML声明的元素是public还是private访问级别等。包含的工具具体有：  

![](http://i.imgur.com/amqfD9K.png)   
![](http://i.imgur.com/4Bnqnbz.png)  

分为3部分：  
Attribute、标记扩展、XAML指令元素   

**Attribute**  

- **x:Class**   
告诉XAML编译器将XAML标签的编译结果与后台代码中指定的类合并。 
- **x:ClassModifier**  
告诉XAML编译器由标签编译生成的类具有怎样的访问控制级别。  
- **x:Name**   
XAML的标签声明的是对象，一个XAML标签会对应一个对象，这个对象一般为控件类的实例。x:Name的两个作用，一是告诉XAML编译器，当一个标签带有x:Name时除了为这个标签生成对应的实例外，还要为这个实例声明一个引用变量，变量名就是x:Name的值；二是将XAML标签对应的对象的Name属性设为x:Name，并将这个值注册到UI树上，方便查找。  
- **x:FieldModifier**  
用来在XAML中改变引用对象的访问级别   

		<StackPanel>
			<TextBox x:Name="textBox1" x:FieldModifier="public"/>
		</StackPanel>  
引用变量在XAML中默认访问级别为:internal。**注意**在使用x:FieldModifier的前提是，该标签同时使用了x:Name的特性。  
- **x:Key**   
为资源贴上用于检索的索引。WPF中，每个元素都有自己的Resources属性，属性是个"Key-Value"式的集合，为标签添加x:Key，将元素放进这个集合，就能够被检索。需要被重复使用的内容，都会放在资源里。使用实例：  

		<Window x:Class="About_X_Key.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:About_X_Key"
        mc:Ignorable="d"
        xmlns:sys="clr-namespace:System;assembly=mscorlib"
        Title="MainWindow" Height="350" Width="525">
    		<Window.Resources>
        		<sys:String x:Key="myString">Hello WPF Resources!
            	</sys:String>
    		</Window.Resources>
           <Grid>
        		<StackPanel>
            		<TextBox Text="{StaticResource ResourceKey=myString}" Margin="5"/>
            		<TextBox x:Name="textBox1" Margin="5"/>
            		<Button Content="Show" Click="ButtonBase_OnClick" Margin="5"/>
        	</StackPanel>
    	</Grid>
		</Window>      
这里将一个字符串对象添加x:Key作为重复使用的资源。在XAML中使用String类，使用  

		xmlns:sys="clr-namespace:System;assembly=mscorlib"   
引用mscorlib.dll,使用属性标签向Window.Resources添加字符串。代码效果：  
![](http://i.imgur.com/GQdXHqL.png)  
同时，该资源在后台代码中也能被使用，通过FindResources方法传入key找到对应资源：  

	 	private void ButtonBase_OnClick(object sender, RoutedEventArgs e)
        {
            string str = this.FindResource("myString") as string;
            this.textBox1.Text = str;
        }  
点击Button按钮效果：     
![](http://i.imgur.com/sZn80C6.png)  
- **x:Shared**   
与x:key配合使用，如果x:Shared的值为true,检索到该对象时得到的都是同一个对象（类似C#引用类型变量）,如果x:Shared的值为false，则每次检索得到的对象是新的副本（类似C#的数值类型变量），默认是true。  

**标记扩展**  

- **x:Type**   
当在XAML中想表达某个数据类型时，需要使用x:Type标记扩展。比如某一个类的属性，它的值要求是一种数据类型，当在XAML为这个属性赋值时需要使用x:Type。使用实例：  
**创建自定义窗口控价**  
	
		<UserControl x:Class="About_X_Type.MyWindow"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" 
             xmlns:d="http://schemas.microsoft.com/expression/blend/2008" 
             xmlns:local="clr-namespace:About_X_Type"
             mc:Ignorable="d" 
             d:DesignHeight="300" d:DesignWidth="300">
    	<Grid>
        <StackPanel Background="LightBlue">
            <TextBox Margin="5"/>
            <TextBox Margin="5"/>
            <TextBox Margin="5"/>
            <Button Content="OK" Margin="5"/>
        </StackPanel>
    	</Grid>
		</UserControl>  
这里根标签是UserControl类型，不是Window类型，控件效果：  
![](http://i.imgur.com/6leR2Q1.png)  
**创建Button派生类**  

	 	class MyButton:Button
    	{
        public Type UserWindowType { get; set; }

        protected override void OnClick()
        {
            base.OnClick();//激发Click事件
            UserControl win=Activator.CreateInstance(this.UserWindowType) as UserControl;
            //MessageBox.Show("done");
            if (win != null)
            {
                Window window=new Window();
                window.Width = 300;
                window.Height = 300;
                window.Content = win;
                window.ShowDialog();
            }
        }
    	}   
派生类中有一个Type类型的属性，UserWindowType，需要将一种类型赋值给该属性，重写OnClick方法，在调用父类Button的OnClick()方法外，创建出UserWindowType的实例，并将该实例加载到一个Window中，通过Window显示出该控件。  
**创建主窗口**   

		<Window x:Class="About_X_Type.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:About_X_Type"
        mc:Ignorable="d"
        Title="MainWindow" Height="350" Width="525">
    		<Grid>
        		<StackPanel>
            <local:MyButton Content="Show" UserWindowType="{x:Type TypeName=local:MyWindow}" Margin="5"/>
        		</StackPanel>
    		</Grid>
	</Window>  
这里MyWindow与MyButon都在这个名称空间下  

		xmlns:local="clr-namespace:About_X_Type"  
因此使用这个两个类型时，加 **local:** 主窗体上添加了自定义的MyButton,并把UserControl类型的自定义窗体赋值给MyButton的UserWindowType属性。运行效果：  
![](http://i.imgur.com/6SXDtbJ.png)   
点击"Show"   
![](http://i.imgur.com/v4Z3sTz.png)   

- **x:Null**    
显示地对具有默认值的属性而不需要此属性值设置null值。例如：

		<Button Content="OK" Style="{x:Null}"/>  

- **x:Array**  
x:Array的作用是向使用者暴露一个类型已知的ArrayList的实例，ArrayList内成员类型由x:Array的Type指明。WPF中将包含数据的对象称为数据源。例如：将x:Array当做数据源向一个ListBox提供数据 

		<Window x:Class="About_X_Array.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:About_X_Array"
        xmlns:sys="clr-namespace:System;assembly=mscorlib"
        mc:Ignorable="d"
        Title="MainWindow" Height="350" Width="525">
    	<Grid Background="LightBlue">
        <ListBox Margin="5">
            <ListBox.ItemsSource>
                <x:Array Type="sys:String">
                    <sys:String>Tim</sys:String>
                    <sys:String>Tom</sys:String>
                    <sys:String>Victor</sys:String>
                </x:Array>
            </ListBox.ItemsSource>
        </ListBox>
    	</Grid>
		</Window>  

运行效果：  
![](http://i.imgur.com/QJ1gGq3.png)   

- **x:Static**  
x:Static在XAML文档中使用数据类型的static成员。XAML不能编写逻辑代码，使用X:static访问的是数据类型的属性或字段。 
		
		<Window x:Class="About_X_Static.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:About_X_Static"
        mc:Ignorable="d"
        Title="{x:Static local:MainWindow.windowTitle}" Height="350" Width="525">
    	<Grid>
        <TextBlock FontSize="32" Text="{x:Static local:MainWindow.windowText}"/>
    	</Grid>
		</Window>
后台代码：  

		public partial class MainWindow : Window
    	{
        public static string windowTitle = "山高月小";

        public static string windowText
        {
            get { return "水落石出"; }
        }

        public MainWindow()
        {
            InitializeComponent();
        }
    	}
运行效果：   
![](http://i.imgur.com/UD5likY.png)  

**XAML指令元素**  
XAML指令元素只有两个：  
**x:Code**    
**x:XData**  

< x:Code >标签，将后台代码中的C#代码放置到XAML中，一般不这么做。   

x:Data是专用标签，用于数据提供。WPF中将包含数据的对象称为数据源，用于将数据源中的数据提供给数据使用者的对象称为数据提供者(Data Provider)。WPF类库中包含多种数据提供者，其中有一个XmlDataProvider，专门用于提供XML化的数据。示例： 

	Window x:Class="About_X_Static.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:About_X_Static"
        mc:Ignorable="d"
         Title="{x:Static local:MainWindow.windowTitle}" Height="350" Width="525">
    <Window.Resources>
        <XmlDataProvider x:Key="InventoryData" >
            <x:XData>
                <Supermarket xmlns="">
                    <Fruits>
                        <Fruit>Peach</Fruit>
                        <Fruit>Banana</Fruit>
                        <Fruit>Orange</Fruit>
                    </Fruits>
                    <Drinks>
                        <Drink >Tea</Drink>
                        <Drink >Coca Cola</Drink>
                        <Drink >PEPSI Cola</Drink>
                    </Drinks>
                </Supermarket>
            </x:XData> 
        </XmlDataProvider>
    </Window.Resources>
    <Grid>
        <TextBlock FontSize="32" Text="{x:Static local:MainWindow.windowText}" Height="80" Margin="5" TextAlignment="Center" VerticalAlignment="Top"/>
        <ListBox  ItemsSource="{Binding Source={StaticResource InventoryData},XPath=/Supermarket/Fruits/Fruit}" Width="240" Height="120" HorizontalAlignment="Left" Margin="5">
            </ListBox>
        <ListBox  ItemsSource="{Binding Source={StaticResource InventoryData},XPath=/Supermarket/Drinks/Drink}" Width="240" Height="120" HorizontalAlignment="Right" Margin="5">
        </ListBox>
    </Grid>
	</Window>
在 Window 的资源（<Window.Resources> ）中，有一个 XMLDataProvider 对象。x:Key 表示 Binding 对象中用来引用它的名称。这里，我们创建了 XML 内联数据（数据使用<x:XData>标签包含起来)。   
在ListBox的Binding中，使用"InventoryData"的StaticResource作为Source。如果某个数据源位于XAML文档中,需要指定该对象是一个静态源（StaticResource），使用Xpath指定XAML文档中的哪个集合填充ListBox。 运行结果：  
![](http://i.imgur.com/Cb6KsBK.png)

### 控件与布局 ###   
WPF中是数据驱动UI，数据是核心，是主动的；UI从属于数据并表达数据，是被动的。WPF中控件应该是抽象的数据和行为，而不是控件具体的形象。  
经常使用的6类控件：   

- **布局控件**   
可以容纳多个控件或嵌套其他布局控件。Grid、StackPanel、DockPanel等控件都属于此类，拥有共同的父类Panel。  

- **内容控件**  
只能容纳一个其他控件或布局控件作为它的内容。Window、Button等控件属于此类，由于只能容纳一个控件作为其内容，需要借助布局控件来规划其内容。  

- **带标题内容控件**   
相当于一个内容控件，但可以加一个标题（Header）,标题部分亦可以容纳一个控件或布局。如GroupBox、TabItem等 

- **条目控件**    
显示一列数据，数据类型一般都相同，如ListBox、ComboBox等。此类控件在显示集合类型的数据功能较强大。 

- **带标题条目控件**  
相当于一个条目控件加上一个标题显示区。如TreeViewItem、MenuItem等。这类控件用于显示层级关系数据，节点显示在其Header区域，子节点则显示在其条目控件区域。   

- **特殊内容控件**    
如TextBox容纳的是字符串、TextBlock可以容纳可自由控制格式的文本、Image容纳图片类型数据。  

6类控件的派生关系图：  
![](http://i.imgur.com/6Rxj6zl.png)  

FrameworkElement类在UIElement类的基础上添加了许多专门用于WPF开发的API，从FrameworkElement开始才算是进入WPF的开发框架中。   
WPF的UI元素根据是否可以装载内容、能够装载什么样的内容，可以分为如下类型：  

![](http://i.imgur.com/E9NsqiN.png)  

控件的内容可以直接是数据，也可以是控件。若控件的内容还是控件则为控件的嵌套，嵌套的控件称为子级控件。控件是内存中的对象，控件内容也是内存中的对象。控件通过自身某个属性引用作为其内容的对象，这个属性称为内容属性（Content Property）。不同的控件，内容属性的名称不同，有的叫"Content"有的叫"Child"等，如：  

		<Button Content"OK"/>  

Button控件的内容属性的名字为"Content",也可以写成  

		<Button>
			<Button.Content>
				<sys:String>OK</sys:String>
			</Button.Content>
		</Button>  
其内容标签的属性也可以省略：  

		<Button>	
			<sys:String>OK</sys:String>
		</Button>  

以StackPanel为例，其内容属性是Children,添加三个TextBox和一个Button，完整的语法：   

		<StackPanel Background="Gray">
			<StackPanel.Children>
				<TextBox Margin="5"/>
				<TextBox Margin="5"/>
				<TextBox Margin="5"/>
				<Button Content="OK" Margin="5"/>
			</StackPanel.Children>
		</StackPanel>  
简化后的代码：       
		
		<StackPanel Background="Gray">	      
				<TextBox Margin="5"/>
				<TextBox Margin="5"/>
				<TextBox Margin="5"/>
				<Button Content="OK" Margin="5"/>
		</StackPanel>  
