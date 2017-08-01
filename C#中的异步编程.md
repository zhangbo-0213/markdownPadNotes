# C#中的异步编程 #
----------
## 进程与线程 ##
程序在启动时，系统会在内存中创建一个进程。进程是程序运行所需资源的集合，这些资源包括虚地址空间、文件句柄和其他程序运行所需的东西。在进程的内部，系统创建一个称为线程的内核对象，代表真正执行的程序。当线程被建立时，系统在Main方法的第一行语句处开始执行线程。关于线程：  

- 默认情况下，一个进程只包含一个线程，从程序开始执行到结束；
- 线程可以派生其他线程，因此一个进程可能包含多个不同状态的线程，执行程序的不同部分；
- 一个进程如果包含多个线程，这些线程共享进程的资源；
- 系统中处理器执行的规划单元是线程，不是进程。  
##  ##
## 同步与异步 ##
- 同步是指从语句出现的先后顺序执行直到完成。
- 异步指语句并不严格按照出现的顺序执行。有时需要在一个新的线程中运行一部分代码，有时无需创建新的线程，为了提高单个线程的效率，改变代码的执行顺序。 

当某个操作需要花费大量的时间进行处理，若是使用同步编程，那么程序在等待响应的时间内不能处理其他事物，这样效率比较低；而使用异步编程时，在进行等待相应的时间内，程序可以利用等待的时间处理其他事物，当得到响应时，再回到响应处继续执行，这样程序的效率会更高。
##  ##
## 同步方法与异步方法 （async/await）##
如果某个方法被调用时，等待所有的执行操作完成后在进行后续操作，该方法为同步的；异步方法则在该方法处理完成前就回到被调用处。  
同步方法示例：  

    class Program
    {
        static void Main(string[] args)
        {
            MyDownLoadString my=new MyDownLoadString();
            my.DoRun();

            Console.ReadKey();
        }
    }
    class MyDownLoadString
    {
        Stopwatch sw=new Stopwatch();

        public void DoRun()
        {
            const int largeNumber = 600000;
            sw.Start();
            int t1 = CountCharacters(1, "http://www.baidu.com");
            int t2 = CountCharacters(2, "https://www.jd.com");
            CountToAlrgeNumnber(1,largeNumber);
            CountToAlrgeNumnber(2,largeNumber);
            CountToAlrgeNumnber(3,largeNumber);
            CountToAlrgeNumnber(4,largeNumber);
            Console.WriteLine("Chars in http://www.baidu.com        :{0}",t1);
            Console.WriteLine("Chars in https://www.jd.com            :{0}", t2);
        }

        private int CountCharacters(int id, string uristring)
        {
            WebClient wc1=new WebClient();
            Console.WriteLine("String call {0}      ：   {1, 4} ms",id,sw.Elapsed.TotalMilliseconds);
            string result=wc1.DownloadString(new Uri(uristring));
            Console.WriteLine("call {0} complete：   {1, 4} ms", id, sw.Elapsed.TotalMilliseconds);
            return result.Length;
        }

        private void CountToAlrgeNumnber(int id,int largerNumber)
        {
            for (int i = 0; i < largerNumber; i++)
            {

            }
            Console.WriteLine("  End Counting{0}:   {1,4} ms",id,sw.Elapsed.TotalMilliseconds);
        }
    }  

运行结果：

![](http://i.imgur.com/PaRiPYH.png)

被调用的DoRun()方法的执行顺序是按照语句的时序进行的，在获取网址字符长度的时间内，程序进行等待并未做其他操作，整个程序执行完耗时为：779.6366ms

异步方法示例：
    
     static void Main(string[] args)
        {
            MyDownLoadStringAsync myAsync=new MyDownLoadStringAsync();
            myAsync.DoRun();

            Console.ReadKey();
        }    
    class MyDownLoadStringAsync
    {
        Stopwatch sw = new Stopwatch();

        public void DoRun()
        {
            const int largeNumber = 600000;
            sw.Start();
           Task <int> t1 = CountCharacters(1, "http://www.baidu.com");
           Task<int> t2 = CountCharacters(2, "https://www.jd.com");
          
            CountToAlrgeNumnber(1, largeNumber);
            CountToAlrgeNumnber(2, largeNumber);
            CountToAlrgeNumnber(3, largeNumber);
            CountToAlrgeNumnber(4, largeNumber);   
            Console.WriteLine("Chars in http://www.baidu.com        :{0}", t1.Result);
            Console.WriteLine("Chars in https://www.jd.com            :{0}", t2.Result);
        }

        private async  Task<int> CountCharacters(int id, string uristring)
        {
            WebClient wc1 = new WebClient();
            Console.WriteLine("String call {0}      ：   {1, 4} ms", id, sw.Elapsed.TotalMilliseconds);
            string result =  await  wc1.DownloadStringTaskAsync(new Uri(uristring));
            Console.WriteLine("call {0} complete：   {1, 4} ms", id, sw.Elapsed.TotalMilliseconds);
            return result.Length;
        }

        private void CountToAlrgeNumnber(int id, int largerNumber)
        {
            for (int i = 0; i < largerNumber; i++) ;
            Console.WriteLine("  End Counting{0}:   {1,4} ms", id, sw.Elapsed.TotalMilliseconds);
        }
    }   

运行结果：

![](http://i.imgur.com/gRD9Tea.png)

从运行结果可以看出，当调用DoRun()方法时，执行顺序并不是按照语句的时序进行的，当调用 
    
    Task<int> t1 = CountCharacters(1, "http://www.baidu.com");
    Task<int> t2 = CountCharacters(2, "https://www.jd.com");
方法时，在进入方法内部开始等待后，程序回到调用处DoRun()方法，利用等待的时间执行后续的

    CountToAlrgeNumnber(1, largeNumber);
    CountToAlrgeNumnber(2, largeNumber);
    CountToAlrgeNumnber(3, largeNumber);
    CountToAlrgeNumnber(4, largeNumber); 
输出操作，当等待完成后，输出获取的网址字符串长度，整个方法耗时574.4725ms，而同步方法的耗时是779.6336ms。可见，异步方法的效率更高。
##  ##
## async/await 特性 ##
C#中使用async/await特性创建异步方法。这个特性结构包括三个部分：  

- 调用方法  

         public void DoRun()
        {
        ...
        Task <int> t1 = CountCharacters(1, "http://www.baidu.com");
        Task<int> t2 = CountCharacters(2, "https://www.jd.com");
        ...
        } 
调用方法调用异步方法，在异步方法（可能在相同的线程，可能在新的线程）执行任务的时候继续执行自身任务。

- 异步方法
     
        private async  Task<int> CountCharacters(int id, string uristring)
        {
           ...
        string result =  await  wc1.DownloadStringTaskAsync(new Uri(uristring));
	    ...
        return result.Length;
        }
异步方法异步执行方法内的任务，进入到异步执行的任务处，立即返回到调用方法

- await 表达式  
await表达式指出需要异步执行的任务，异步方法也是到该处返回到调用方法，异步任务执行的时候，调用方法继续执行自身任务。异步方法中至少包含一个await表达式。

关于异步方法的说明：  


- 方法签名中包含***async***关键字   
***async***在方法返回值前，该关键字用以说明方法中包含至少一个await表达式，该关键字并不能创建任何异步任务。

- 异步方法具备三种返回类型: ***void***、***Task***、***Task*<T*>***
>
>- ***void*** 调用方法仅仅执行异步方法，不需要从异步方法中获取返回值，或是获取和操作异步方法的状态  
>
>         private async  void CountCharacters(int id, string uristring)
        {
           ...
        string result =  await  wc1.DownloadStringTaskAsync(new Uri(uristring));
	    ...
        }
>- ***task*** 调用方法执行异步方法，不需要从异步方法中获取执行完成后得结果，但需要获取和操作异步方法 的状态
>
>         private async  Task CountCharacters(int id, string uristring)
        {
           ...
        string result =  await  wc1.DownloadStringTaskAsync(new Uri(uristring));
	       ...
        }
        public DoRun(){
        Task task=CountCharacters(1,"www.baidu.com");
        task.Wait();
        }
>- ****Task*<T*>** 调用方法执行异步方法，并从异步方法中获取执行完成后得结果
>
>         private async  Task<int> CountCharacters(int id, string uristring)
        {
           ...
        string result =  await  wc1.DownloadStringTaskAsync(new Uri(uristring));
	       ...
        return result.Length;
         }
        public DoRun(){
        Task<int> taskValue=CountCharacters(1,"www.baidu.com");
        Console.WriteLine("The StringLength is {0}",taskValue);
        }

关于异步方法的返回值说明，异步方法在遇到**await表达式**时，就会返回异步方法的***返回类型**（***void***、***Task**、****Task*<T*>**）,这个返回类型与await语句的返回值没有关系，这个语句的返回值可以用来设置返回类型的属性，而异步方法末尾可能出现的return语句只是设置返回类型的属性（若为**Task*<T*>**，设置****Task*<T*>.Result**的属性）和标识异步方法的结束退出。     
##  ##
##调用方法与异步方法之间的控制流  ##
调用方法与异步方法之间的控制流图：  

![](http://i.imgur.com/u8jdkPp.png)  
异步方法中可能包含多个await语句，当最后一个await语句所创建的异步任务执行完毕，Task的属性和返回值设置完成，await语句后的代码在异步方法内同步执行完成，异步方法执行完成并退出。
##  ##
## await表达式 ##
异步方法中的await表达式指定一个异步任务。这个任务可能是一个Task类型的对象，也可能不是。***默认情况下，该任务在当前线程下异步运行***  

    await task;  
这个任务是awaitable类型的对象，awaitable类型是指实现了GetAwaiter()的方法的类型，GetAwaiter()方法返回一个awaiter类型对象,awaiter类型包括成员：  
>
- bool IsCompleted{get;}
- void OnCompleted(Action);
- void GetResult(); 或 T GetResult();  

在实际使用过程中，并不需要自己构建awaitable类型(就好比使用yield return构建枚举类型(Enumerable)和枚举器类型(Enumerator)一样)，Task类是awaitable类型，使用Task类的对象在await表达式中即可创建指定的异步任务。BCL中包含许多异步方法，这些异步方法可以返回Task*<T*>对象，与await表达式一起，在***当前线程异步执行***。比如webClient.DownLoadStringTaskAsync()方法：

      await  wc1.DownloadStringTaskAsync(new Uri(uristring));
BCL中包含许多异步方法，当我们需要创建自己的异步方法时，需要使用Task.Run()方法。
##  ##
## Task.Run()方法 ##
Task.Run（）方法可以创建异步方法，与await表达式在当前线程创建异步任务不同的是，**Task.Run()会在另一个线程上执行异步方法**  
Task.Run（）方法的参数为一个委托，该委托没有参数，有返回值，Task.Run（）方法的重载:  
>
    Task				Run(Action action);
    Task				Run(Action action,CancellationToken token);
    Task<TResult>		Run(Func<TResult> function);
	Task<TResult>		Run(Func<TResult> function,CancellationToken token);
	Task				Run(Func<Task> function);
	Task				Run(Func<Task> function,CancellationToken token);
	Task<TResult>		Run(Func<Task<TResult>> function);
	Task<TResult>		Run(Func<Task<TResult>> function,CancellationToken token);  

不同委托类型作为参数示例：  
>  
	static class MyClass{
	public static async Task DoWorkAsync(){
		await Task.Run(()=>Console.WriteLine(5.ToString()));
		Console.WriteLine(await Task.Run(()=>6).ToString());
		await Task.Run(()=>Task.Run(()=>Console.WriteLine(7.ToString())));	
		int value=await Task.Run(()=>Task.Run(=>8));
		Console.WriteLine(value.ToString());
	}
	} 
	class Program{
	static void Main(){
	Task t=MyClass.DoWorkAsync();
	t.Wait();
	Console.ReadKey();
	}
	}  

运行结果：
>
    5
	6
	7	
	8  

在await/async特性的用法上await 与一个Task类型的对象组合语句可以创建异步任务，这个Task对象可以是BCL中的异步方法的返回类型，也可以是使用Task.Run()方法创建的Task类型对象，区别在于，**由Task.Run()方法创建的异步任务是在一个新的线程中异步执行的。**
##  ##
## 异步方法的取消 ##
异步方法可以进行终止操作。异步方法的取消需要用到Task.Run()方法中的第二个参数类型——**CancellationToken**  
Cancellation是System.Threading.Tasks命名空间中的一个类，异步方法的终止还需要该命名空间下的另一个类——**CancellationTokenSource**  
>
	CancellationTokenSouce cts=new CancellationTokenSource();
	CancellationToken token=cts.Token;  



- CancellationTokenSource对象创建可以分配给不同任务的CancellationToken对象。持有CancellationTokenSource的对象可以调用其Cancel方法，将CancellationToken对象的IsCancellationRequested的值设置为True。
- CancellationToken对象包含一个任务是否被取消的信息。持有CancellationToken对象的任务需要定期检查其状态。如果CancellationToken对象的IsCancellationRequested的值被设置为True,任务需停止并退出。
- CancellationToken的IsCancellationRequested的值设置是**不可逆的，只有一次**，一旦该值为True,则不能更改了。  

值得注意的是，CancellationTokenSource.Cancel()方法的调用**并不会进行终止任务的操作**，只是给CancellationToken对象的IsCancellationRequested的值设置为True，真正的终止操作是有持有CancellationToken对象的任务在检查其IsCancellationRequested的值之后所做的终止并退出。  
示例代码：  
>
>     class Program
    {
        static void Main(string[] args)
        {
            CancellationTokenSource cts=new CancellationTokenSource();
            CancellationToken token = cts.Token;
            MyClass mc=new MyClass();
            Task t=mc.RunTask(token);
            Console.WriteLine("Async Action");
            Thread.Sleep(3000);
            cts.Cancel();
            t.Wait();
            Console.WriteLine("Was Canceled:{0}",token.IsCancellationRequested);
            Console.ReadKey();
        }
    }
    class MyClass
    {
        private Stopwatch sw=new Stopwatch();
        public async Task RunTask(CancellationToken ct)
        {
            if (ct.IsCancellationRequested)
                return;
            await Task.Run(()=>CycleMethod(ct),ct);
        }
        void CycleMethod(CancellationToken ct)
        {
            sw.Start();
            Console.WriteLine("Starting CycleMethod at time {0,4}",sw.Elapsed.TotalMilliseconds);
            const int max = 5;
            for (int i = 0; i < max; i++)
            {
                if (ct.IsCancellationRequested)
                    return;
                Thread.Sleep(1000);
                Console.WriteLine("{0} of {1} iterations completed at time {2,4}",i,max,sw.Elapsed.TotalMilliseconds);
            }
        }
    }  

运行结果：  
>![](http://i.imgur.com/Le3T1Rp.png)  

上述代码中注释以下两行：  
>
>       //Thread.Sleep(3000);   
      //cts.Cancel();  

运行结果：
>![](http://i.imgur.com/wRqrnhJ.png)  

当执行CancelationTokenSource.Cancel()后,由持有CancelationToken的RunTask任务检查IsCancelationRequested的属性值，此时为True，结束任务并退出。
##  ##
## 方法的等待 ##
方法的等待分为在调用方法中的等待和异步方法中的等待
### 调用方法中的同步等待 ###
调用方法内可以调用任意数量的异步方法，可以先执行其他任务，然后接收异步方法返回的Task对象，也可以在等待某一个异步方法执行完毕后，在进行后续任务的处理。可以通过Task的**实例方法task.Wait（）**对任务进行等待。task.Wait()用于单一Task对象，对多个Task对象的等待，Task类提供两个**静态方法**，**Task.WaitAll()**,**Task.WaitAny()**,这两个方法的区别在于**Task.WaitAll()**是在调用方法内等待所有的异步方法执行完毕后在执行后续的任务，而**Task.WaitAny()**则是在调用方法内至少等待某一个异步任务完成，然后执行后续的任务，不必等待所有的异步方法执行完毕。  
### 异步方法中的异步等待 ###
在异步方法的内部，可以通过await 表达式来等待异步方法中的任务，这时控制权会回到调用方法，但在异步方法内部可以等待一个或所有任务的完成。通过Task的静态方法**Task.WhenAny()**和**Task.WhenAll()**来实现。   
代码示例：
>    
    class Program
    {
        static void Main(string[] args)
        {
            MyDownloadString mc=new MyDownloadString();
            mc.DoRun();
            Console.ReadKey();
        }
    }
    class MyDownloadString
    {
        public void DoRun()
        {
            Task<int> t = CountCharactersAsync("http://www.baidu.com", "http://www.hao123.com");
            Console.WriteLine("DoRun: Task{0} Finished",t.IsCompleted?"":"NOT");
            Console.WriteLine("DoRun: Result={0}",t.Result);
        }
        private async Task<int> CountCharactersAsync(string str1,string str2)
        {
            WebClient wc1=new WebClient();
            WebClient wc2=new WebClient();
            Task<string> t1= wc1.DownloadStringTaskAsync(new Uri(str1));
            Task<string> t2 = wc2.DownloadStringTaskAsync(new Uri(str2));
            List<Task<string>> tasks=new List<Task<string>>();
            tasks.Add(t1);
            tasks.Add(t2);
            await Task.WhenAny(tasks);
            Console.WriteLine("     CCA:    t1 {0} Completed",t1.IsCompleted?"":"NOT");
            Console.WriteLine("     CCA:    t2 {0} Completed", t2.IsCompleted ? "" : "NOT");
            return t1.IsCompleted ? t1.Result.Length : t2.Result.Length;
        }
    }  

运行结果：
>![](http://i.imgur.com/2UM4Hh9.png)  

这里使用WhenAll()方法，当执行到异步方法中的await()后，回到调用方法输出，完成异步执行，在异步方法中的内部其中一个任务执行完毕后，就完成异步方法的调用并回到调用方法。这里将WhenAny改为WhenAll,结果为：  
>![](http://i.imgur.com/RKdx18v.png)

WhenAll()方法则是在异步方法内部等待所有的任务完成才退出回到调用方法。

----------
关于异步基础先总结到这里，还有BeginInvoke()和EndInvoke()以及等待——完成，轮询，回调等异步执行模式，有兴趣的朋友可以自己找找看  
参考**C#图解教程2012（第四版）**