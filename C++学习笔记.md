# C++ 学习笔记    
## 智能指针 
- shared_ptr 定义  
shared_ptr是一种 智能指针（ smart pointer）。shared_ptr的作用有如内 指针，但会记录有多少个tr1::shared_ptrs共同指向一个对象。这便是所谓的 引用计数（reference counting）。一旦最后一个这样的指针被销毁，也就是一旦某个对象的引用计数变为0，这个对象会被自动删除。这在非环形数据结构中防止资源泄露很有帮助。
————————————————    
[详解链接](https://blog.csdn.net/shaosunrise/article/details/85228823)