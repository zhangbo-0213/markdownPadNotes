# C++ 学习笔记    
## 智能指针 
- shared_ptr 定义    
shared_ptr是一种 智能指针（ smart pointer）。shared_ptr的作用有如内 指针，但会记录有多少个tr1::shared_ptrs共同指向一个对象。这便是所谓的 引用计数（reference counting）。一旦最后一个这样的指针被销毁，也就是一旦某个对象的引用计数变为0，这个对象会被自动删除。这在非环形数据结构中防止资源泄露很有帮助。  

- [详解链接](https://blog.csdn.net/shaosunrise/article/details/85228823)     

- unique_ptr 定义   

unique_ptr的直观认知就是“独占”。它的意思就是某个时刻只能有一个unique_ptr指向一个给定的对象，即不能拷贝和赋值

```
int* p (new int(3));
shared_ptr<int> p1(p);
auto p2 = p1;　//ok
unique_ptr<int> p3(p);
unique_ptr<int> p4(p); //ok，智能指针毕竟只是一个帮助你管理的一个工具，它并不能知道有多少初始化的动作
unique_ptr<int> p5 = p3; //error
```
- 初始化　 
unique_ptr没有类似make_shared的操作，只能直接初始化
```
unique_ptr<int> p(new int(1024)); //ok
unique_ptr<int> p1 = new int; //error
unique_ptr<int> p2(p); //error 使用一个unique_ptr初始化另一个unique_ptr,相当于做拷贝操作，错误
```
- unique_ptr基本操作

unique_pre<T> p1; //空unique_ptr，可以指向类型为T的对象，p1会使用delete来释放它的指针，p2会使用一个类型为D的可调用对象来释放它的指针　　

unique_ptr<T, D> p2; 
unique_ptr<T, D> p(d); //空unique_ptr，指向类型为T的对象，用类型为D的对象d来代替delete

```
p = nullptr; //释放p指向的对象，将p置为空
p.release(); //p放弃对指针的控制权，返回指针，并将p置空
p.reset(); //释放p所指向的对象
p.reset(q); //如果提供了内置指针q，令p指向这个对象；
p.reset(nullptr); 
``` 

虽然我们不能拷贝和赋值，但我们可以调用上述所说的reset和release将指针的所有权从一个（非const）unqiue_ptr转移给另一个unique_ptr
```
//将所有权从p1转移给p2
unique_ptr<string> p2(p1.release());
p2.reset(p1.release());
```
单纯调用release是错误的  
```
p.release(); //错误，release放弃了控制权不会释放内存，丢失了指针
auto q = p.release();　//正确，记得delete掉q
```
特殊的版本   
```
unique_ptr针对new出来的数组特殊化，是一个特殊化的版本
unique_ptr<int []> q(new int[10]);
q.release();
//自动用delete[]销毁其指针释放内存
```

unique_ptr作为参数传递和返回值     
unique_ptr的不能拷贝有一个例外：  
```
unique_ptr<int> Fun(int p) 
{
    return unique_ptr<int>(new int(p));
}
//返回一个局部对象的拷贝
unique_ptr<int> Fun(int p) 
{
    unique_ptr<int> ret(new int(p));
    //...
    return ret;
}
```
编译器知道要返回的对象将要被销毁，执行了一种特殊“拷贝”（移动操作）    
[详解链接](https://blog.csdn.net/weixin_36888577/article/details/80188414)  

---  

## 左值引用与右值引用     
1.左值和右值

在C++11中可以取地址的、有名字的就是左值，反之，不能取地址的、没有名字的就是右值（将亡值或纯右值）。

举个例子，int a = b+c, a 就是左值，其有变量名为a，通过&a可以获取该变量的地址；  
表达式b+c、函数int func()的返回值是右值，在其被赋值给某一变量前，我们不能通过变量名找到它，＆(b+c)这样的操作则不会通过编译。

左值是可以放在赋值号左边可以被赋值的值；左值必须要在内存中有实体；

右值当在赋值号右边取出值赋给其他变量的值；右值可以在内存也可以在CPU寄存器。

一个对象被用作右值时，使用的是它的内容(值)，被当作左值时，使用的是它的地址。

2.左值引用

左值引用就是我们平常使用的“引用”。引用是为对象起的别名，必须被初始化，与变量绑定到一起，且将一直绑定在一起。

我们通过 & 来获得左值引用，
type &引用名 = 左值表达式；
可以把引用绑定到一个左值上，而不能绑定到要求转换的表达式、字面常量或是返回右值的表达式。举个例子：

```
int i = 42;
int &r = i;    //正确，左值引用
int &r1 = i * 42;   //错误， i*42是一个右值
const int &r2 = i * 42; //正确，可以将一个const的引用绑定到一个右值上
```

3.右值引用

右值引用是C++11中引入的新特性 , 它实现了转移语义和精确传递。

它的主要目的有两个方面：

消除两个对象交互时不必要的对象拷贝，节省运算存储资源，提高效率。   
能够更简洁明确地定义泛型函数。   
右值引用就是必须绑定到右值的引用，他有着与左值引用完全相反的绑定特性，我们通过 && 来获得右值引用。

右值引用的基本语法type &&引用名 = 右值表达式；

右值有一个重要的性质——只能绑定到一个将要销毁的对象上。举个例子：
```
int  &&rr = i;  //错误，i是一个变量，变量都是左值
int &&rr1 = i *42;  //正确，i*42是一个右值
``` 

4、右值引用和左值引用的区别   

左值可以寻址，而右值不可以。
左值可以被赋值，右值不可以被赋值，可以用来给左值赋值。
左值可变,右值不可变（仅对基础类型适用，用户自定义类型右值引用可以通过成员函数改变）。

---     

## Vector.size() 使用问题    
vector 的size函数返回vector集合内元素的数量，返回值类型为size_type，Member type size_type is an unsigned integral type，即无符号整数；

vector A;

A.size()-1因为size返回值是无符号类型所以 A.size()-1越界，是个很大的数
所以要用的时候
要么预先定义一个 int n = A.size() - 1;
或者int(A.size() - 1)或者int(A.size()) - 1
或者static_cast<int>(A.size()) 

## 命名空间内函数指针成员的使用  
函数指针可以作为命名空间内的成员，当命名空间内的类不想通过头文件暴露出去，而外部需要获取这个类内部信息时，可以通过命名空间内 函数指针的方式，获取未暴露在头文件中的类的内部信息   
``` C++
namespace utils {
using FilamentErrorLogCallBack = std::function<void(const char*)>;
static FilamentErrorLogCallBack filamentErrorLogCallBack;
} 
```  
```  C++
namespace utils {
namespace io {
//初始化 static 函数指针
FilamentErrorLogCallBack filamentErrorLogCallBack = nullptr;
class LogStream : public ostream {
public:
    ostream& flush() noexcept override;
};

ostream& LogStream::flush() noexcept {
    std::lock_guard lock(mImpl->mLock);
    Buffer& buf = getBuffer();
    if(filamentErrorLogCallBack != nullptr)
        {
            filamentErrorLogCallBack(buf.get());
        }
    }
}
}
```  
外部使用：  
```  C++  
namespace utils {
namespace  io {
FilamentErrorLogCallBack filamentErrorLogCallBack = [](const char * info){Printf(info);};
}
}
```  

## 使用位操作符进行色值转换   
0xffffff 十六进制的颜色值共有24位，使用左移操作符进行快捷转换  
``` C++ 
unsigned long createRGB(float r, float g, float b)
{
  return ((((int)r * 255) & 0xff) << 16) + (((int)(g * 255) & 0xff) << 8) + (((int)(b * 255) & 0xff));
}
```
