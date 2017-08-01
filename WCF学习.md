## WCF学习 ##
WCF：Windows Communication Foundation  
视窗操作系统通信框架，.net3.5后支持，.net中提供的统一的通信编程模型，通过配置文件对通信方式进行修改。  

----------
WCF程序的基本组成包括：  

- **契约**（服务端和客户端之间的通信约定）   
- **服务端**（实现契约接口和提供服务）   
- **客户端**（使用，消费服务端提供的服务）  
- **配置文件**（对通信方式进行配置）  

**入门小实例**  

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
通过右键 编辑WCF配置 对App.config进行向导式配置，添加客户端和绑定，添加绑定时制定一个绑定节点名，添加客户端的地址应与服务端添加服务时的地址一致，通过向导式配置得到的客户端配置文件：    

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