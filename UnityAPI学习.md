## Unity API 学习 （部分）##
### Time时间类静态变量 ###

----------
    void Update () {
	     Debug.Log("Time.deltaTime: "+Time.deltaTime);
            Debug.Log("Time.fixedDeltaTime: " + Time.fixedDeltaTime);
            Debug.Log("Time.fixedTime: "+Time.fixedTime);
            Debug.Log("Time.frameCount: "+Time.frameCount);
            Debug.Log("Time.realtimeSinceStartUp: "+Time.realtimeSinceStartup);
            Debug.Log("Time.smoothDeltaTime: "+Time.smoothDeltaTime);
            Debug.Log("Time.time: "+Time.time);
            Debug.Log("Time.timeScale: "+Time.timeScale);
            Debug.Log("Time.timeSinceLevelLoad: "+Time.timeSinceLevelLoad);
            Debug.Log("Time.unscaledTime: "+Time.unscaledDeltaTime);
    }
其中，  
**Time.frameCount**为自运行游戏开始后总的运行帧数；    
**Time.realtimeSinceStartup**为自游戏运行开始后总时间，在暂停调试时该值也会进行计时,可以通过调用一个方法前后获取该值，输出得到的时间差进行一个方法的性能测试；     
**Time.timeSinceLevelLoad**在场景进行切换时，会重置计时；   
**Time.timeScale**常用于对游戏进行慢放和快放的操作，   **Time.timeScale**值为0时，场景中所有通过  **Time.deltaTime**进行的移动控制会被暂停    
### 游戏物体GameObject ###

----------
**创建游戏对象的三种方式：**    
    
- **GameObject**的构造方法       
 
        GameObject go=new GameObject("Cube");    
这种方法可以创建出名为“Cube”的空物体，
一般用来创建空物体并指定空物体的子对象；     
- **GameObject.Instantiate()**      
 
        GameObject go=GameObject.Instantiate(prefab);    
这种方法可以克隆出prefab的预制体或者场景中的非预制体，使用最为频繁；   
- **GameObject.CreatePrimitive()**      
 
        GameObject plane=GameObject.CreatePrimitive(PrimitiveType.Plane);
        GameObject cube=GameObject.CreatePrimitive(PrimitiveType.Cube);
这种方法可以创建出基本的3D对象，如Cube,Plane,Sphere等，参数为枚举类型；       

**关于变量：**   

- **activeInHierarchy**     
在场景中是否激活，GameObject.SetActive()进行设置，当activeInHierarchy为false游戏对象不会在场景中显示，所挂脚本的Update()函数不会执行，节约开销，该游戏对象依旧在内存中，可以随时设置为True;   

**成员方法：**    

- **BroadcastMessage**   

        target.BroadcastMessage("Attack",null,SendMessageOptions.DontRequireReceiver);    
广播消息，对该游戏对象以及该游戏对象的所有子对象调用对应方法名的方法，使用**BroadcastMessage**可以不必通过获取对应引用的方式进行方法的调用，降低耦合性；      
- **SendMessage**   

        target.SendMessage("Attack",null,SendMessageOptions.DontRequireReceiver);   
发送消息，对目标游戏对象调用对应方法名的方法；   
- **SendMessageUpwards**   

        target.SendMessageUpwards("Attack",null,SendMessageOptions.DontRequireReceiver);    
向上广播消息，对该游戏对象以及该游戏对象的所有父对象（父对象的父对象）调用对应方法名的方法，使用**SendMessageUpwards**可以不必通过获取对应引用的方式进行方法的调用，降低耦合性；      
### MonoBehaviour ###

----------
**[ExecuteInEditMode]特性**  
一般MonoBehaviour的实例是在Play模式下进行执行，对某一个MonoBehaviour的实例（某一个类）添加该特性，该脚本可以在编辑器模式下进行执行；
    
**成员方法：**    

- **Invoke**    
      
        Invoke("Method",5f);   
在对应等待时间后，调用指定的方法，参数为方法名的字符串和需要等待的时间；  

- **isInvoking**   

        bool res = IsInvoking("Method");   
查看对应方法是否在等待调用队列中；

- **InvokeRepeating**   
 
        InvokeRepeating("Method",5f,2f);    
在对应等待时间后，以对应时间间隔，重复调用指定的方法，参数为方法名的字符串和需要等待的时间，重复调用的时间间隔；   

- **CancelInvoke**  

        CancelInvoke("Method");    
取消在等待队列中指定的方法，若不传递参数，则会取消该脚本中所有等待队列中的方法；     

- **StartCoroutine**     
	    
        private IEnumerator coroutine;
        void Start()
        {
         print("Starting " + Time.time);
         coroutine = WaitAndPrint(2.0f);
         StartCoroutine(coroutine);
         print("Before WaitAndPrint Finishes " + Time.time);
        }       
	    private IEnumerator WaitAndPrint(float waitTime)
        {
        while (true)
        {
            yield return new WaitForSeconds(waitTime);
            print("WaitAndPrint " + Time.time);
        }
        }
开启一个协程，完成延时等待功能 ；  

- **StopCoutoutine**   

	    StopCoroutine(coroutine);       
停止一个正在运行的协程方法，协程的开启和关闭也可以通过传入方法名的字符串进行；
- **StopAllCoutoutines**   

	    StopAllCoroutines();       
停止所有正在进行的协程方法；  

### Mathf中的静态成员 ###

----------  
Mathf中的成员均为静态的，包括变量与方法；  

**变量：**  


- **Mathf.Deg2Rad与Mathf.Rad2Deg** 角度与弧度的相互转换    

        public float deg = 30.0F;
        void Start() {
        float rad = deg * Mathf.Deg2Rad;
        Debug.Log(deg + "degrees are equal to " + rad + " radians.");
        float deg = rad * Mathf.Rad2Deg;
        Debug.Log(rad + " radians are equal to " + deg + " degrees.");
        }    

**方法：**

- **Mathf.Ceil** 向上取整，返回float,**Mathf.Ceil2Int**向上取整，返回Int

        void Example()
        {
        // Prints 10
        Debug.Log(Mathf.Ceil(10.0F));
        // Prints 11
        Debug.Log(Mathf.Ceil(10.2F));
        // Prints 11
        Debug.Log(Mathf.Ceil(10.7F));
        // Prints -10
        Debug.Log(Mathf.Ceil(-10.0F));
        // Prints -10
        Debug.Log(Mathf.Ceil(-10.2F));
        // Prints -10
        Debug.Log(Mathf.Ceil(-10.7F));
        }    

- **Mathf.Floor** 向下取整，返回float,**Mathf.Floor2Int**向下取整，返回Int

        void Example()
        {
        // Prints 10
        Debug.Log(Mathf.Ceil(10.0F));
        // Prints 10
        Debug.Log(Mathf.Ceil(10.2F));
        // Prints 10
        Debug.Log(Mathf.Ceil(10.7F));
        // Prints -10
        Debug.Log(Mathf.Ceil(-10.0F));
        // Prints -11
        Debug.Log(Mathf.Ceil(-10.2F));
        // Prints -11
        Debug.Log(Mathf.Ceil(-10.7F));
        }    
- **Mathf.Clamp**夹紧取值

        public static float Clamp(float value, float min, float max);     
value值小于min,返回min，大于max，返回max，若在两者之间，则返回原值，一般用于对有取值范围的值进行限定，**Mathf.Clamp01**参数为一个float值，min为0，max为1；  
          
	    hp=Mathf.Clamp(hp,0,100);   

- **Mathf.ClosestPowerOfTwo**返回距离参数值最近的2的n次方；      

        //print  8;
        Debug.Log(Mathf.ClosestPowerOfTwo(7));
        //print 16;
        Debug.Log(Mathf.ClosestPowerOfTwo(15));
        //print 16;
        Debug.Log(Mathf.ClosestPowerOfTwo(21));

- **Mathf.Lerp**插值运算，返回插值结果，运用比较多，使一个值向另一个值靠近；   
 
        transform.position = new Vector3(Mathf.Lerp(minimum, maximum, t), 0, 0);    
在做插值靠近时，由于每次是按照比例进行插值，所以越靠近终点，变化会越慢；   


- **Mathf.MoveTowards**一个值按照固定值的增长朝另一个值靠近（如果传入的固定增长值的参数为负值，则会远离目标值），这种变化是匀速的；   

        currStrength = Mathf.MoveTowards(currStrength, maxStrength, recoveryRate * Time.deltaTime);

- **Mathf.Pingpong**返回一个0到Length值之间的值，默认起点值为0，通常用来使一个值来回运动；

        transform.position=new Vector3(transform.position.x,Mathf.PingPong(Time.time*pingpongSpeed,10),transform.position.z);       
一般在做运动变化时，传入的变化速度为Time.time的整数倍，当到达Length值后，会以变化速度朝0值运动；  

        5+Mathf.PingPong(Time.time*pingpongSpeed,5);    
通过给返回结果+value，得到初始值不为0的来回运动；     

### Input ###

----------

Input类中的成员均为静态成员

- **Input.GetAxis**与**Input.GetAxisRaw**  
**Input.GetAxis**与**Input.GetAixsRaw** 均为从键盘获得方向键的输入，返回值为-1~1的值，区别在于**Input.GetAxis** 的值在变化时，会有一个加速变化的过程，而**Input.GetAixsRaw** 则是按键后立即响应得到1或者-1的值，一般在做角色方向控制时，会使用**Input.GetAxis** 来模拟一个速度加速的过程；  

### Vector2 ###

----------

**成员变量：**  
- **normalized**  
vector2.normalized会返回vector2二维向量的单位向量，对原vector2向量的值无影响；
而**vector2.Normalize()**成员方法无返回值，而是将原有向量的值变为其单位向量的值；   

      void Start () {
	    vector2=new Vector2(2,2);
        Debug.Log("vector2.normalized: "+vector2.normalized);    //print(0.7,0.7);
        Debug.Log("befroe vector2.Normalize(): "+vector2);       //print(2,2);
        vector2.Normalize();
        Debug.Log("after vector2.Normalize():　"+vector2);       //print(0.7,0.7);
	}

**静态方法：**    

- **Vector2.ClampMagnitude**      
返回一个向量，若传入的向量的模长小于传入的长度值，则返回对应方向上的模长为传入的长度值的向量，否则返回该向量自身，对某一个向量的长度做限制；   
        
        public static Vector2 ClampMagnitude(Vector2 vector, float maxLength);       
- **Vector2.Reflect**       

	    public static Vector2 Reflect(Vector2 inDirection, Vector2 inNormal);
返回一个向量沿着某一个方向做反射后的得到的二维向量；      
		
	    vector2=new Vector2(1,2);
        Vector2 newVector=Vector2.Reflect(vector2,Vector2.right);
        Debug.Log("the vector off vector2 defined by Vector2.right as normal:  "+newVector);  
        //print(-1,2);      
- **Vector2.Lerp**  

	    public static Vector2 Lerp(Vector2 a, Vector2 b, float t);      
向量或点做插值运算，按比例进行插值运算；  
- **Vector2.MoveTowards**     

	    public static Vector2 MoveTowards(Vector2 current, Vector2 target, float maxDistanceDelta);  
当前向量或点向目标向量或点进行匀速靠近，得到的结果更新当前向量或点；    

### **Vector3** ###

----------

- **Vector3.Cross**    

		public static Vector3 Cross(Vector3 lhs, Vector3 rhs);
三维向量之间的x乘，得到的向量方向满足 左手定则，长度值为两个向量模长与向量夹角的sin值乘积；  

- **Vector3.Dot**    

        public static float Dot(Vector3 lhs, Vector3 rhs);
三维向量之间的点乘，得到的是一个值，大小为两个向量模长与向量夹角的cos值乘积；  

- **Vector3.Lerp**  

        public static Vector3 Lerp(Vector3 a, Vector3 b, float t);     
向量之间的线性插值，将a和b当做点来进行插值；

- **Vector3.Slerp**     

	    public static Vector3 Slerp(Vector3 a, Vector3 b, float t);  
向量之间的球形插值，这里将a和b看做是一个方向来进行插值；    

	    public Transform sunrise;  
   	    public Transform sunset;  
    	public float journeyTime = 1.0F;  
    	private float startTime;      
   	    void Start() {
          startTime = Time.time;
          }        
    	void Update() {
        Vector3 center = (sunrise.position + sunset.position) * 0.5F;
        center -= new Vector3(0, 1, 0);
        Vector3 riseRelCenter = sunrise.position - center;
        Vector3 setRelCenter = sunset.position - center;
        float fracComplete = (Time.time - startTime) / journeyTime;
        transform.position = Vector3.Slerp(riseRelCenter, setRelCenter, fracComplete);
        transform.position += center;
        }   

### Random ###

----------

Random随机，该类只有静态变量和方法   
**静态方法：**

- **Random.Range**

	    public static float Range(float min, float max); 
	    public static int Range(int min, int max);  
生成min与max之间的随机数，包含min不包含max；
- **Random.InitState** 
随机数种子，Random.Range()在调用时，会根据随机整数种子在内部通过一定的公式进行计算产生随机数，如果不设置随机数种子，系统会自动设置一个随机数种子，得到的随机数的序列实际是可以预测的，只有每次使用不同随机数种子，那么得到的随机数的序列就是一个不可预测的；    

        void Start () {
		Random.InitState(40);
	      }
	    void Update () {
	    if (Input.GetKeyDown(KeyCode.Space))
	    {
            Debug.Log(Random.Range(4,20));
	    }
	    }   
这里每次启动运行后所产生的随机数序列是一定的，因为设置的是固定种子，系统默认的设置与时间有关，因此默认不设置种子，生产随机数时的序列也是不确定的；    
- **Random.insideUnitCircle**     
随机产生一个在单位圆内的点或向量，用于产生二维随机点；

		transform.position = Random.insideUnitCircle * 5;     
- **Random.insideUnitSphere**  
随机产生一个在单位球内的点或向量，用于产生三维随机点；

		 transform.position = Random.insideUnitSphere * 5;   
- **Random.onUnitSphere**
随机产生一个在单位球表面上的点或向量，用于产生三维随机点； 

		GetComponent<Rigidbody>().velocity = Random.onUnitSphere * 10;    
### Quaternion###

----------

四元数，用来记录一个旋转；  
**成员变量：**  

- **Quaternion.eulerAngles**   
返回一个游戏对象的旋转所对应的欧拉角；    
**静态方法：**

- **Quaternion.Euler**     
将一个欧拉角的三维向量转化为一个四元数；   

		public Quaternion rotation = Quaternion.Euler(new Vector3(0, 30, 0));       
- **Quaternion.LookRotation**  
将游戏对象的正前方向与传入的方向保持一致所得到的旋转，传入的参数是一个Vector3的矢量，结果为一个四元数；   

		public static Quaternion LookRotation(Vector3 forward, Vector3 upwards = Vector3.up); 
示例：  
		
		Vector3 dir = target.position - player.position;    
	    dir.y = 0;  //使计算忽略两者的高度差，从而得到的旋转使player不产生弯腰和抬头
        player.rotation=Quaternion.LookRotation(dir);         
- **Quaternion.Lerp**   

		public static Quaternion Lerp(Quaternion a, Quaternion b, float t);     
对rotation进行线性插值，传入的参数为两个四元数，和插值比例，返回值为四元数；
- **Quaternion.Slerp**

		public static Quaternion Slerp(Quaternion a, Quaternion b, float t);      
对rotation进行球形插值，相比于线性插值，计算耗时会长一点，但是效果会比线性插值更自然，传入的参数为两个四元数，和插值比例，返回值为四元数；    

	    Vector3 dir = target.position - player.position;
        dir.y = 0;
        Quaternion targetRotation = Quaternion.LookRotation(dir);
        player.rotation = Quaternion.Slerp(player.rotation, targetRotation, Time.deltaTime);     
### Rigidbody 刚体组件###

----------
**成员变量：**    

- **rigidbody.position**  
通过给rigidbody.position赋值，来改变刚体组件所在的游戏对象的位置，使用这种方式会比Update中使用transform.position的速度更快；  

		GetComponent<Rigidbody>().position = Vector3.zero;     
**成员方法：**    

- **rigidbodg.MovePosition**   
通过调用rigidbody.MovePosition()方法，来连续改变刚体组件所在的游戏对象的位置，操作会比transform.Translate（）更快，一般在Fixedupdate中；      	
		
		public void MovePosition(Vector3 position);     
示例：     

		rb.MovePosition(transform.position + transform.forward * Time.deltaTime);  
- **rigidbody.MoveRotation**  
与 MovePosition类似，通过调用rigidbody.MoveRotation()方法，来连续改变刚体组件所在的游戏对象的旋转，操作会比transform.Rotate（）更快，一般在Fixedupdate中,传入的参数为目标旋转值，为一个四元数；    
    	
		void FixedUpdate() {
        Quaternion deltaRotation = Quaternion.Euler(eulerAngleVelocity * Time.deltaTime);
        rb.MoveRotation(rb.rotation * deltaRotation);
        }   

### Camera ###

----------
**静态变量：**

- **Camera.main**   
获得场景中Tag为MainCamera的相机游戏对象的Camera组件；  
**成员方法：**   
- **camera.ScreenPointToRay()**  
从屏幕位置点发出一条射线；    

		Ray ray = Camera.main.ScreenPointToRay(Input.mousePosition);
	    RaycastHit hit;
	    bool isHit = Physics.Raycast(ray, out hit);
	    if (isHit)
	    {
            Debug.Log(hit.collider.name);
	    }      
### Application ###

----------
**静态方法：**  

- **Application.dadapath**   
返回工程文件路径的文件夹字符串；

- **Application.StreamingAssetsPath**
返回StreamingAssetsPath的文件路径字符串，在Unity中，一旦Build工程文件后，工程内的文件会被合并打包，而在Unity下建立StreamingAssetsPath的文件夹，文件夹内的流文件（音频，视频）在导出后不会被打包，可以通过外部查看文件夹的方式查看该文件夹内的流文件；  

- **Application.OpenURL**  
通过该方法打开一个网站；  

		 Application.OpenURL("http://unity3d.com/");   
- **Application.CaptureScreenShot**  
保存当前屏幕截图，传入的参数为截图文件名的字符串；  

		Application.CaptureScreenshot("Screenshot.png");    
### SceneManager ###

----------
**静态方法：**   

- **SceneManager.LoadManager**  
加载场景所调用的方法，传入的参数可以是场景的索引，也可以是场景名的字符串；第二个参数为可选参数，参数为场景加载模式。默认是Single,加载新的场景时，上一个场景内的游戏对象会被清空，可以设置为合并模式，即加载新的场景会将旧场景内的游戏对象与新场景进行合并；  

		public static void LoadScene(int sceneBuildIndex, SceneManagement.LoadSceneMode mode = LoadSceneMode.Single);

- **SceneManager.LoadManageAsyncr**  
异步加载新的场景，返回一个AsyncOperation,通过AsyncOperation对象可以得到加载的进度；  

		public static AsyncOperation LoadSceneAsync(string sceneName, SceneManagement.LoadSceneMode mode = LoadSceneMode.Single);
 





  
