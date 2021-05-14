# C + Plus

## 指针

### const与指针

指向常量的指针 (指针是low-level const的)

```c++
const int a = 1;
const int *ptr = &a;
```

常量指针 (指针是top-level const的)

```c++
int a = 1;
int *const const_ptr = &a; // ptr points to the object 'a' forever

//可以用常量表达式声明定义一个常量指针（注意常量表达式的值需要编译时就得到计算）
constexpr int *p = nullptr;
```

一个指针可以同时是顶层const和底层cosnt的，这种指针称为“指向常量的常量指针”



## 处理类型

### 类型别名

```c++
typedef double wages; //wages是double的同义词
wages h; //等价于double h;

using S = string;//c++11新特性，别名声明
S sentence; //等价于string sentence;
```

<img src="C:\Users\misaki\AppData\Roaming\Typora\typora-user-images\image-20210105002819336.png" alt="image-20210105002819336" align="left" />



### auto类型说明符

通过**初始值**推断出变量的类型

<img src="C:\Users\misaki\AppData\Roaming\Typora\typora-user-images\image-20210105003450780.png" alt="image-20210105003450780" align="left" style="zoom:100%;" />

<img src="C:\Users\misaki\AppData\Roaming\Typora\typora-user-images\image-20210105003614776.png" alt="image-20210105003614776" align="left" />

i 和 \*p 的类型都是int，所以正确（*注意：一个是const int一个是int类型的也不允许！*）



auto会忽略掉顶层const，但是保留底层const

```c++
const int ci = i, &cr = ci;
auto c = cr; //c是一个整数（cr是ci的别名，ci本身是一个顶层const，所以忽略const）
auto d = i; //d是一个整数（整数的地址就是指向整数的指针）
auto e = &ci; //e是一个指向整数常量的指针（对常量对象取地址是一种底层const，意思是&ci是一个底层const指针）

//明确指出希望推断出的类型是顶层const的
const auto f = ci;//ci的推演类型是int，f是const int

auto &g = ci; //g是一个const int引用，绑定到ci
```



### decltype类型指示符

分析表达式并得到它的类型，但不计算它的值

```c++
decltype(f()) sum = x; //sum的类型是函数f的返回类型

const int ci = 0, &cj = ci;
decltype(ci) x = 0; //x的类型是const int
decltype(cj) y = x; //y的类型是const int&，y绑定到变量x
```



decltype( (variable) )的结果永远是引用





## STL

### 迭代器失效

暂记：可以把迭代器看作一种智能的指针，对容器的增删引起所指地址存储的元素发生变化（原来的元素被删除或移动），而引起迭代器失效



#### vector和string的迭代器失效

##### 添加元素引发的迭代器失效

1. 如果向vector中添加元素引起了存储空间的重新分配（容器大小不足，重新分配内存，cp原数组到新的内存），则迭代器全部失效
2. 如果存储空间未重新分配，则只有插入位置后的迭代器失效

##### 删除元素引起的迭代器失效

1. 被删除元素及其位置后的迭代器全部失效



#### deque的迭代器失效

暂记：deque双端队列，类似一个可以两头增删而不必移动元素的vector（和vector相比在头部增删元素的效率更高）

##### 添加元素引发的迭代器失效

1. 插入除首尾位置外的任何位置都会导致全部迭代器失效
2. 在首尾添加，迭代器也会失效（？？？不懂），但指向存在元素的指针和引用不会失效

##### 删除元素引发的迭代器失效

1. 删除在首尾位置外的任何位置都会导致全部迭代器失效
2. 删除首/尾元素，首/尾迭代器会失效，但其他位置的不会



#### list和forward_list的迭代器失效

1. 很显然无论插入还是删除，其他位置的元素的存储空间是不会变的，所以只有删除位置的迭代器会失效





# C++11

## \<future>

### future对象

1. 从异步执行的函数获取return value，例如从另一个线程执行的函数获得返回值
2. 从异步的对象获取值，详见promise

c++保证不同线程对future对象的正确同步访问，例如一个线程要通过future.get获取在另一个线程执行的函数fn的return value，但fn尚未返回值，那么该线程会阻塞直到fn返回值

future的值只能get一次，如果要get多次需要使用shared_future



### async函数

给调用者返回一个future对象

- 设置async策略：可以开一个新线程执行fn并将fn的返回值存入future，调用线程通过future对象可以获取这个返回值
- 设置defer策略：延迟执行fn，当线程要通过future获取fn返回值时再执行fn

http://www.cplusplus.com/reference/future/async/



### package_task对象

类似function对象，但是可以通过future来获得执行的fn的返回值

```c++
int incre(int &a)
{
	int temp = a;
	a++;
	return temp;
}
int main() {
	int a = 1;

	packaged_task<int(int&)> tsk(incre);
	auto f1 = tsk.get_future();

	thread t(move(tsk), ref(a));
	std::cout <<"a cur val: " << a 
		<<"\t incre func return val: "<< f1.get() << endl;	//output： a cur val:2  incre func return val:1
}
```



### promise对象

理解：当一个过程需要一个值，但这个值又不能马上提供，这时可以通过future稍后某个时刻再提供这个值

下面的例子，thread2需要a的值，main通过promise对象设置这个值，然后future再从promise对象获取这个值

```c++
void p_int(future<int> & f)
{
	int a = f.get();
	cout << "p_int: " << a << endl;
}

promise<int> prom;
auto f2 = prom.get_future();
thread t2(p_int, ref(f2)); t2.detach();

cout << "main thread ready to setValue" << endl;
prom.set_value(a);
```

