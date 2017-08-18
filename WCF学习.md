## WCF学习 ##
WCF：Windows Communication Foundation  
视窗操作系统通信框架，.net3.5后支持，.net中提供的统一的通信编程模型，通过配置文件对通信方式进行修改。    
WCF是微软实现SOA（面向服务架构）的解决方案，服务就是提供给别人使用的应用程序。比如说网站要显示天气预报，只要知道天气预报网站给我的接口就可以了，具体实现细节不用管，这就是SOA的松耦合性。

----------
WCF程序的基本组成包括：  

- **契约**（服务端和客户端之间的通信约定）   
- **服务端**（实现契约接口和提供服务）   
- **客户端**（使用，消费服务端提供的服务）  
- **配置文件**（对通信方式进行配置）  

作为客户端也就是服务的消费者，必须要知道服务部署在哪，要知道如何通信，也要知道双方的数据传输规则，在WCF中这三样内容被整体称为“终结点”，上述三个方面被称为“地址、绑定、契约”（也就是“ABC”），也就是说终结点是服务的“出入口”。


----------

**1.入门小实例**  

- **契约部分**   
       
新建解决方案，添加第一个项目作为契约部分，添加类库ICommunicationContract.cs,添加System.ServiceModel的引用   

    namespace WCF_Interface
	{
	[ServiceContract]
    public interface ICommunicationContract
    {
        [OperationContract]
        int Add(int a, int b);
    }	
	}
   
契约部分的接口和接口成员需要添加ServiceContract和OperationContract的特性，这样在进行配置的时候，客户端才能找到对应的服务关联    

- **服务端部分**   

添加新控制台项目作为服务端程序，添加一个类用来**实现契约的接口**，服务端项目需要添加对契约项目的引用   

	 namespace WCF_Service
	{
    [ServiceBehavior]
    public class CommunicationService:ICommunicationContract
    {
        [OperationBehavior]
        public int Add(int a, int b)
        {
            return a + b;
        }
    }
	}   
服务端实现契约接口部分需要添加ServiceBehaviour和OperationBehaviour的特性，与契约部分对应  
**服务端配置文件**    
通过右键 **编辑WCF配置** 对App.config进行向导式配置，添加服务和绑定，在对App.config进行配置之前，需要对服务端程序进行编译得到对应的服务类型，通过向导式配置方式得到的配置文件：   
   
	 <?xml version="1.0" encoding="utf-8" ?>
	<configuration>
    <startup> 
        <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.5.2" />
    </startup>
    <system.serviceModel>
        <bindings>
            <netTcpBinding>
                <binding name="NewBinding0">
                    <security mode="None" />
                </binding>
            </netTcpBinding>
        </bindings>
        <services>
            <service name="WCF_Service.CommunicationService">
                <endpoint address="net.tcp://localhost:8080/wcf" binding="netTcpBinding"
                    bindingConfiguration="" contract="WCF_Interface.ICommunicationContract" />
            </service>
        </services>
    </system.serviceModel>
	</configuration>     
**服务端启动程序**      
服务端启动程序为控制台应用程序    

	namespace WCF_Service
	{
    class Program
    {
        static void Main(string[] args)
        {
            ServiceHost host=new ServiceHost(typeof(WCF_Service.CommunicationService));
            host.Open();
            Console.WriteLine("服务启动！");
            Console.ReadLine();
        }
    }	
	}

- **客户端部分**  

新建控制台应用程序作为客户端使用服务端提供的程序，**添加对契约项目的引用**  ，添加System.ServiceModel引用   

	namespace WCF_Client
	{
    class Program
    {
        static void Main(string[] args)
        {
            var channel=new ChannelFactory<ICommunicationContract>("clientEndpoint");
            var client = channel.CreateChannel();
            Console.WriteLine(client.Add(23,20));
            Console.ReadLine();
        }
    }
	}
需要注意的是，这里ChannelFactory< >("clientEndpoint")传入的参数一般和配置文件的部分属性有关，如绑定的节点名等   
**客户端配置文件**    
通过右键 编辑WCF配置 对App.config进行向导式配置，添加客户端和客户端终结点，添加绑定时制定一个绑定节点名，添加客户端的地址应与服务端添加服务时的地址一致，通过向导式配置得到的客户端配置文件：    

	<?xml version="1.0" encoding="utf-8" ?>	
	<configuration>
    <startup> 
        <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.5.2" />
    </startup>
    <system.serviceModel>
        <bindings>
            <netTcpBinding>
                <binding name="NewBinding0">
                    <security mode="None" />
                </binding>
            </netTcpBinding>
        </bindings>
        <client>
            <endpoint address="net.tcp://localhost:8080/wcf" binding="netTcpBinding"
                bindingConfiguration="" contract="WCF_Interface.ICommunicationContract"
                name="clientEndpoint" kind="" endpointConfiguration="" />
        </client>
    </system.serviceModel>
	</configuration>   

**运行测试**   
运行时先启动服务端程序，启动客户端程序后，客户端调用服务端实现的契约接口方法（client.Add(23,20)），并输出调用结果（43）

**使用多个Binding建立通信**    
入门实例中的Binding使用的是net.tcpBinding，除了这种常用的Binding类型，httpBinding也是较为常用的类型，在建立的一个契约下，可以创建多个终结点，使用不同类型的Binding进行服务的传递，服务器端和客户端对应添加终结点，需要指定对应的Address,Binding和Contract，**值得注意的是**，当使用httpBinding类型节点时，调试程序可能会出现"http无法注册...该进程不具备此进程命名空间的访问权限",此问题是因为在Win7及以后系统中运行注册URL的程序需要具有管理员特权。**解决方法：**使用管理员权限运行生成的EXE文件
使用管理员权限运行VS，则宿主主机也会使用管理员权限启动  


----------

**2.使用代码实现配置过程**   
入门实例中通过向导式的配置设置配置文件，这是微软推荐的方式，除了使用这种方式完成配置文件的设置，还可以通过纯代码的方式实现配置过程。  

- **服务端程序**  
新建控制台程序，删除App.config配置文件，这里将契约接口部分，实现部分，和服务器端的程序写在一个项目里  
**契约部分**   
		
		namespace WCF_Demo3_Service
		{
    	[ServiceContract]
    	public interface ICommunicationContract
    	{
        [OperationContract]
        string SayHello();
    	}
		}  
**实现部分**   
		
		namespace WCF_Demo3_Service
		{
    	[ServiceBehavior]
    	public class CommunicationContract:ICommunicationContract
    	{
        [OperationBehavior]
        public string SayHello()
        {
            return "这里是服务器端提供的程序";
        }
    	}
		}  
契约部分和实现部分与之前的例子中没有什么区别，区别在于接下来服务端程序中通过代码实现配置过程     
		
		namespace WCF_Demo3_Service
		{
    	class Program
    	{
        static void Main(string[] args)
        {
            //定义两个基地址，一个用于http,一个用于tcp
            Uri httpAddress=new Uri("http://localhost:8080/wcf");
            Uri tcpAddress=new Uri("net.tcp://localhost:8081/wcf");
            //服务类型，实现接口的类
            Type serviceType = typeof (CommunicationContract);

            //定义一个ServiceHost
            using (ServiceHost host = new ServiceHost(serviceType, new Uri[] { httpAddress, tcpAddress }))
            {
                //定义一个basicHttpBinding 
                Binding basicHttpBinding=new BasicHttpBinding();
                string address = "";
                //创建endPoint，使用Binding和address作为参数
                host.AddServiceEndpoint(typeof (WCF_Demo3_Service.ICommunicationContract), basicHttpBinding, address);

                //定义一个netTcpBinding
                Binding tcpBinding=new NetTcpBinding();
                address = "";
                host.AddServiceEndpoint(typeof (WCF_Demo3_Service.ICommunicationContract), tcpBinding, address);

                //启动服务
                host.Open();
                Console.WriteLine("WCF_Demo3 服务已开启");
                Console.ReadKey();
                host.Close();
            }
        }
    	}
		}

代码中添加的内容可以与之前App.config文件中的配置内容进行对应     
ServiceHost  
  CommunicationContract　　 　实现类的类型  
  Uri[]                      基地址，对应config中的 baseAddresses  
  ServiceEndpoint[]          服务终结点，对应config中的多个endpoint  
      
- ServiceContract    服务契约，对应config中<endpoint>的contract属性  

- Binding            绑定，对应config中<endpoint>的binding属性   

- EndpointAddress    终结点地址，对应config中<endpoint>的address属性   

这样服务器端的终结点的配置过程就通过代码完成了  

- **客户端程序**  
客户端同样需要通过代码来完成终结点的配置，同时创建契约的代理来进行服务的调用  ，删除新建的客户端项目的App.config文件 

		namespace WCF_Demo3_client
		{
    	class Program
    		{
        static void Main(string[] args)
        {
            //定义Binding与服务地址
            Binding httpBinding=new BasicHttpBinding();
            EndpointAddress httpAddress=new EndpointAddress("http://localhost:8080/wcf");
            //使用ChannelFactory创建一个IData的代理对象,指定Binding和Address
            var client=new ChannelFactory<ICommunicationContract>(httpBinding,httpAddress).CreateChannel();
            Console.WriteLine("httpBinding: "+client.SayHello());
            ((IChannel) client).Close();

            //换成Tcp的Binding和服务地址
            Binding tcpBinding=new NetTcpBinding();
            EndpointAddress tcpAddress=new EndpointAddress("net.tcp://localhost:8081/wcf");
            //使用ChannelFactory创建一个IData的代理对象,指定Binding和Address
            var client2 = new ChannelFactory<ICommunicationContract>(tcpBinding, tcpAddress).CreateChannel();
            Console.WriteLine("tcpBinding: "+client2.SayHello());
            ((IChannel)client2).Close();

            Console.ReadKey();
        }
    	}
		}  
测试过程中，同样需要使用管理员权限运行生成的EXE文件，客户端能够通过netTcpbinding和basicHttpBinding对服务进行调用

----------

**3.通过添加服务引用的方式使客户端摆脱对契约接口的dll引用**   
在实例1中，通过在客户端对契约项目的引用，创建了契约代理的对象来调用服务端提供的服务，即     

		var channel=new ChannelFactory<ICommunicationContract>("clientEndpoint");
        var client = channel.CreateChannel();
这里在生成契约代理对象时，需要指定契约的接口类型，也就是需要应用契约接口的.dll文件，可以通过添加服务引用的方式来摆脱对该.dll文件的引用，具体实例：  

- **契约及实现部分**    
	
		namespace Contracts
		{
    	[ServiceContract(Name = "CalculatrService",Namespace = "http://www.artech.com/")]
    	public interface ICalculator
    	{
        [OperationContract]
        double Add(double x,double y);

        [OperationContract]
        double Subtract(double x, double y);

        [OperationContract]
        double Multiply(double x,double y);

        [OperationContract]
        double Divide(double x,double y);
    	}
		}
	
	契约部分在特性中添加了Name属性，在客户端可以通过该属性名找到对应的服务，契约部分需要添加System.ServiceModel引用  
		
		namespace Services
		{
    	public class CalculatorService:ICalculator
    	{
        public double Add(double x, double y)
        {
            return x + y;
        }

        public double Subtract(double x, double y)
        {
            return x - y;
        }

        public double Multiply(double x, double y)
        {
            return x*y;
        }

        public double Divide(double x, double y)
        {
            return x/y;
        }
    	}
		}    
	
	契约实现部分
   
- **服务端部分**    

		namespace Hosting
		{
    	class Program
    	{
        static void Main(string[] args)
        {
            using (ServiceHost host = new ServiceHost(typeof (CalculatorService)))
            {
                //通过代码的方式添加终结点，指定ABC（Address,Binding,Contract）
                host.AddServiceEndpoint(typeof (ICalculator), new WSHttpBinding(), "http://127.0.0.1:9999/calculatorservice");
                if (host.Description.Behaviors.Find<ServiceMetadataBehavior>() == null)
                {
                    //WCF服务的描述通过元数据（Metadata）形式发布；
                    //WCF中元数据的发布通过特殊的服务行为ServiceMetadataBehavior来实现；
                    //为创建的host添加ServiceMetadataBehavior，采用基于HTTP-GET的元数据获取方式，发布地址由ServiceMetadataBehavior的HttpGetUri指定；
                    //服务开启后，通过http://127.0.0.1:9999/calculatorservice/metadata 得到以WSDL形式体现的服务元数据
                    ServiceMetadataBehavior behavior =new ServiceMetadataBehavior();
                    behavior.HttpGetEnabled = true;
                    behavior.HttpGetUrl=new Uri("http://127.0.0.1:9999/calculatorservice/metadata");
                    host.Description.Behaviors.Add(behavior);
                }
                host.Opened += delegate { Console.WriteLine("CalculatorService 已经启动，请按任意键终止"); };
                host.Open();
                Console.ReadKey();
            }
        }
    	}
		}  
	服务端部分使用代码的方式添加终结点，并将元数据进行发布，这样客户端在添加服务引用时指定引用的地址，即可找到对应的服务，服务端需要引用System.ServiceModel及契约和实现部分的项目引用   


- **客户端部分**  

		namespace Client
		{
    	class Program
    	{
        static void Main(string[] args)
        {
            CalculatrServiceClient client=new CalculatrServiceClient();
            Console.WriteLine("x+y={2},when x={0},y={1}",1,2,client.Add(1,2));
            Console.WriteLine("x-y={2},when x={0},y={1}", 1, 2, client.Subtract(1, 2));
            Console.WriteLine("x*y={2},when x={0},y={1}", 1, 2, client.Multiply(1, 2));
            Console.WriteLine("x/y={2},when x={0},y={1}", 1, 2, client.Divide(1, 2));
            Console.ReadKey();
        }
    	}
		}
		
	客户端中CalculatorServiceClient类是在添加服务引用时，WCF根据服务地址自动生成的类，在客户端的Service References文件夹的References.cs中，这个类中包含了契约实现部分的方法，相当于是服务端的一个等效的契约接口，这样客户端只需要引用System.ServiceModel，从而摆脱对服务端契约项目的引用   

----------

**4.终结点地址与WCF寻址**  
WCF服务端与客户端之间的通信都是基于终结点进行，服务提供者将终结点（一个或多个）暴露给消费者，通过HTTP-GET或MEX终结点的方式将元数据(Metadata)以WSDL（网络服务描述语言）的方式对外发布，消费者通过访问WSDL，导入元数据生成服务代理相关的代码和配置。借助于自动生成的服务代理相关的代码和配置，创建相匹配的终结点对服务进行访问。  

**为服务指定地址**    

1. **通过代码方式指定地址**     
使用自我寄宿的方式对某个服务进行寄宿的时候，可以通过ServiceHostBase或其子类ServiceHost的AddServiceEndpoint方法为添加的终结点指定相应的地址，代码如下：	

		using (ServiceHost host = new ServiceHost(typeof (CalculatorService)))
    	{
     	//通过代码的方式添加终结点，指定ABC（Address,Binding,Contract）
     	host.AddServiceEndpoint(typeof (ICalculator), new WSHttpBinding(), "http://127.0.0.1:9999/calculatorservice");  
		 host.Open();
	 	//Other Code
    	}   

2. **通过配置指定地址**     
WCF中可以通过配置的方式添加终结点，在对应服务的配置节点下,可以添加相关的终结点列表。      

		<system.serviceModel>
        <bindings>
            <netTcpBinding>
                <binding name="NewBinding0">
                    <security mode="None" />
                </binding>
            </netTcpBinding>
        </bindings>
        <services>
            <service name="WCF_Service.CommunicationService">
                <endpoint address="net.tcp://localhost:8080/wcf" binding="netTcpBinding"
                    bindingConfiguration="" contract="WCF_Interface.ICommunicationContract" />
            </service>
        </services>
    	</system.serviceModel>

3. **IIS寄宿下对地址的指定**   
与自我寄宿不同，IIS寄宿的方式需要为服务创建一个.svc文件，并将该文件部署到一个确定的IIS虚拟目录下，服务的消费者通过访问.svc文件进行服务的调用，所以svc文件的地址就是服务地址，无需再通过配置指定终结点的地址。  


----------
**基地址与相对地址**  
某个服务的终结点地址除了可以以绝对地址的方式指定，还可以采取“基地址+相对地址”的方式进行设置。对于一个服务来说，可以指定一个或多个基地址，但对于一个具体的传输协议类型，只能有一个唯一的基地址。服务的基地址和终结点的相对地址可以通过代码的方式，在创建ServiceHost对象时在构造函数中指定：  

	public class ServiceHost : ServiceHostBase
	{	
	public ServiceHost(Type serviceType, params Uri[] baseAddresses);
	public ServiceHost(object singletonInstance, params Uri[] baseAddresses);
	}  

在下面的代码中，在自我寄宿服务时，添加了两个基地址，一个是基于HTTP的，另一个是基于net.tcp的。添加了两个终结点，一个采用Http的BasicHttpBinding,另一个采用基于TCP的NetTcpBinding  

	static void Main(string[] args)
        {
            //定义两个基地址，一个用于http,一个用于tcp
            Uri httpAddress=new Uri("http://localhost:8080/wcf");
            Uri tcpAddress=new Uri("net.tcp://localhost:8081/wcf");
            //服务类型，实现接口的类
            Type serviceType = typeof (CommunicationContract);

            //定义一个ServiceHost
            using (ServiceHost host = new ServiceHost(serviceType, new Uri[] { httpAddress, tcpAddress }))
            {
                //定义一个basicHttpBinding 
                Binding basicHttpBinding=new BasicHttpBinding();
                string address = "";
                //创建endPoint，使用Binding和address作为参数
                host.AddServiceEndpoint(typeof (WCF_Demo3_Service.ICommunicationContract), basicHttpBinding, address);

                //定义一个netTcpBinding
                Binding tcpBinding=new NetTcpBinding();
                address = "";
                host.AddServiceEndpoint(typeof (WCF_Demo3_Service.ICommunicationContract), tcpBinding, address);

                //启动服务
                host.Open();
                Console.WriteLine("WCF_Demo3 服务已开启");
                Console.ReadKey();
                host.Close();
            }
        }  
由于AddServiceEndpoint指定的是相对地址，WCF系统会根据Binding采用的传输协议在ServiceHost的基地址列表中寻找与之相配的基地址，相对地址和基地址组合确定为终结点的绝对地址。**值得注意的是**对于一个确定的传输协议，最多只能有一个基地址。服务的基地址和相对地址同样可以通过配置的方式进行设定。  


----------

**地址的终结点共享**   
一个终结点由ABC三个要素构成（地址，绑定，契约），对于同一个服务的若干终结点，服务一般只实现唯一一个契约，所有的终结点共享相同的服务契约，这种情况下，终结点的地址不能共享，对应地址必须是不同的。  
如果一个服务实现了多个服务契约，基于多个服务契约在添加终结点时，这些针对不同服务契约的终结点可以共享相同的地址和绑定。  


----------
**在客户端指定地址**    
客户端调用服务时，通过两种调用方式，一种是通过代码生成工具或者添加服务引用导入元数据生成服务代理类型，另一种通过ChannelFactory<T>或DuplexChannelFactory<T>来创建服务代理对象。  
 
- **为ClientBase< TChannel >指定地址**   
 
通过代码生成器或添加服务引用的方式，生成的核心类是继承自System.ServiceModel.ClinetBase< TChannel >的子类，TChannel为服务契约类型，对于如下这个契约：   

	 namespace Contracts
	{
    [ServiceContract(Name = "CalculatrService",Namespace = "http://www.artech.com/")]
    public interface ICalculator
    {
        [OperationContract]
        double Add(double x,double y);

        [OperationContract]
        double Subtract(double x, double y);

        [OperationContract]
        double Multiply(double x,double y);

        [OperationContract]
        double Divide(double x,double y);
    }
	}  
通过代码生成器和添加服务引用的方式，会生成3个类，CalculatrService,CalculatrServiceChannel,CalculatrServiceClient(这里的Calculatr实际上对应的就是服务契约ICalculator，由于在服务契约ICalculator中添加ServieContract特性时，添加了Name属性，所以客户端生成的对应类使用了Name这个属性来表示ICalculator服务契约)   

	namespace Client.CalculatorService { 
    [System.CodeDom.Compiler.GeneratedCodeAttribute("System.ServiceModel", "4.0.0.0")]
    [System.ServiceModel.ServiceContractAttribute(Namespace="http://www.artech.com/", ConfigurationName="CalculatorService.CalculatrService")]  
  
	//CalculatrService
    public interface CalculatrService {
        
        [System.ServiceModel.OperationContractAttribute(Action="http://www.artech.com/CalculatrService/Add", ReplyAction="http://www.artech.com/CalculatrService/AddResponse")]
        double Add(double x, double y);
        
        [System.ServiceModel.OperationContractAttribute(Action="http://www.artech.com/CalculatrService/Add", ReplyAction="http://www.artech.com/CalculatrService/AddResponse")]
        System.Threading.Tasks.Task<double> AddAsync(double x, double y);
        
        [System.ServiceModel.OperationContractAttribute(Action="http://www.artech.com/CalculatrService/Subtract", ReplyAction="http://www.artech.com/CalculatrService/SubtractResponse")]
        double Subtract(double x, double y);
        
        [System.ServiceModel.OperationContractAttribute(Action="http://www.artech.com/CalculatrService/Subtract", ReplyAction="http://www.artech.com/CalculatrService/SubtractResponse")]
        System.Threading.Tasks.Task<double> SubtractAsync(double x, double y);
        
        [System.ServiceModel.OperationContractAttribute(Action="http://www.artech.com/CalculatrService/Multiply", ReplyAction="http://www.artech.com/CalculatrService/MultiplyResponse")]
        double Multiply(double x, double y);
        
        [System.ServiceModel.OperationContractAttribute(Action="http://www.artech.com/CalculatrService/Multiply", ReplyAction="http://www.artech.com/CalculatrService/MultiplyResponse")]
        System.Threading.Tasks.Task<double> MultiplyAsync(double x, double y);
        
        [System.ServiceModel.OperationContractAttribute(Action="http://www.artech.com/CalculatrService/Divide", ReplyAction="http://www.artech.com/CalculatrService/DivideResponse")]
        double Divide(double x, double y);
        
        [System.ServiceModel.OperationContractAttribute(Action="http://www.artech.com/CalculatrService/Divide", ReplyAction="http://www.artech.com/CalculatrService/DivideResponse")]
        System.Threading.Tasks.Task<double> DivideAsync(double x, double y);
    }
    

    [System.CodeDom.Compiler.GeneratedCodeAttribute("System.ServiceModel", "4.0.0.0")]

	//CalculatrServiceChannel
    public interface CalculatrServiceChannel : Client.CalculatorService.CalculatrService, System.ServiceModel.IClientChannel {
    }
    
    [System.Diagnostics.DebuggerStepThroughAttribute()]
    [System.CodeDom.Compiler.GeneratedCodeAttribute("System.ServiceModel", "4.0.0.0")]

	//CalculatrServiceClient
    public partial class CalculatrServiceClient : System.ServiceModel.ClientBase<Client.CalculatorService.CalculatrService>, Client.CalculatorService.CalculatrService {
        
        public CalculatrServiceClient() {
        }
        
        public CalculatrServiceClient(string endpointConfigurationName) : 
                base(endpointConfigurationName) {
        }
        
        public CalculatrServiceClient(string endpointConfigurationName, string remoteAddress) : 
                base(endpointConfigurationName, remoteAddress) {
        }
        
        public CalculatrServiceClient(string endpointConfigurationName, System.ServiceModel.EndpointAddress remoteAddress) : 
                base(endpointConfigurationName, remoteAddress) {
        }
        
        public CalculatrServiceClient(System.ServiceModel.Channels.Binding binding, System.ServiceModel.EndpointAddress remoteAddress) : 
                base(binding, remoteAddress) {
        }
        
        public double Add(double x, double y) {
            return base.Channel.Add(x, y);
        }
        
        public System.Threading.Tasks.Task<double> AddAsync(double x, double y) {
            return base.Channel.AddAsync(x, y);
        }
        
        public double Subtract(double x, double y) {
            return base.Channel.Subtract(x, y);
        }
        
        public System.Threading.Tasks.Task<double> SubtractAsync(double x, double y) {
            return base.Channel.SubtractAsync(x, y);
        }
        
        public double Multiply(double x, double y) {
            return base.Channel.Multiply(x, y);
        }
        
        public System.Threading.Tasks.Task<double> MultiplyAsync(double x, double y) {
            return base.Channel.MultiplyAsync(x, y);
        }
        
        public double Divide(double x, double y) {
            return base.Channel.Divide(x, y);
        }
        
        public System.Threading.Tasks.Task<double> DivideAsync(double x, double y) {
            return base.Channel.DivideAsync(x, y);
        }
    }
	}
CalculatrService是服务端契约ICalculator的客户端的表示，CalculatrServiceChannel是继承System.ServiceModel.IClientChannel，后端定义了客户端信道的基本行为，CalculatrServiceClient最终用于服务访问的服务代理类，继承泛型基类ClientBase< Client.CalculatorService.CalculatrService > ,
实现了服务契约，客户端对服务的访问，通过该类的实例进行。继承自ClientBase< Client.CalculatorService.CalculatrService >的CalculatrServiceClient，地址指定是通过其构造函数进行。   

	 public CalculatrServiceClient(string endpointConfigurationName) : 
                base(endpointConfigurationName) {
        }
        
        public CalculatrServiceClient(string endpointConfigurationName, string remoteAddress) : 
                base(endpointConfigurationName, remoteAddress) {
        }
        
        public CalculatrServiceClient(string endpointConfigurationName, System.ServiceModel.EndpointAddress remoteAddress) : 
                base(endpointConfigurationName, remoteAddress) {
        }
        
        public CalculatrServiceClient(System.ServiceModel.Channels.Binding binding, System.ServiceModel.EndpointAddress remoteAddress) : 
                base(binding, remoteAddress) {
        }   
ClientBase< TChannel >的部分定义：   
	
	public abstract class ClientBase<TChannel>
	{
	protected ClientBase();
	protected ClientBase(string endpointConfigurationName);
	protected ClientBase(ServiceEndpoint endpoint);
	protected ClientBase(InstanceContext callbackInstance);  
	protected ClientBase(string endpointConfigurationName, string remoteAddress);
	protected ClientBase(string endpointConfigurationName, EndpointAddress remoteAddress);   
	protected ClientBase(Binding binding, EndpointAddress remoteAddress);  
	protected ClientBase(InstanceContext callbackInstance, string endpointConfigurationName);  
	protected ClientBase(InstanceContext callbackInstance, ServiceEndpoint endpoint);  
	protected ClientBase(InstanceContext callbackInstance, Binding binding, EndpointAddress remoteAddress);
	protected ClientBase(InstanceContext callbackInstance, string endpointConfigurationName, EndpointAddress remoteAddress);
	protected ClientBase(InstanceContext callbackInstance, string endpointConfigurationName, string remoteAddress);
	}  
Client< TChannel >是客户端进行服务调用的服务代理基类。ClientBase< TChannel >提供一系列重载构造函数，去指定终结点的3要素（地址，绑定，契约），对于双工通信（Duplex）的情况，通过InstanceContent回调实例实现从服务端对客户端操作的回调。  
当没有显式地在构造函数中指定终结点要素的情况，默认从配置中读取。例如：   

	class Program
    {
        static void Main(string[] args)
        {
            CalculatrServiceClient client=new CalculatrServiceClient("WSHttpBinding_CalculatrService");
            Console.WriteLine("x+y={2},when x={0},y={1}",1,2,client.Add(1,2));
            Console.WriteLine("x-y={2},when x={0},y={1}", 1, 2, client.Subtract(1, 2));
            Console.WriteLine("x*y={2},when x={0},y={1}", 1, 2, client.Multiply(1, 2));
            Console.WriteLine("x/y={2},when x={0},y={1}", 1, 2, client.Divide(1, 2));
            Console.ReadKey();
        }
    }
客户端在使用CalculatrServiceClient类实例化时，并没有通过构造函数指定终结点的三要素，会从配置文档中读取，配置文档内容：  
	
	<client>
            <endpoint address="http://127.0.0.1:9999/calculatorservice" binding="wsHttpBinding"
                bindingConfiguration="WSHttpBinding_CalculatrService" contract="CalculatorService.CalculatrService"
                name="WSHttpBinding_CalculatrService">
                <identity>
                    <userPrincipalName value="DESKTOP-VG1IL9U\lenovo" />
                </identity>
            </endpoint>
        </client>   
   
配置文档中，终结点的三要素均已指定，还添加有Name属性,这样构造函数中通过终结点的名称，实现服务代理对象的创建。如果基于服务契约的终结点在配置中是唯一的，那么构造函数可以不传递任何参数。因此对于上面客户端的创建：  
	
	CalculatrServiceClient client=new CalculatrServiceClient();   

也能完成客户端服务代理对象的创建    

- **通过ChannelFactory<TChannel>指定地址**   

当创建继承自ClientBase< TChannel >的客户端服务代理对象时，调用某个服务的时候，实际上是调用基类(ClientBase< TChannel >)的Channel属性的对应方法实现的  

	public double Add(double x, double y) {
    	return base.Channel.Add(x, y);
    	}

基类中的Channel属性实际上是通过WCF客户端框架的另一个重要对象创建的，即ChannelFactory< TChannel >,TChannel一般是服务契约类型。ChannelFactory< TChannel >不仅为ClientBase< TChannel >服务，同时也可以单独使用。通过ChannelFactory< TChannel >直接创建服务代理对象。 ChannelFactory< TChannel >的部分定义：  
	
	public class ChannelFactory<TChannel>{
	public ChannelFactory();  
	public ChannelFactory(string endpointConfigurationName);  
	public ChannelFactory(Binding binding);  
	public ChannelFactory(ServiceEndpoint endpoint); 
	public ChannelFactory(string endpointConfigurationName, EndpointAddress remoteAddress);  
	public ChannelFactory(Binding binding, string remoteAddress);  
	public ChannelFactory(Binding binding, EndpointAddress remoteAddress);  
	protected ChannelFactory(Type channelType);
	public static TChannel CreateChannel(Binding binding, EndpointAddress endpointAddress);
	public static TChannel CreateChannel(Binding binding, EndpointAddress endpointAddress, Uri via);
	}   
ChannelFactory< TChannel >和ClientBase< TChannel >具有类似的重载构造函数，指定终结点的三要素。如果将终结点的信息通过配置的方式给出，在进行ChannelFactory< TChannel >的构造函数时，直接传入终结点的名称的字符串   
	
	var channel = new ChannelFactory<ICommunicationContract>("wcfDemo");
    var client = channel.CreateChannel();  

**AddressHeader的指定**  
这里先看看EndpointAdress的定义   

	public class EndpointAddress{
	public Uri Uri { get; }
	public AddressHeaderCollection Headers { get; }
	public EndpointIdentity Identity { get; }
	}

AddressHeader定义在System.ServiceModel.Channels命名空间下，表示用于消息寻址相关的信息的报头，通过AddressHeader的静态方法CreatAddressHeader可以创建AddressHeader对象，除了AddressHeader类型，在System.ServiceModel.Channels命名空间下定义AddressHeaderCollection对象，表示AddressHeader的集合，继承自ReadOnlyCollection< AddressHeader >,该集合为只读集合。EndpointAddress的Headers属性的类型为AddressHeaderCollection，该属性为只读属性，所以不能通过该属性其EndpointAddress添加AddressHeader。   

**为服务指定AddressHeader**  
对服务进行寄宿时，可以通过代码和配置为EndpointAddress添加相应的AddressHeader。

![](http://i.imgur.com/yaynFQH.png)  

示例代码中，为确定受访者类型，购买服务的用户（Licensed user）和免费试用的用户（Trivial User）,添加一个Name为UserType的AddressHeader,NameSpace为 http://www.artech.com/。 目的是让这个终结点只能为第一类用户提供服务，将AddressHeader的Value设为Licensed User。通过AddressHeader.CreateAddressHeader静态方法创建AddressHeader对象，传入EndpointAddress的构造函数创建EndpointAddress对象，再创建ServiceEndpoint对象，添加到服务宿主的终结点中。  
通过配置的方式与之前类似，定义在< headers >的配置项中，指定Name，NameSpace,Value，与上面的等效配置如下 ： 

![](http://i.imgur.com/AUAXMIt.png)   

客户端同样通过代码和配置的方式对AddressHeader进行设定，采用配置的方式设定：   

![](http://i.imgur.com/hUNb2jh.png)  
**由于在服务端为服务的终结点指定了AddressHeader，意味着该终结点只接受消息的报头与该AddressHeader相匹配的消息请求。**  

----------
**5.端口共享**    
对于WCF来说，当对某个服务进行寄宿的时候，一个端口会被独占使用。例如使用两个控制台程序对两个服务Service1和Service2进行寄宿，两个服务的终结点地址设置为同一个，先后运行这两个服务寄宿的控制台程序时，第一个能正常运行，第二个则会报错，提示端口已经用于网络监听。    
当将某个服务寄宿于一个进程中，实际是通过该进程监听和处理来自客户端的Socket请求。一般情况下，一个端口被一个监听进程独占使用，如果部署了若干服务，这些服务寄宿于不同的应用程序中，那么监听的端口必须不同。
对于WCF寄宿的端口共享，采用不同的传输协议，有不同的解决方案。  

**基于HTTP/HTTPS的端口共享**   
HTTP/HTTPS最为常见的是80|443端口共享，对于WCF来说，基于80|443端口共享仅限于采用IIS寄宿方式的服务。如果采取自我寄宿的方式，80|443端口是不可用的。  

**基于TCP的端口共享**   
IIS只能接受基于HTTP的服务寄宿方式，如果采用TCP的服务，需要通过其他的寄宿方式。Windows提供Net.TCP端口共享服务来实现基于TCP的端口共享。WCF对Net.TCP端口共享服务提供原生支持，原理图：  

![](http://i.imgur.com/fubb3Gi.png)   

服务客户端proxy1和proxy2，分别调用Service1和Service2，当基于各自服务调用的socket连接请求抵达目标主机时，Net.TCP端口共享服务会截获请求消息，并获取目的地址。根据该地址，结合内部维护的目的地址和目标进程匹配列表，得到对应的目标应用程序，并将请求消息转发给真正的服务程序。    

WCF下基于TCP的端口共享建立在Net.TCP Port Sharing Service Windows服务上的。    
默认情况下，该服务需要手动开启。“开始”-->"控制面板"-->"管理工具"-->"服务"，定位到Net.TCP Port Sharing Service. 
在基于TCP的WCF通信中，NetTcpBinding实现了通信的细节，包括端口的共享。在NetTcpBinding，定义了一个特殊属性，PortSharingEnabled,表明是否启用端口共享机制。如果启用，需要设置该属性：  

	using（ServiceHost serviceHost = new ServiceHost(typeof(Service1))）{
		NetTcpBinding binding=new NetTcpBinding();
		binding.PortSharingEnabled=true;
		serviceHost.AddServiceEndpoint(typeof(IService1),binding,"net.tcp://127.0.0.1:9999/service1");
		serviceHost.Open();
		Console.Read();
	}   
也可以通过配置进行启用端口共享机制： 

![](http://i.imgur.com/7SmihKm.png)    

----------
**6.WCF寻址（Addressing）**  
WCF通过创建不同的终结点，实现基于不同传输协议的通信方式。对于通信来讲，首先要解决的是寻址（Addressing）问题，基于消息的通信方式首先要知道消息该发往何处。  

- **服务的角色**  

在具体一个服务调用中，客户端和服务端准确来说是服务的消费者和服务的提供者。在一个服务场景中，若为别人提供某种功能的实现调用，则为服务提供者；若该服务需要调用其他服务完成某项功能，则它为服务的消费者。服务的消费者向服务提供者发送消息请求。  

![](http://i.imgur.com/GHq0FxV.png)

除了服务的消费者和服务提供者外，有时候服务还具有第三种角色：中介服务。中介服务不提供具体业务功能的实现，而是将收到的服务请求转发给最终服务提供者。   

![](http://i.imgur.com/JCXYV49.png)    

- **逻辑地址&物理地址**  

WCF中，每个终结点包含两个不同地址：逻辑地址和物理地址。以终结点Address属性表示的地址为**逻辑地址**。从**消息发送端来** 讲，**物理地址**是消息被真正发送的目的地址，从**消息接收端** 来讲，**物理地址**是监听器真正监听的地址。逻辑地址和物理地址分别对应To和Via。 

**服务端逻辑地址与物理地址**  
对于消息的接收方的终结点来说，物理地址就是监听地址，通过ServiceEndpoint的ListenUri属性表示。  

![](http://i.imgur.com/s0NWPKu.png)

对服务进行寄宿的时候，可以调用ServiceHost的AddressEndpoint对应的重载方法为终结点指定ListenUri。例如：为终结点指定不同于逻辑地址的物理地址。

![](http://i.imgur.com/5z1tlGr.png)   

ListenUri也能通过配置方式指定：   

![](http://i.imgur.com/DabPcMf.png)  

**客户端逻辑地址与物理地址**     
于消息发送端来说，物理地址就是消息发送的真正目的地址。该地址通过特殊的终结点行为（EndpointBehavior）来指定：ClientViaBehavior。ClientViaBehavior定义的URI表示该物理地址。ClientViaBehavior通过相应的配置应用到WCF客户端。通过viaURI设置不同于逻辑地址的物理地址。   

![](http://i.imgur.com/yxiY0VV.png)

- **消息筛选**   
在WCF的消息分发系统中ChannelDispatcher（信道分发器）和ChannelListener（信道监听器）对象经常会被用到。  

**连接请求的监听**

当服务被成功寄宿时，WCF在服务端创建一个或多个监听器，用于服务调用请求的监听。例如，为CalculatorService添加三个BasicHttpBinding的终结点，为其中两个终结点指定一个不同于终结点地址的监听地址（物理地址），第三个终结点的监听地址（物理地址）与逻辑地址共享相同的地址。（默认情况下，监听地址与逻辑地址相同）	。对应配置：

![](http://i.imgur.com/ecrFqvL.png)	 

当前一个服务，3个终结点，2个监听地址。当服务成功寄宿时，WCF创建两个ChannelDispatcher(信道分发器)对象，每个信道分发器有自己的信道监听器(ChannelListener)，分别绑定到监听地址对应的端口进行服务调用请求的监听。WCF还会创建3个EndpointDispatcher（终结点分发器），当信道分发器通过信道监听接收到消息，根据消息自身选择终结点分发器。这种根据消息选择终结点分发器的选择机制叫做**消息筛选**。关系图为：  

![](http://i.imgur.com/ml7Vx86.png)   

**EndpointDispatcher的选择和消息的分发**   
在WCF消息监听与发布中，ChannelDisPatcher(信道分发器)和EndpointDispatcher（终结点分发器）是两个核心对象。信道分发器监听请求和接收消息，分发给相应终结点分发器，由终结点分发器最终完成消息的处理。消息筛选解决对终结点的选择问题，消息筛选依赖于**终结点分发器** 的两个对象 **Addressfilter** 和 **Contractfilter**,从名称可以看出，分别对终结点的地址和契约进行筛选。 
**Addressfliter** 和 **Contractfliter** 是同一种类型：System.ServiceModel.Dispatcher.MessageFileter,定义两个Match重载方法，来判断拥有MessageFilter的EndpointDispatcher是否和接收的消息相匹配。  

![](http://i.imgur.com/dBYOcVg.png)   

当ChannelDispatcher接收到客户端的请求消息时，遍历属于自己的EndpointDispatcher列表，获取它们的MessageFilter:Addressfilter和Contractfilter，将**消息对象**传入Match方法。若返回值为True，则该EndpointDispatcher为该消息的真正目标终结点。  


----------

**7.绑定与信道栈**    
WCF在基础架构的层次架构上分为两个部分：服务模型层（Service Model）和信道层（Channel Layer）。服务模型层建立在信道层之上，提供统一，可扩展的编程模型。信道层是通过绑定创建的信道栈为消息通信提供传输、处理的通道。信道栈是由承载不同功能的信道组合而成。    
绑定在两个部分中扮演中间人角色。    
在服务端，WCF通过服务终结点的绑定创建一个或多个信道监听器，绑定到监听端口进行请求监听。    
在客户端，通过绑定创建信道工厂，借助信道工厂创建的服务代理对象进行服务的请求和调用。

**实例：通过绑定进行消息通信**
整个解决方案由两个Console应用程序组成，分别模拟消息的监听端和请求端。   
**Step1:创建请求监听端应用程序**  

	namespace Listener
	{
    class Program
    {
        static void Main(string[] args)
        {
            //创建监听地址和绑定
            Uri listenUri=new Uri("http://localhost:9999/listener");
            Binding binding=new BasicHttpBinding();
            //通过监听地址创建信道监听器
            IChannelListener<IReplyChannel> channelListener = binding.BuildChannelListener<IReplyChannel>(listenUri);
            //开启信道监听器
            channelListener.Open();

            //通过信道监听器对象创建回复信道对象
            IReplyChannel channel = channelListener.AcceptChannel(TimeSpan.MaxValue);
            //开启回复信道
            channel.Open();

            Console.WriteLine("开始监听...");

            while (true)
            {
                //通过回复信道创建请求文本对象接收请求内容
                RequestContext requestContext = channel.ReceiveRequest(TimeSpan.MaxValue);
                //输出请求文本对象里存储的从请求端发送的请求信息
                Console.WriteLine("接收到请求消息：\n{0}",requestContext.RequestMessage);
                //通过请求文本对象的Reply()方法向请求端发送回复信息
                requestContext.Reply(CreateReplyMessage(binding));
            }
        }

        static Message CreateReplyMessage(Binding binding)
        {
            string action = "urn:zhangbo/reply";
            string body = "这是一个简单的回复消息";

            return Message.CreateMessage(binding.MessageVersion, action, body);
        }
    }
	}  

**Step2:创建请求发送端应用程序**   

	namespace Sender
	{
    class Program
    {
        static void Main(string[] args)
        {
            //创建请求的目的地址和绑定
            Uri listenUri=new Uri("http://localhost:9999/listener");
            Binding binding=new BasicHttpBinding();
            //通过绑定创建信道工厂
            IChannelFactory<IRequestChannel> channelFactory = binding.BuildChannelFactory<IRequestChannel>();
            //开启信道工厂
            channelFactory.Open();

            //通过信道工厂创建请求信道，传入终结点地址对象
            IRequestChannel channel = channelFactory.CreateChannel(new EndpointAddress(listenUri));
            //开启请求信道
            channel.Open();

            //通过请求信道发送请求，并接受请求回复信息
            Message replyMessage = channel.Request(CreateRequestMessage(binding));
            //输出请求回复信息
            Console.WriteLine("接收到回复消息\n{0}",replyMessage);
            Console.Read();
            }

        static Message CreateRequestMessage(Binding binding)
        {
            string action = "urn:zhangbo/request";
            string body = "这是一个简单的请求消息！";

            return Message.CreateMessage(binding.MessageVersion,action,body);
        }
    }
	}   

**执行结果：**  
请求监听端：  

![](http://i.imgur.com/RNh3TkE.png)   

请求发送端：  

![](http://i.imgur.com/FRnr0R7.png)   

**WCF的绑定模型**    
在整个绑定模型中，最底层的是信道和信道栈。不同功能的信道组成信道栈（消息通信的通道），根据功能信道可分为三类：

- **传输信道（Transport Channel）**  
传输信道实现基于某种网络协议（HTTP、HTTPS、TCP等）的消息传输

- **消息编码信道（Encoding Channel）**   
消息编码信道实现对消息的编码，常见消息编码方式有：Text/XML、Binary和MTOM  

- **协议信道（ProtocolChannel）**  
协议信道实现WCF对WS-*协议的支持，如WS-Security、WS-RM、WS-AT   
  
WCF中，信道是通过信道管理器创建（信道监听器和信道工厂都是信道管理器），服务端的信道管理器作用在于创建信道栈监听请求，接收消息，称为**信道监听器**，而客户端的信道管理器在于创建信道进行消息请求的发送和回复消息的接收，所以称为**信道工厂**，而信道管理器都会由一个绑定实例进行创建，上述关系可以由以下关系图表示： 

![](http://i.imgur.com/FtlL2sg.png)   

**绑定编程**  
对于WCF来说，客户端和服务端实现通信是通过终结点进行。作为终结点的三要素之一，绑定实现消息的通信细节。终结点的创建过程中，必须指定相应的绑定。  

- **服务端对绑定的指定**   
服务端在自我寄宿时，会创建ServiceHost对象，通过ServiceHost.AddServiceEndpoint方法对终结点进行添加，AddServiceEndpoint要求传入指定的Binding对象，定义在ServiceHost和ServiceHostBase中的AddServiceEndpoint方法： 

![](http://i.imgur.com/jgBykMt.png)  

为CalculatorService服务添加3个终结点，3个终结点分别对应各自地址的绑定类型：BasicHttpBinding、WsHttpBinding、NetTcpBinding   

![](http://i.imgur.com/XE0oGdO.png)     

终结点的配置也可以通过WCF配置完成，绑定在对应的设置过程中进行设置。  

- **客户端对绑定的指定**    
客户端是服务的调用端，服务调用有两种方式，一种是通过代码生成工具或者添加服务引用，第二种是通过ChannelFactory< TChannel >进行创建服务代理对象。  

第一种方式生成继承自System.ServiceModel.ClientBase<TChannel>的子类，该子类的实例对象即为服务的代理对象，在构造过程中，可以通过其构造方法进行对绑定的指定。   

第二种方式直接通过ChannelFactory< TChannel >创建服务代理对象或是信道栈，其构造方法指定传入相应绑定对象。    

两种方式都可以通过配置的方式去指定绑定，当代码中没有指定终结点的三要素时，会采取配置中的配置内容。  

**消息交换与信道形状**   
在SOA中，消息交换模式（MEP）代表一系列模板，定义了消息的发送者和接受者相互进行**消息传输的次序**。 典型的消息交换模式包括3种：数据报模式（Datagram）、请求/恢复模式(Request/Reply)及双工模式（Duplex）。  


- **数据报模式（Datagram）**   
数据报模式是最简单的消息交换模式，也叫做发送/遗忘或单向模式。数据报模式基于从一个源到一个或多个目的地的**单向消息传输**。 数据报模式的一些变型：  

- 单目的地模式   
一个消息的发送方将消息发送给单一接收方
- 多投模式   
一个消息发送方将消息发送给一系列预定义的接收方   
- 广播模式    
和多投模式类似，接收方的范围更加广泛     
数据报模式一般采用**异步**的消息发送模式，**并不希望接收到对方的回复消息**  个别情况下，甚至不关心消息能否被正常接收。  

- **请求/回复模式（Request/Reply）**   
该模式是使用的最多的模式。这种模式下，消息发送方将消息发送给接收方等待对方回复。请求/回复模式一般采用同步通信方式。   

- **双工模式（Duplex）**  
双工模式下，消息在交换过程中，任何一方都可以向对方发送消息。双工通信可以使服务端回调客户端。比较典型的双工通信是订阅/发布模式。订阅/发布模式下消息交换的双方从发送方和接收方变成了订阅方和发布方。 

消息的交换依赖网络传递，不同的网络传输协议对双工通信具有不同的支持方式。TCP其协议本身就是全双工的网络通信协议，提供双工通信的原生支持。HTTP本身是基于请求/回复的网络协议，不支持双工通信。WCF通过WsDualHttpBinding实现基于HTTP的双工通信，实际是采用两个HTTP通道实现的。

**信道形状**   
不同消息交换模式下，管道在消息发送端和接收端作用不同。   

- **数据报模式下**  
发送端信道栈作用：输出数据   
接收端信道栈作用：输入数据    

- **请求/回复模式下**  
发送端信道栈作用：发送消息请求   
接收端信道栈作用：接收和回复消息请求   

- **双工模式下**  
消息双方完全等价，均具有输入和输出的功能  

WCF通过**信道形状（Channel Shape）**来表述不同消息交换模式下消息交换双方信道的不同要求。信道形状按照消息交换模式的不同，将信道进行如下分类（WCF为信道定义一系列接口）：   

![](http://i.imgur.com/a1NqiPJ.png)
 
接口之间层次结构：  

![](http://i.imgur.com/TvX2QrH.png)

**创建自定义信道**    
自定义信道的方法和属性，仅仅通过System.Console在控制台打印方法和属性名称，可以通过自身需求进行扩展。静态打印类及打印方法：

	public static class PrintHelper
    {
        public static void Print(object instance,string method)
        {
            Console.WriteLine("{0}.{1}",instance.GetType().Name,method);
        }
    }
  
自定义请求信道SimpleRequestChannel实现IRequest接口，并直接继承自ChannelBase。  

	namespace CustomChannel
	{
    public class SimpleRequestChannel:ChannelBase,IRequestChannel
    {
        private IRequestChannel _innerChannel;

        public SimpleRequestChannel(ChannelManagerBase channelManager, IRequestChannel innnerChannel):base(channelManager)
        {
            PrintHelper.Print(this,"SimpleRequestChannel");
            this._innerChannel = innnerChannel;
        }

        protected override void OnAbort()
        {
            PrintHelper.Print(this,"OnAbort");
            this._innerChannel.Abort();
        }

        protected override void OnClose(TimeSpan timeout)
        {
            PrintHelper.Print(this, "OnClose");
            this._innerChannel.Close(timeout);
        }

        protected override void OnEndClose(IAsyncResult result)
        {
            PrintHelper.Print(this, "OnEndClose");
            this._innerChannel.EndClose(result);
        }

        protected override IAsyncResult OnBeginClose(TimeSpan timeout, AsyncCallback callback, object state)
        {
            PrintHelper.Print(this, "OnBeginClose");
            return this._innerChannel.BeginClose(timeout,callback,state);
        }

        protected override void OnOpen(TimeSpan timeout)
        {
            PrintHelper.Print(this, "OnOpen");
            this._innerChannel.Open(timeout);
        }

        protected override IAsyncResult OnBeginOpen(TimeSpan timeout, AsyncCallback callback, object state)
        {
            PrintHelper.Print(this, "OnBeginOpen");
            return this._innerChannel.BeginOpen(timeout, callback, state);
        }

        protected override void OnEndOpen(IAsyncResult result)
        {
            PrintHelper.Print(this, "OnEndOpen");
            this._innerChannel.EndOpen(result);
        }

        #region IRequestChannel Members
        public Message Request(Message message)
        {
            PrintHelper.Print(this,"Request");
            return this._innerChannel.Request(message);
        }

        public Message Request(Message message, TimeSpan timeout)
        {
            PrintHelper.Print(this, "Request");
            return this._innerChannel.Request(message,timeout);
        }

        public IAsyncResult BeginRequest(Message message, AsyncCallback callback, object state)
        {
            PrintHelper.Print(this,"BeginReques");
            return this._innerChannel.BeginRequest(message,callback,state);
        }

        public IAsyncResult BeginRequest(Message message, TimeSpan timeout, AsyncCallback callback, object state)
        {
            PrintHelper.Print(this,"BeginRequest");
            return this._innerChannel.BeginRequest(message, timeout,callback, state);
        }

        public Message EndRequest(IAsyncResult result)
        {
            PrintHelper.Print(this,"EndRequest");
            return this._innerChannel.EndRequest(result);
        }

        public EndpointAddress RemoteAddress {
            get
            {
              PrintHelper.Print(this,"RemoteAddress");
              return this._innerChannel.RemoteAddress;
            } 
        }
        public Uri Via {
            get
            {
                PrintHelper.Print(this,"Via");
                return this._innerChannel.Via;
            }
        }

        #endregion
    }
	}

信道自身不能孤立存在，存在于一个或多个信道对象连接而成的信道栈中。因此，对于不在栈尾的信道来说，处理完消息，一般会把处理后的消息传递给下一个信道。反映在方法上，需要执行当前信道的某个方法后，获取下一个信道对象，调用同名方法。  

自定义回复信道与自定义请求信道类似，实现IReplyChannel接口，直接继承自ChannelBase。  
自定义会话信道SimpleDuplexSessionChannel,实现IDuplexSessionChannel接口，并直接继承自ChannelBase。  

自定义的信道完成后，并不能直接通过信道管理器创建自定义信道对象，还需要对信道管理器进行自定义。

**信道管理器**     
信道管理器是信道的创建者，信道栈中的每一个信道都对应一个信道管理器。服务端的信道用于监听来自客户端的请求，又称为信道监听器；客户端的信道创建用于请求发送和回复接收的通道，又称为信道工厂。    

**信道监听器**

- **基于数据报信道的监听机制**   
对于数据报的消息交换模式下，消息发送是单向性的，当服务端开始监听时，监听信道会被创建出来，一旦消息请求被成功监听，无论来自哪个客户端（服务代理），信道监听器都会使用建好的信道栈对消息进行接收和处理。而对于服务端来说，相同的监听地址采用同一个信道栈。  

- **基于会话信道的监听机制**    
对于会话场景，信道监听器创建出的信道栈不是共享，而是和客户端一一匹配。信道栈的创建时机在成功监听到消息请求，并不是开始监听的时候。对于来自某个客户端的第一次请求，信道监听器会为之创建一个新的信道栈，因此WCF具有最大并发会话的限制。  

信道监听器是位于服务端的信道管理器，所有的信道监听器继承自基类：ChannelManagerBase。WCF定义了相关的接口和抽象基类。对应关系为：  
![](http://i.imgur.com/SSR37UW.png)


**创建自定义信道监听器**  

- **创建数据报信道监听器**  
自定义信道监听器SimpleDatagramChannelListener<TChannel>继承泛型ChannelListennerBase< TChannel >。某一个信道只负责某个信道的创建，整个信道栈的创建依赖相关信道监听器的协同工作。在创建自定义信道监听器时，需要指定一个内部信道监听器（_innerChannelListener），代表下一个信道对应的信道监听器。构造函数中，_innerChannelListener通过传入的BindingContext创建。自定义信道监听器最重要的功能是创建自定义ReplyChannel:SimpleReplyChannel。SimpleReplyChannel的创建实现在OnAcceptChannel和OnEndAcceptChannel方法中。 

		 public class SimpleDatagramChannelListener<TChannel>:ChannelListenerBase<TChannel> where    TChannel:class ,IChannel
  	 	{
       	private IChannelListener<TChannel> _channelListener;

       	public SimpleDatagramChannelListener(BindingContext context)
       	{
            PrintHelper.Print(this,"SimpleDatagramChannelListener");
           this._channelListener = context.BuildInnerChannelListener<TChannel>();
       	}

       	protected override void OnAbort()
       	{
           PrintHelper.Print(this,"OnAbort");
            this._channelListener.Abort();
       	}

       	protected override void OnClose(TimeSpan timeout)
       	{
           PrintHelper.Print(this,"OnClose");
            this._channelListener.Close();
       	}

       	protected override void OnEndClose(IAsyncResult result)
       	{
           PrintHelper.Print(this,"OnEndClose");
           this._channelListener.EndClose(result);
       	}

       	protected override IAsyncResult OnBeginClose(TimeSpan timeout, AsyncCallback callback, object state)
       	{
           PrintHelper.Print(this,"OnBeginClose");
           return this._channelListener.BeginClose(callback, state);
       	}

       	protected override void OnOpen(TimeSpan timeout)
       	{
          PrintHelper.Print(this,"OnOpen");
          this._channelListener.Open(timeout);
       	}

       	protected override IAsyncResult OnBeginOpen(TimeSpan timeout, AsyncCallback callback, object state)
       	{
           PrintHelper.Print(this,"OnBeginOpen");
           return this._channelListener.BeginOpen(callback, state);
       	}

       	protected override void OnEndOpen(IAsyncResult result)
       	{
           PrintHelper.Print(this,"OnEndOpen");
            this._channelListener.EndOpen(result);
       	}

       	protected override bool OnWaitForChannel(TimeSpan timeout)
       	{
           PrintHelper.Print(this,"OnWaitForChannel");
           return this._channelListener.WaitForChannel(timeout);
       	}

       	protected override IAsyncResult OnBeginWaitForChannel(TimeSpan timeout, AsyncCallback callback, object state)
       	{
           PrintHelper.Print(this,"OnBeginWaitForChannel");
           return this._channelListener.BeginAcceptChannel(timeout,callback, state);
       	}

       	protected override bool OnEndWaitForChannel(IAsyncResult result)
       	{
           PrintHelper.Print(this,"OnEndWaitForChannel");
           return this._channelListener.EndWaitForChannel(result);
       	}

       	public override Uri Uri {
           get
           {
               PrintHelper.Print(this,"Uri");
               return this._channelListener.Uri;
           }
       	}
       	protected override TChannel OnAcceptChannel(TimeSpan timeout)
       	{
           PrintHelper.Print(this,"OnAcceptChannel");
            IReplyChannel innerChannel=this._channelListener.AcceptChannel(timeout) as IReplyChannel;
            return new SimpleReplyChannel(this,innerChannel) as TChannel;
       	}

       	protected override IAsyncResult OnBeginAcceptChannel(TimeSpan timeout, AsyncCallback callback, object state)
       	{
           PrintHelper.Print(this,"OnBeginAcceptChannel");
           return this._channelListener.BeginAcceptChannel(timeout, callback, state);
       	}

       	protected override TChannel OnEndAcceptChannel(IAsyncResult result)
       	{
           PrintHelper.Print(this,"OnEndAcceptChannel");
           return new SimpleReplyChannel(this,this._channelListener.EndAcceptChannel(result) as IReplyChannel) as TChannel;
       	}
    	}

- **创建会话信道监听器**   
自定义会话信道监听器SimpleSessionChannelListener<TChannel>。定义与上面类似，不同在于需要重写GetProperty<T>方法。  

		public class SimpleSessionChannelListener<TChannel>:ChannelListenerBase<TChannel> where TChannel:class,IChannel
    	{
        private IChannelListener<TChannel> _channelListener;

        public SimpleSessionChannelListener(BindingContext context)
        {
            PrintHelper.Print(this,"SimpleChannelListener");
            this._channelListener = context.BuildInnerChannelListener<TChannel>();
        }

        protected override void OnAbort()
        {
            PrintHelper.Print(this,"OnAbort");
            this._channelListener.Abort();
        }

        protected override void OnClose(TimeSpan timeout)
        {
            PrintHelper.Print(this,"Close");
            this._channelListener.Close(timeout);
        }

        protected override void OnEndClose(IAsyncResult result)
        {
            PrintHelper.Print(this,"EndClose");
            this._channelListener.EndClose(result);
        }

        protected override IAsyncResult OnBeginClose(TimeSpan timeout, AsyncCallback callback, object state)
        {
            PrintHelper.Print(this,"OnBeginClose");
            return this._channelListener.BeginClose(timeout,callback,state);
        }

        protected override void OnOpen(TimeSpan timeout)
        {
            PrintHelper.Print(this,"OnOpen");
            this._channelListener.Open(timeout);
        }

        protected override IAsyncResult OnBeginOpen(TimeSpan timeout, AsyncCallback callback, object state)
        {
            PrintHelper.Print(this,"OnBeginOpen");
            return this._channelListener.BeginOpen(timeout, callback, state);
        }

        protected override void OnEndOpen(IAsyncResult result)
        {
            PrintHelper.Print(this,"EndOpen");
            this._channelListener.EndOpen(result);
        }

        protected override bool OnWaitForChannel(TimeSpan timeout)
        {
            PrintHelper.Print(this,"OnWaitForChannel");
            return this._channelListener.WaitForChannel(timeout);
        }

        protected override IAsyncResult OnBeginWaitForChannel(TimeSpan timeout, AsyncCallback callback, object state)
        {
            PrintHelper.Print(this,"OnBeginWaitForChannel");
            return this._channelListener.BeginAcceptChannel(timeout, callback, state);
        }

        protected override bool OnEndWaitForChannel(IAsyncResult result)
        {
            PrintHelper.Print(this,"OnEndWaitForChannel");
            return this._channelListener.EndWaitForChannel(result);
        }

        public override Uri Uri {
            get
            {
                PrintHelper.Print(this,"Uri");
                return this._channelListener.Uri;
            }
        }
        protected override TChannel OnAcceptChannel(TimeSpan timeout)
        {
            PrintHelper.Print(this,"OnAcceptChannel");
            IDuplexSessionChannel innerChannel=this._channelListener.AcceptChannel(timeout) as IDuplexSessionChannel;
            return new SimpleDuplexSessionChannel(this,innerChannel) as TChannel;
        }

        protected override IAsyncResult OnBeginAcceptChannel(TimeSpan timeout, AsyncCallback callback, object state)
        {
            PrintHelper.Print(this,"OnBeginAcceptChannel");
            return this._channelListener.BeginAcceptChannel(timeout, callback, state);
        }

        protected override TChannel OnEndAcceptChannel(IAsyncResult result)
        {
            PrintHelper.Print(this,"OnEndAcceptChannel");
            return new SimpleDuplexSessionChannel(this,this._channelListener.EndAcceptChannel(result) as IDuplexSessionChannel) as TChannel;
        }

        public override T GetProperty<T>()
        {
            return this._channelListener.GetProperty<T>();
        }
    	} 


**信道工厂**   
信道工厂是客户端的信道管理器别名，信道工厂作用是单纯地创建用于发送请求和接受回复的信道。WCF为信道工厂创建两个接口：IChannelFactory和IChannelFactory<TChannel>和两个抽象基类：ChannelFactoryBase和ChannelFactoryBase<TChannel>。继承关系及接口实现关系：  

![](http://i.imgur.com/f5ywNmP.png)  

**创建数据报信道工厂**   
为之前的SimpleRequestChannel创建信道工厂：SimpleDatagramChannelFactory<TChannel>直接继承抽象基类ChannelFactoryBase<TChannel>。OnCreatChanel是核心方法，实现真正信道创建。     

		public class SimpleDatagramChanelFactory<TChannel>:ChannelFactoryBase<TChannel>
    {
        private IChannelFactory<TChannel> _innerChannelFactory;

        public SimpleDatagramChanelFactory(BindingContext context)
        {
            PrintHelper.Print(this,"SimpleDatagramChannelFactory");
            this._innerChannelFactory = context.BuildInnerChannelFactory<TChannel>();
        }

        protected override void OnOpen(TimeSpan timeout)
        {
            PrintHelper.Print(this,"OnOpen");
            this._innerChannelFactory.Open(timeout);
        }

        protected override IAsyncResult OnBeginOpen(TimeSpan timeout, AsyncCallback callback, object state)
        {
            PrintHelper.Print(this,"OnBeginOpen");
            return this._innerChannelFactory.BeginOpen(timeout,callback,state);
        }

        protected override void OnEndOpen(IAsyncResult result)
        {
            PrintHelper.Print(this,"OnEndOpen");
            this._innerChannelFactory.EndOpen(result);
        }

        protected override TChannel OnCreateChannel(EndpointAddress address, Uri via)
        {
            PrintHelper.Print(this,"OnCreatChannel");
            return (TChannel)(object)new SimpleRequestChannel(this,this._innerChannelFactory.CreateChannel(address,via) as IRequestChannel);
        }
    }   

**创建会话信道工厂**   
为之前的会话信道SimpleDuplexSessionCahnnel创建信道工厂：SimpleSessionChannelFactory<TChannel>。  

		 public class SimpleSessionChannelFactory<TChannel>:ChannelFactoryBase<TChannel>
    {
        private IChannelFactory<TChannel> _innerChannelFactory;

        public SimpleSessionChannelFactory(BindingContext context)
        {
            PrintHelper.Print(this,"SimpleSessionChannelFactory");
            this._innerChannelFactory = context.BuildInnerChannelFactory<TChannel>();
        }

        protected override void OnOpen(TimeSpan timeout)
        {
            PrintHelper.Print(this,"OnOpen");
            this._innerChannelFactory.Open(timeout);
        }

        protected override IAsyncResult OnBeginOpen(TimeSpan timeout, AsyncCallback callback, object state)
        {
            PrintHelper.Print(this,"OnBeginOpen");
            return this._innerChannelFactory.BeginOpen(timeout, callback, state);
        }

        protected override void OnEndOpen(IAsyncResult result)
        {
            PrintHelper.Print(this,"OnEndOpen");
            this._innerChannelFactory.EndOpen(result);
        }

        protected override TChannel OnCreateChannel(EndpointAddress address, Uri via)
        {
            PrintHelper.Print(this,"OnCreatChannel");
            return (TChannel)(object)new SimpleDuplexSessionChannel(this,this._innerChannelFactory.CreateChannel(address,via) as SimpleDuplexSessionChannel);
        }

        public override T GetProperty<T>()
        {
            return this._innerChannelFactory.GetProperty<T>();
        }
    }


**绑定与绑定元素**     
 
绑定对象作为终结点三要素实现所有通信细节，通过创建信道栈实现消息传递。绑定是信道层所有通信对象的最终缔造者，信道工厂和信道监听器最终的创建靠绑定对象实现。创建过程和另一个重要对象密切相关，即绑定元素。    
**绑定元素**   
一个绑定对象由一系列绑定元素组成，每个绑定元素负责创建相应的信道。绑定元素集合的构成及先后顺序，最终决定信道栈中的信道及先后顺序。信道可分为**传输信道**、**消息编码信道** 和 **协议信道**，相对的绑定元素可分为**传输绑定元素**、**消息编码绑定元素** 和 **协议绑定元素**    
绑定元素的功能实现对信道监听器和信道工厂的创建。所有绑定元素的基类：System.ServiceModel.Channels.BindingElement的定义： 
![](http://i.imgur.com/ftrlJMK.png)   
定义中的Build方法用于创建对应的信道监听器和信道工厂。  

**自定义绑定元素**     
根据之前创建的数据报信道管理器SimpleDatagramChannelFactory和SimpleDataChannelListener，创建相应的绑定元素SimpleDatagramElement。   

	 public class SimpleDatagramBindingElement:BindingElement
    {
        public SimpleDatagramBindingElement()
        {
            PrintHelper.Print(this,"SimpleDatagramBindingElement");
        }

        public override BindingElement Clone()
        {
            PrintHelper.Print(this,"Clone");
            return new SimpleDatagramBindingElement();
        }

        public override T GetProperty<T>(BindingContext context)
        {
            PrintHelper.Print(this,string.Format("GetProperty<{0}>",typeof(T).Name));
            return context.GetInnerProperty<T>();
        }

        public override IChannelFactory<TChannel> BuildChannelFactory<TChannel>(BindingContext context)
        {
            PrintHelper.Print(this,"BuildChannelFactory<TChannel>");
            return new SimpleDatagramChanelFactory<TChannel>(context) as IChannelFactory<TChannel>;
        }

        public override IChannelListener<TChannel> BuildChannelListener<TChannel>(BindingContext context)
        {
            PrintHelper.Print(this, "BuildChannelListener<TChannel>");
            return new SimpleDatagramChannelListener<TChannel>(context) as IChannelListener<TChannel>;
        }
    }  
根据之前创建的会话信道监听器和信道工厂，创建自定义会话绑定元素SimpleSessionBindingElement。  

	public class SimpleSessionBindingElement:BindingElement
    {
        public SimpleSessionBindingElement()
        {
            PrintHelper.Print(this,"SimpleSessionBindingElement");
        }

        public override BindingElement Clone()
        {
            PrintHelper.Print(this,"Clone");
            return new SimpleSessionBindingElement();
        }

        public override T GetProperty<T>(BindingContext context)
        {
            PrintHelper.Print(this,string.Format("GetProperty<{0}>",typeof(T).Name));
            return context.GetInnerProperty<T>();
        }

        public override IChannelFactory<TChannel> BuildChannelFactory<TChannel>(BindingContext context)
        {
            PrintHelper.Print(this, "BuildChannelFactory<TChannel>");
            return new SimpleSessionChannelFactory<TChannel>(context) as IChannelFactory<TChannel>;
        }

        public override IChannelListener<TChannel> BuildChannelListener<TChannel>(BindingContext context)
        {
            PrintHelper.Print(this, "BuildChannelListener<TChannel>");
            return new SimpleSessionChannelListener<TChannel>(context) as IChannelListener<TChannel>;
        }
    }  

**绑定是绑定元素的有序集合**     
绑定的目标是实现对信道栈的创建，绑定元素决定信道的创建，绑定对象本身的特性与能力由自身包含的所有绑定元素，及绑定元素之间的先后次序决定。  
当判断一个绑定类型是否支持某种特性，只需查看该绑定是否具有与该特性相关的绑定元素。 
WCF中，所有绑定直接或间接继承自抽象基类：System.ServiceModel.Channels.Binding。对于Binding,最重要的是构建组成该绑定对象的所有绑定元素集和。基于绑定元素的创建，通过抽象方法CreateBindingElements实现，具体的绑定类型均要实现该方法。   

	public abstract class Binding:IDefaultCommunicationTimeouts
	{
		//其他成员
		public abstract BindingElementColletion CreateBindingElements();
	}
 
**创建自定义绑定**  
之前创建了自定义信道、信道监听器、信道工厂与绑定元素。这些对象只有通过具体的绑定对象才能应用到WCF的运行环境中。   
自定义绑定中仅包含三个必须绑定元素：**传输绑定元素** 和 **消息编码绑定元素** 及 **自定义绑定元素** ，传输绑定元素采用基于HTTP协议的HttpTransportBindingElement,消息编码采用基于文本编码方式的TextMessageEncodingBindingElement。自定义绑定为SimpleDatagramBinding。  

	public class SimpleDatagramBinding:Binding
    {
        private TransportBindingElement _transportBindingElement=new HttpTransportBindingElement();
        private MessageEncodingBindingElement _messageEncodingBindingElement=new TextMessageEncodingBindingElement();
        private SimpleDatagramBindingElement _simpleDatagramBindingElement=new SimpleDatagramBindingElement();
        public override BindingElementCollection CreateBindingElements()
        {
            BindingElementCollection elements=new BindingElementCollection();
            elements.Add(this._simpleDatagramBindingElement);
            elements.Add(this._messageEncodingBindingElement);
            elements.Add(this._transportBindingElement);
            return elements;
        }

        public override string Scheme {
            get { return this._transportBindingElement.Scheme; }
        }
    }   
Scheme为只读属性，返回HttpTransportBindingElement的Scheme:http。   

**使用自定义绑定案例**    
**Contract**   
	
		namespace Contract
	{
    	[ServiceContract]
    	public interface IService
    	{
        [OperationContract]
        void Dosomething();
    	}
	}
**Service**   
		
		namespace Services
	{
    public class Service:IService
    {
        public void Dosomething()
        {
            Console.WriteLine("Done...");
        }
    }
	}
**服务寄宿**   

	  static void Main(string[] args)
        {
            using (ServiceHost host = new ServiceHost(typeof (Service)))
            {
                host.AddServiceEndpoint(typeof (IService), new SimpleDatagramBinding(), "http://localhost:9999/service");
                host.Open();

                Console.Read();
            }
        }
    }  
**Clients**  

	static void Main(string[] args)
        {
            EndpointAddress address=new EndpointAddress("http://localhost:9999/service");
            using (
                ChannelFactory<IService> channelFactory = new ChannelFactory<IService>(new SimpleDatagramBinding(),
                    address))
            {
                IService proxy = channelFactory.CreateChannel();
                proxy.Dosomething();
                proxy.Dosomething();
                (proxy as ICommunicationObject).Close();

                proxy = channelFactory.CreateChannel();
                proxy.Dosomething();
                proxy.Dosomething();
                (proxy as ICommunicationObject).Close();
            }
            Console.Read();
        }
    } 

服务启动后输出：  
![](http://i.imgur.com/nXWUE3H.png)  
绑定对象先通过绑定元素，调用BuildChannelListener< TChannel >创建信道监听器，信道监听器开启后，调用BeginAcceptChannel创建信道，信道开启后监听来自用户请求，并利用创建的信道栈接收请求消息。  

客户端调用服务客户端输出：  
![](http://i.imgur.com/XQoW0PA.png)
客户端通过绑定元素，调用BuildChannelFactory< TChannel >创建信道工厂，信道工厂开启后，通过CreatChannel创建信道，通过信道第一次调用时，信道开启，开启的信道可多次用于服务调用。 

客户端调用服务后，服务端的输出：  
![](http://i.imgur.com/J7BEDrx.png)   

**系统绑定与自定义绑定**