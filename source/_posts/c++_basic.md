---
title: c++基础
date: 2021-11-09 01:15:37
tags: c++
---

##### 编译流程
txt -> compiling + linking -> exe

compiling  
* c++编译器把文本变成终极格式 -obj     
* pre-process  评估preprocessor语句
* tokenizing 标记解释
* parsing	解析  抽象语法树  -  解释成编译器能懂和处理的语言				          
* 编译器把代码转化成const data 或  instructions 
* include   -  只是把 .h文件里面的代码copy过来  例如 xx.h只有个 } 然后在main最后一行 用 #include "xx.h" 把}补进来
* #define INTEGER int -  在main可以  INTEGER main(){}  只是替换 
* #if 0 main{}  #endif  根据特定条件包含或剔除代码  就不会执行了
* obj文件里面是二进制  cpu的机器码  可以转为asm可读-汇编指令
* #progma once 只 include一次  和老的写法 { #ifndef  #define  #endif } 一样
* c标准库头文件一般有.h扩展文件  而c++没得

linker 把cpp编译好的obj串成exe可执行文件 
* 找到每个符号和函数的位置 链接在一起
* link error code和compiler error code都不一样的  容易区分
* 如果找不到一模一样的 就会报错  相同的名字和签名-返回值和参数相同
* static的函数 被include 只会在内部有效 
* inline的函数 把函数的身体拿过来调用 和copy类似 

static linking  &  dynamic libraries 
static发生在编译期间  dynamic发生在runtime  	 lib是编译时用到的，dll是运行时用到的

Precompiled Headers   预编译 头文件 改动东西时  编译会很快 尤其时大型项目  二进制  本质是包含一堆其他头文件的头文件 
					  针对那些不经常改变的东西  用预编译 头文件 

* * *
##### 数据类型
内存  堆栈
char 1byte 
short 2byte 
int 4byte 
long 4byte
long long 8byte
float  4byte   5.5f
double 8byte	5.5
bool   1byte   无法创建1bit的变量类型 内存没法寻址  1byte=8bit
数据类型多大根据编译器来的 用sizeof操作符来检查下  返回byte个数 
为了不搞多个函数 用inline 不用跳内存 push堆栈之类的  嵌套越多 就越慢
```c++
Casting		type casting 类型转换   隐式转换  例如  int a = (int) 1.34f;   
									静态转换  stataic_cast<int>(value)  
									动态转换  dynamic_cast<Derived*>(base)
动态转换  dynamic_cast 如果有效 则返回指针值 否则null   
		RTTI 运行时类型信息 让程序在运行时能根据基类的指针或引用来获得该指针或引用所指的对象的实际类型
		可以设置编译器  把这个关掉  再去编译 会得到warning信息  

strings faster  性能  处理字符串更快		void printName(const std::string& name)
				string视图 对现有内存 开启一个小窗口  没新对象产生   
					std::string name = "yan chernikov"; 
					std::string_view firstname(name.c_str(), 3);
					std::string_view lastname(name.c_str()+4, 3);
```
std::string 会自动扩容  初始化是8个字节  
```c++
DATA STRUCTURES 	   arrays,lists,sets,maps,trees
				int array[5];   int* heapArray = new int[5];    std::array<int,5> arrays;  class Array{  int* m_Data;  m_Data=(int*)alloca(size); }
                
ITERATORS     for(int i=0; i<arr.size(); i++)    for(int item : arr)     for(std::vector<int>::iterator it=arr.begin(); it!=arr.end(); it++)
			  include <unordered_map>	->   std::unordered_map<std::string, int> map; 

自己写一个vector类  不复制 直接 move push  pop  clear  delete  


```

* * *
##### 指针
计算机操作的本质是对内存的操作  访问内存可以是地址，起名字或者起小名
reference别名 int a=5; int &ref =a; 那么ref就是a的别名
void plus(int& a) 不会传参而是传内存地址   比指针要好看和简洁些

raw pointer   原始指针   smart pointer 智能指针
一个指针只是地址  整数 存储内存地址  
char* buffer = new char[8]
char** ptr = &buffer
那么 ptr是内存地址  指向buffer的内存地址  然后通过指向buffer的内存地址找到buffer
address ->  address -> value

```c++
智能指针	std::unique_ptr, std::shared_ptr, std::weak_ptr
		自动申请和释放内存  不要new和delete   超出scope就释放
		#include <memory> 
		std::unique_ptr<Eneity> eneity(new Eneity());  唯一指针 在scope后就会自动销毁
		std::shared_ptr<Eneity> eneity = std::make_shared<Eneity>(); // 引用计数  没用引用就释放  可以复制 共同指向
		std::weak_ptr<Eneity> e0;   e0 = entity;  如果离开了entity的scope  e0也会释放  弱引用        
```

```c++
lambdas	 和函数指针搭配食用    https://en.cppreference.com/w/cpp/language/lambda
		  auto lambda = [](int value) {std::cout<<"value"<<value<<std::endl; };
		  forEach(values, lambda);
		  
		  使用lambda之外的值  四种方式 =  &  a  &a  
		  void forEach(const std::vector<int>& values, const std::function<void(int)>& func){}
		  int a = 5;
		  auto lambda = [=](int value) mutable {a=5; std::cout<<"value"<<a<<std::endl;};
		  forEach(values, lambda)
		  
		  类似于rxjava里面的filter
		  auto it = std::find_if(values.begin(), values.end(), [](int value){return value>3;});
		  
		 
funtion pointer  函数指针    和python一样  高阶函数  把函数当作参数变量
		定义函数  void HelloWorld(int a) { //打印逻辑 }
				
		使用指针 typedef void(*hellofunction)(int);
				 HelloFunction fu = HelloWorld;
				 fu(8);
		定义高阶函数  void forEach(const std::vector<int>& values, void(*func)(int))
					  {  for (int item : values)  func(item); }
		使用 		std::vector<int> values = {1,2,3,4,5,6} 
					forEach(values, HelloWorld)
					或者内部函数写法  forEach(values, [](int value){ std::cout<<"value"<<std::endl; })
                    
Union  类似于别名    在Vector4，去改变x就相当于改变a 
Vector2{
	float x,y;
};
struct Vector4
{
	union{
		struct{
			float x,y,z,w;
		};
		struct{
			Vector2 a,b;
		};
	}
}
```

* * *

##### OOP
OOP   面向对象  --  面向过程，基于对象，面向对象，泛型编程
extern  在外部寻找 
static 只执行初始化一次 只会在你声明的那个c++文件是可见的  extern 外部声明也不可以 

构造函数 析构函数  Eneity() +   ~Eneity() =>  Eneity e;  
继承: : public Eneity  
virtual fun虚函数:  允许覆盖子类中的方法   如果要想实现多态 父类必须是虚函数 子类才能覆盖
 俩个时间损耗 
1.需要额外的内存将信息存储在虚函数表中
 2.每次调用需要遍历虚函数表 查看映射的函数 
现在允许加个 override  std::getName() override {return m_name; }

pure virtual fun 纯虚函数   像java的interface   virtuan std::string getName() = 0; 

```c++
Implicit Conversion  隐式转换   explicit 关键字  构造函数 
	隐式转换  构造函数 Entity(const std::string& name)
							: m_name(name), m_age(-1) {}
					   Entity(int age)
							: m_name("unknow"), m_age(age) {}
	调用的时候  直接      Entity a = "CHERNO"; 或者  Entity b = 22;
	这会去调用构造函数 和 Entity a = Entity("CHERNO"); 或者  Entity b = Entity(22); 一样的效果
	explicit 关键字 就是为了禁止这种奇奇怪怪的东西出现  例
			explicit Entity(const std::string& name)
						 : m_name(name), m_age(-1) {}
			那么 就不能像上面那样玩了   必须 Entity("cherno"); 去实例化对象  

initializer list：  性能优化 不会创建新的对象 直接赋值 在构造函数中使用  按顺序来 
					 Entity()
						  : m_name("Unknow"), m_score(0)
					 {}
                     
对象生命周期 
	 int main(){
		{
			Eneity e; //执行构造函数					如果  Entity* e=new Entity(); 
		}    		  //执行析构函数  free memory		// 不会释放 在堆上面创建的实例
		return 0;
	 }

this	当前对象实例的指针

箭头  Entity e; e.print();  but  Entity* ptr = &e; ptr->print();
	  操作符重载  更酷的代码    
       
Virtual Destructors	 虚析构函数	  父类base 子类derived  
					 如果创建base    父类构造函数 -> 父类析构函数
					 如果创建derived 父类构造函数  子类构造函数 ->  子类析构函数 父类析构函数
					 如果用多态 	 父类构造函数  子类构造函数 ->  父类析构函数  		======= 不会去执行子类析构函数 内存泄漏maybe
									 在父类析构函数前面加上  virtual 关键字  意味着有可能被扩展  需要调用扩展者的析构函数 
                                     
singletones    在C++中只是将一堆全局变量和静态函数组织起来的方法   牛逼 比java灵活
				Singleton(const Singleton&) = delete; 删除掉默认的构造函数                                 
```
* * *
##### 函数
```c++
函数多个返回值  	tuple  		 pair   
			#include <itility>  #include <functional>  函数返回类型=std::tuple<std::string, std::string>   return std::make_pair(s1,s2);
			或者 struct  作为函数 返回多个值  
			或者对象  也可以 
            
STRUCTURED BINDINGS  结构化绑定声明  在一次声明中同时引入多个变量，同时绑定初始化表达式的各个子对象的语法形式。
					 用tuple  返回多个值 
```
* * *
##### 数组和字符串
```c++
string：  传统的char需要以0结尾  才能打印  char name[5] = {'n', 'a', 'm', 'e', 0}; 
		  用string库   std::string name = "name";
		  用stdlib.h   char=u8""  wchar_t=L""  char16_t=u""  char32_t=U""    std::u32string   R""  
		  字符串拼接 "cherno"s + "hello"  或者  std::string("cherno") + "hello" 
          
array:  创建数组
			int arr1[5];     
			int* arr2 = new int[5];   delete[] arr2;
			std::array<int,5> arr3;    #include <array>
            
动态数组 std::vector  不指定大小  会自动扩容  在堆上 
		#include <vector>  std::vector<Entity> arrs;
		arrs.push_back    for(Entity item : arrs) {}   arrs.erase(2)		
		arrs.emplace_back() 不要复制 
        
sorting		std::sort  
			std::vector<int> values = {3, 5, 1, 4, 2};
			std::sort(values.begin(), values.end());
			std::sort(values.begin(), values.end(), [](int a, int b) {
				return a > b;
			});

Multidimensional Arrays  多维数组 
			考虑内存优化  分配一块紧凑的  设置宽度去赋值  会很快  
            
```

* * *
##### 操作内存

```c++
创建对象  在栈和堆的区别  
		在堆上面   Entity* entity = new Eneity("Cherno"); 必须释放 delete entity;
		在栈  Entity e; 默认调用了构造函数  在函数中  生命周期就是函数执行完成的时候
        
new   在堆上面分配内存  要手动释放	
	Entity* e = new Eneity();    调用了构造函数
	Entity* e = (Entity*)malloc(siizeof(Entity));  只是存粹的分配内存

copy 复制  c++里面的复制和java相反  int a=2; int b=a; 其实b会copy到a  改变b 不会影响a
	  用指针的话 就不一样  在堆  指向的是内存  
	  memcpy() 复制字符串 
	  在对象里面  用运算符重载  真的好看  例如  char& operator[](unsigned int index){ return m_buffer[index]; }
	  真正的深拷贝  靠构造函数来  重新在堆上分配内存  复制 memcpy
	  函数传递引用  不能改变值  void print(const String& string) 
```
```c++
Move Semantics		移动语义	当一个函数的参数按值传递时，这就会进行拷贝 
					std::move、std::forward  避免copy的一种手段    避免内存的重新分配
					右值引用提供了一个暂时的对象 我们可以进行移动  右值引用和移动语义允许我们避免不必要的拷贝
					要移动而不是拷贝右值参数的内容。这就会节省很多的空间
								int a =10;
								int&  lr = a;       // a 是个变量
								int&& rv = 10;      // 10 是常量	
					String string = "hello";
					String dest((String&&)string);   //move string to dest  no copy 
					String dest( std::move(string)); //move string to dest  no copy 
					String dest = std::move(string); //move string to dest  no copy 
```
* * *
##### 关键字
```c++
mutable:  俩种用途   一种是 const  另一种是 lambda    英文  sth changeable
		  用const修饰的函数 不能改东西  但想改  就用mutable去修饰需要改变的变量
	int x = 8;
    auto f = [=]() mutable   // [& =]  ->  {&传递地址}  {=复制值不改变原来的值}
    {
        x++;
        std::cout<< x << std::endl;
    };
    //x = 8  just copy not reference passing  只是复制 会重新new一个 没传递地址 不会改变原值

const： 修饰变量  全局定义 或者函数传参  sth不变   const int MAX_AGE = 120;  
		修饰函数 放在后面 里面的东西不能改变  但是可以靠mutable来改变
clase Entity{
private:
	int m_x, m_y;
	mutable int var;
public:
		int getX() const {
		  var =2;   //这里其实不允许改变的  可以用mutable去修饰var 
		  return m_x; 
	    }
}

内存自己操作  操作符operator自定义  灵活

运算符重载    bool operator==const(Vector2& other) const    俩个类相等
				{
					return x==other.x && y=other.y;
				}
	使用:  Vector2 result1; Vector2 result2;  if(result1==result2) { } 
    

Track MEMORY ALLOCATIONS  就是重载关键字  然后把里面的东西弄出来  强的一批  和java反射一样 
		void* operator new(size_t size)
		{
			std::cout<<"allocating "<<size<<" bytes\n";
			return malloc(size);
		}

		void operator delete(void* memory, size_t size)
		{
			std::cout<<"freeing "<<size<<" bytes\n";
			free(memory);
		}
        

auto   自动推导类型  如果是string int 基础 不要用  复杂的 写起来很长的就用 代码非常简洁  例如array迭代器 
		std::vector<std::string>::iterator it = strings::begin();    =>  auto it = strings.begin();


Macros  宏定义 define  替换字符串   就是替换    用 \ 换行
		#define LOG(x) std::cout<<x<<std::endl   LOG("HELLO");
		还能用#if  #else #endif 去做条件适配

 
template   就是泛型  	Array<std::string, 5> array;  Array<int, 5> array;

						template<typename T, int N>
						class Array
						{
						private:
							T m_array[N];
						public:
							int get_size() const {return N;}
						};
                        
Timing		#include <cherno>    std::chrono::high_resolution_clock::now()
			统计函数执行时间  用struct放start和end  构造函数初始化start 在析构函数初始化end 打印耗时 
							  在需要统计的函数 创建Timer  因为是栈 所以脱离scope就会释放 妙啊 这个设计 


Thread  线程 	#include <thread>	std::thread worker(hello);  worker.join(); 
				void hello(){ while(true){ std::cout<<"hello"<<std::endl;} }             
```
* * *
##### 新特征
```c++
#include <optional>  c++17新特征   
					std::optional<std::string> ReadFile(const std::string& filepath)
					{
						std::ifstream stream(filepath);
						if(stream)
						{
							std::string result;
							//read fil
							stream.close();
							return result;
						}
						return {};
					}
					使用:
					std::optional<std::string> data = ReadFile("");
					if(data.has_value()){  }    就能知道文件是否存在 或者打开过 
                    
Multiple TYPES of Data in a SINGLE VARIABLE  一个变量保存多个类型数据  c++17新特征 
					#include <variant>   std::variant<std::string, int> data;
										 data = "cherno"; data=2;
										 std::get<std::string>(data)   or std::get<int>(data)                    
store ANY data     存储任何数据   #include <any>   std::any   c++17新特征 



```
* * *
##### 其他
using namespace   最好不用  难调试  特别是大型的项目 
Type Punning	类型双关	int a=50;  double value = *(double*)&a;  奇怪的用法  最好别碰 
断点 有条件的断点  
BENCHMARKING 工具  测试性能  用之前 类去做  构造函数开始  析构函数结束  用scope {}
google chrome  tracing  https://www.chromium.org/developers/how-tos/trace-event-profiling-tool

```c++
std::asyn		并发  多线程  #include <future>   std::async( std::launch::async, loadmesh(meshes,filepath); )
				锁  std::mutex  s_mutex;    std::lock_guard<std::mutex> lock(s_mutex);  
```
lvalues and rvalues	  int i=10;  i=lvalue  10=rvalue   i可以改  10不能改 
Argument Evaluation Order  参数传递顺序 	

Static Analysis   code review 分析哪里写的有问题    用工具  VS自带的也有 

持续集成（Continuous Integration）	Jenkins	  不管是哪一个持续集成工具，它本质上只不过是一个定时器，时间一到，做你脚本里让它去做的事

看别人的开源代码  然后吸收营养  直到自己也能写出那样骚的代码   用静态分析工具  找到哪些不合理的地方  然后改进

[youtube](https://www.youtube.com/playlist?list=PLlrATfBNZ98dudnM48yfGUldqGD0S4FFb)

[参考手册](https://zh.cppreference.com/w/%E9%A6%96%E9%A1%B5)

[编译器](https://wandbox.org/)

[书籍](https://github.com/ShujiaHuang/Cpp-Primer-Plus-6th)