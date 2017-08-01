## WCF学习 ##
WCF：Windows Communication Foundation  
视窗操作系统通信框架，.net3.5后支持，.net中提供的统一的通信编程模型，通过配置文件对通信方式进行修改。  

----------
WCF程序的基本组成包括：  

- **契约**（服务端和客户端之间的通信约定）   
- **服务端**（实现契约接口和提供服务）   
- **客户端**（使用，消费服务端提供的服务）  
- **配置文件**（对通信方式进行配置）  

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

新建控制台应用程序作为客户端使用服务端提供的程序，添加对契约项目的引用     

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
