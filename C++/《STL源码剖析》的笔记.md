# 庖丁解牛 恢恢乎游刃有余

第 1 章 STL 概论与版本简介
第 2 章 空间配置器（allocator）
第 3 章 迭代器（iterators）概念与 traits 编程技法
第 4 章 序列式容器（sequence containers）
第 5 章 关联式容器（associattive containers）
第 6 章 算法（algorithms）
第 7 章 仿函数（functors，另名 函数对象 function objects）
第 8 章 配接器（adapters）

## Chapter1 STL概论与版本简介

STL的价值在两方面。低层次而言，STL 带给我们一套极具实用价值的零组件，以及一个整合的组织。这种价值就像 MFC 或 VCL 之于 Windows软件开发过程所带来的价值一样，直接而明朗，令大多数人有最立即明显的感受。除此之外 STL还带给我们一个高层次的、以泛型思维（Generic Paradigm）为基础的、系统化的、条理分明的「软件组件分类学（components taxonomy）」。从这个角度来看，STL是个抽象概念库（library of abstract concepts），这些「抽象概念」包括最基础的Assignable（可被赋值）、Default Constructible（不需任何自变量就可建构）、Equality Comparable（可判断是否等同）、LessThan Comparable（可比较大小）、Regular（正规）…，高阶一点的概念则包括 Input Iterator（具输入功能的迭代器）、Output Iterator（具输出功能的迭代器）、Forward Iterator（单向迭代器）、Bidirectional Iterator（双向迭代器）、Random Access Iterator（随机存取迭代器）、Unary Function（一元函数）、Binary Function（二元函数）、Predicate（传回真假值的一元判断式）、Binary Predicate（传回真假值的二元判断式）…，更高阶的概念包括 sequence container（序列式容器）、associative container（关系型容器）…。STL的创新价值便在于具体叙述了上述这些抽象概念，并加以系统化

STL 所实现的，是依据泛型思维架设起来的一个概念结构。这个以抽象概念（abstract concepts）为主体而非以实际类别（classes）为主体的结构，形成了一个严谨的接口标准。在此接口之下，任何组件有最大的独立性，并以所谓迭代器（iterator）胶合起来，或以所谓配接器（adapter）互相配接，或以所谓仿函数（functor）动态选择某种策略（policy 或 strategy）。

STL提供六大组件，彼此可以组合套用：

1. 容器（containers）：各种数据结构，如 vector, list, deque, set, map ，用来存放数据，详见本书 4, 5 两章。从实作的角度看，STL 容器是一种 class template。就体积而言，这一部份很像冰山在海面下的比率。
2. 算法（algorithms）：各种常用算法如 sort, search, copy, erase …，详见第 6 章。从实作的角度看，STL 算法是一种 function template。
3. 迭代器（iterators）：扮演容器与算法之间的胶着剂，是所谓的「泛型指标」，详见第3章。共有五种类型，以及其它衍生变化。从实作的角度看，迭代器是一种将 operator*, operator->, operator++, operator-- 等指标相关操作予以多载化的 class template。所有STL容器都附带有自己专属的迭代器—是的，只有容器设计者才知道如何巡访自己的元素。原生指标（native pointer）也是一种迭代器。
4. 仿函数（functors）/函数对象：行为类似函数，可做为算法的某种策略（policy），详见第 7 章。从实作的角度看，仿函数是一种重载了 operator() 的 class 或class template。一般函数指标可视为狭义的仿函数。
5. 配接器（adapters）：一种用来修饰容器（containers）或仿函数（functors）或迭代器（iterators）接口的东西，详见第 8 章。例如 STL 提供的 queue 和stack ，虽然看似容器，其实只能算是一种容器配接器，因为它们的底部完全借助 deque ，所有动作都由底层的 deque 供应。改变functor接口者，称为function adapter，改变container接口者，称为container adapter，改变iterator接口者，称为iterator adapter。配接器的实作技术很难一言以蔽之，必须逐一分析，详见第 8 章。
6. 配置器（allocators）：负责空间配置与管理，详见第 2 章。从实作的角度看，配置器是一个实现了动态空间配置、空间管理、空间释放的 class template。

图 1-1 显示 STL 六大组件的交互关系。

![《STL源码剖析》的笔记-STLsixcomponents.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-STLsixcomponents.png)

### 可能令你困惑的C++语法

#### 临时对象的产生与运用

所谓临时对象，就是一种无名对象（unnamed objects）。它的出现如果不在程序员的预期之下（例如任何pass by value/值传递动作都会引发copy动作，于是形成一个临时对象），往往造成效率上的负担 。但有时候刻意制造一些临时对象，却又是使程序干净清爽的技巧。刻意制造临时对象的方法是，在类型名称之后直接加一对小括号，并可指定初值，例如 Shape(3,5) 或 int(8) ，其意义相当于唤起相 应 的constructor且 不 指 定对象 名 称 。 STL 最 常 将 此 技 巧 应 用 于 仿 函 数（functor）与算法的搭配上

#### function call操作符（operator()）

函数调用符号`()`也可以被重载

许多STL算法都提供两个版本，一个用于一般状况（例如排序时以递增方式排列），一个用于特殊状况（例如排序时由使用者指定以何种特殊关系进行排列）。像这种情况，需要使用者指定某个条件或某个策略，而条件或策略的背后由一整组动作构成，便需要某种特殊的东西来代表这「一整组动作」。代表「一整组动作」的，当然是函式。过去 C语言时代，欲将函式当做参数传递，唯有透过函式指标（pointer to function，或称 function pointer）才能达成

但是函式指标有缺点，最重要的是它无法持有自己的状态（所谓区域状态，local states），也无法达到组件技术中的可配接性（adaptability）—也就是无法再将某些修饰条件加诸于其上而改变其状态。为此，STL 算法的特殊版本所接受的所谓「条件」或「策略」或「一整组动作」，都以仿函式形式呈现。所谓仿函式（functor）就是使用起来像函式一样的东西。如果你针对某个 class 进行 operator() 多载化，它就成为一个仿函式。至于要成为一个可配接的仿函式，还需要一些额外的努力（详见第 8 章）。

下面是将operator()重载的例子

```c++
// file: 1functor.cpp
#include <iostream>
using namespace std;
//由于将 operator() 多载化了，因此 plus 成了一个仿函式
template <class T>
struct plus {
    T operator() (const T& x, const T& y) const { return x + y; }
};
//由于将 operator() 多载化了，因此 minus成了一个仿函式
template <class T>
struct minus {
    T operator() (const T& x, const T& y) const { return x - y; }
};
int main()
{
    // 以下产生仿函式对象。
    plus<int> plusobj;
    // 以下使用仿函式，就像使用一般函式一样。
    cout << plusobj(3,5) << endl; // 8
    cout << minusobj(3,5) << endl; // -2
    // 以下直接产生仿函式的暂时对象（第一对小括号），并呼叫之（第二对小括号）。
    cout << plus<int>() (43,50) << endl; // 93
    cout << minus<int>() (43,50) << endl; // -7
}
```

## Chapter2 空间配置器allocator

以STL 的运用角度而言，空间配置器是最不需要介绍的东西，它总是隐藏在一切组件（更具体地说是指容器，container）的背后，默默工作默默付出。但若以 STL的实作角度而言，第一个需要介绍的就是空间配置器，因为整个STL的操作对象（所有的数值）都存放在容器之内，而容器一定需要配置空间以置放数据。不先掌握空间配置器的原理，难免在观察其它 STL 组件的实作时处处遇到挡路石。

因为空间不一定是内存，也可以是磁盘或其他存储媒介，所以不叫内存配置器

### 空间配置器的标准接口

根据 STL 的规范，以下是allocator的必要接口

```c++
//以下各种 type 的设计原由，第三章详述。
allocator::value_type
allocator::pointer
allocator::const_pointer
allocator::reference
allocator::const_reference
allocator::size_type
allocator::difference_type
allocator::rebind
    // 一个嵌套的（nested）class template。class rebind<U> 拥有唯一成员 other ，那是一个 typedef，代表 allocator<U> 。
allocator::allocator()
    // default constructor。
allocator::allocator (const allocator&)
    // copy constructor。
template <class U>allocator:: allocator (const allocator<U>&)
    // 泛化的copy constructor。
allocator::~allocator()
    // default constructor。
pointer allocator:: address (reference x) const
    // 传回某个对象的地址。算式 a.address(x) 等同于 &x 。
const_pointer allocator:: address (const_reference x) const
    // 传回某个 const 对象的地址。算式 a.address(x) 等同于 &x 。
pointer allocator:: allocate (size_type n, cosnt void* = 0)
    // 配置空间，足以储存 n 个 T 对象。第二自变量是个提示。实作上可能会利用它来增进区域性（locality），或完全忽略之。
void allocator:: deallocate (pointer p, size_type n)
    // 归还先前配置的空间。
size_type allocator:: max_size() const
    // 传回可成功配置的最大量。
void allocator:: construct (pointer p, const T& x)
    // 等同于 new(const void*) p) T(x) 。
void allocator:: destroy(pointer p)
    // 等同于 p->~T() 。
```

### 具备次配置力的SGI空间配置器

SGI STL 的 配 置 器 与 众 不 同 ， 也 与 标 准 规 范 不 同 ， 其 名 称 是 alloc 而 非allocator ，而且不接受任何自变量。换句话说如果你要在程序中明白采用SGI 配置器，不能采用标准写法：

```c++
vector<int, std::allocator<int> > iv; // in VC or CB
```

必须这么写：

```c++
vector<int, std::alloc> iv; // in GCC
```

SGI STL allocator未能符合标准规格，这个事实通常不会对我们带来困扰，因为通常我们使用预设的空间配置器，很少需要自行指定配置器名称，而SGI STL的每一个容器都已经指定其预设的空间配置器为 alloc 。例如下面的 vector 宣告：

```c++
template <class T, class Alloc = alloc> // 预设使用 alloc为配置器
class vector { ... };
```

#### SGI 标准的空间配置器，std::allocator（不推荐）

虽然 SGI 也定义有一个符合部份标准、名为 allocator 的配置器，但SGI自己从未用过它，也不建议我们使用。主要原因是效率不彰，只把 C++的 ::operator new 和 ::operator delete 做一层薄薄的包装而已

#### SGI 特殊的空间配置器 ，std::alloc（推荐）

一般我们习惯的C++内存配置操作和释放操作如下

```c++
class Foo { ... };
Foo* pf = new Foo; //配置内存，然后建构对象
delete pf; //将对象解构，然后释放内存
```

new包含两段操作：

- 调用::operator new 配置内存
- 调用Foo::Foo()构造对象内容

delete也包含两段操作

- 调用Foo::~Foo()将对象析构
- 调用::operator delete释放内存

为了精密分工，STL allocator 决定将这两阶段动作区分开来。

- 内存配置动作由 alloc:allocate() 负责
- 内存释放动作由 alloc::deallocate() 负责
- 对象建构动作由 ::construct() 负责
- 对象解构动作由 ::destroy() 负责

![《STL源码剖析》的笔记-stlconstructanddestroy.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-stlconstructanddestroy.png)

#### construct()和destroy()

下面是 <stl_construct.h> 的部份内容（阅读程序代码的同时，请参考图 2-1）：

```c++
#include <new.h>
//欲使用 placement new ，需先含入此文件
template <class T1, class T2>
inline void construct (T1* p, const T2& value) {
    new (p) T1(value) ; // placement new ;唤起 T1::T1(value);
}

//以下是 destroy() 第一版本，接受一个指标。
template <class T>
inline void destroy (T* pointer) {
    pointer->~T() ; //唤起 dtor ~T()
}

//以下是 destroy() 第二版本，接受两个迭代器。此函式设法找出元素的数值型别，
//进而利用 __type_traits<> 求取最适当措施。
template <class ForwardIterator>
inline void destroy (ForwardIterator first, ForwardIterator last) {
    __destroy (first, last, value_type(first));
}

//判断元素的数值型别（ value type ）是否有 trivial destructor
template <class ForwardIterator, class T>
inline void __destroy (ForwardIterator first, ForwardIterator last, T*)
{
    typedef typename __type_traits<T>::has_trivial_destructor trivial_destructor;
    __destroy_aux(first, last, trivial_destructor());
}

//如果元素的数值型别（ value type ）有 non-trivial destructor…
template <class ForwardIterator>
inline void
__destroy_aux (ForwardIterator first, ForwardIterator last, __false_type ) {
    for ( ; first < last; ++first)
        destroy(&*first);
}
//如果元素的数值型别（ value type ）有 trivial destructor…
template <class ForwardIterator>
inline void __destroy_aux (ForwardIterator, ForwardIterator, __true_type ) {}

//以下是 destroy()第二版本针对迭代器为 char*和 wchar_t* 的特化版
inline void destroy (char*, char*) {}
inline void destroy (wchar_t*, wchar_t*) {}
```

![《STL源码剖析》的笔记-constructoranddestructor.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-constructoranddestructor.png)

这两个做为建构、解构之用的函式被设计为全域函式，符合 STL 的规范 4 。此外STL 还规定配置器必须拥有名为 construct() 和 destroy() 的两个成员函式（见2.1 节），然而真正在 SGI STL 中大显身手的那个名为 std::alloc 的配置器并未遵守此一规则（稍后可见）。

#### 空间的配置与释放，std::alloc

考虑小型区块所可能造成的内存破碎问题，SGI 设计了双层级配置器，第一级配置器直接使用 malloc() 和 free() ，第二级配置器则视情况采用不同的策略：当配置区块超过128bytes，视之为「足够大」，便呼叫第一级配置器；当配置区块小于 128bytes，视之为「过小」，为了降低额外负担（overhead，见 2.2.6 节），便采用复杂的memory pool整理方式，而不再求助于第一级配置器。整个设计究竟只开放第一级配置器，或是同时开放第二级配置器，取决于 __USE_MALLOC  是否被定义（唔，我们可以轻易测试出来，SGI STL 并未定义 __USE_MALLOC） ：

>注：__USE_MALLOC 这个名称取得不甚理想，因为无论如何，最终总是使用 malloc()

```c++
# ifdef __USE_MALLOC
...
typedef __malloc_alloc_template <0> malloc_alloc;
typedef malloc_alloc alloc;
# else
...
//令 alloc 为第二级配置器
//令 alloc为第一级配置器
typedef __default_alloc_template<__NODE_ALLOCATOR_THREADS, 0> alloc;
#endif /* ! __USE_MALLOC */
```

其 中 __malloc_alloc_template 就 是 第 一 级 配 置 器 ， __default_alloc_template 就是第二级配置器。稍后分别有详细介绍。再次提醒你注意， alloc 并不接受任何 template 型别参数。

无论 alloc 被定义为第一级或第二级配置器，SGI 还为它再包装一个接口如下，使配置器的接口能够符合 STL规格，（个人认为这个接口的目的是方便容器配置以容器元素大小为单位的空间）

```c++
template<class T, class Alloc>
class simple_alloc {
public:
static T * allocate(size_t n)
    { return 0 == n? 0 : (T*) Alloc::allocate(n * sizeof (T)); }
static T * allocate(void)
    { return (T*) Alloc::allocate(sizeof (T)); }
static void deallocate (T *p, size_t n)
    { if (0 != n) Alloc::deallocate(p, n * sizeof (T)); }
static void deallocate(T *p)
    { Alloc::deallocate(p, sizeof (T)); }
};
```

其内部四个成员函式其实都是单纯的转调用，调用传入之配置器（可能是第一级也可能是第二级）的成员函式。这个接口使配置器的配置单位从 bytes转为个别元素的大小（ sizeof(T) ）。SGI STL 容器全都使用这个 simple_alloc 接口，例如：

```c++
template <class T, class Alloc = alloc> // 预设使用 alloc为配置器
class vector {
protected:
// 专属之空间配置器，每次配置一个元素大小
    typedef simple_alloc<value_type, Alloc>data_allocator;
    void deallocate () {
        if (...)
            data_allocator::deallocate(start, end_of_storage - start);
    }
// ...
};
```

两级配置器的关系、接口包装以及实际应用方式如下

![《STL源码剖析》的笔记-stltwoadapators.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-stltwoadapators.png)

![《STL源码剖析》的笔记-stladaptorinterface.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-stladaptorinterface.png)

#### 第一级配置器 __malloc_alloc_template 剖析

第一级配置器以malloc（），free（），rea1lc（）等C函数执行实际的内存配置、释放、重配置操作，并实现出类似C++ new-handler的机制。是的，它不能直接运用C++ new-handler机制，因为它并非使用::operator new来配置内存所谓C++ new handler机制是，你可以要求系统在内存配置需求无法被满足时，调用一个你所指定的函数。换句话说，一旦::operator new无法完成任务在丢出 std：bad alloc异常状态之前，会先调用由客端指定的处理例程。该处理例程通常即被称为new-handler

请注意，SGI 第一级配置器的 allocate() 和 realloc() 都是在呼叫 malloc()和 realloc() 不成功后，改呼叫 oom_malloc() 和 oom_realloc() 。后两者都有内循环，不断呼叫「内存不足处理例程」，期望在某次呼叫之后，获得足够的内存而圆满达成任务。但如果「内存不足处理例程」并未被客端设定，oom_malloc() 和 oom_realloc() 便老实不客气地呼叫 __THROW_BAD_ALLOC ，丢出bad_alloc异常讯息，或利用 exit(1) 硬生生中止程序。

#### 第二级配置器 __default_alloc_template 剖析

第二级配置器多了一些机制，避免太多小额区块造成内存的破碎。小额区块带
来的其实不仅是内存破碎而已，配置时的额外负担（overhead）也是一大问题 8 。
额外负担永远无法避免，毕竟系统要靠这多出来的空间来管理内存，如图2-3。
但是区块愈小，额外负担所占的比例就愈大、愈显得浪费。

![《STL源码剖析》的笔记-memorytax.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-memorytax.png)

SGI第二级配置器的作法是，如果区块够大，超过 128 bytes，就移交第一级配置器处理。当区块小于 128 bytes，则以记忆池（memory pool）管理，此法又称为次层配置（sub-allocation）：每次配置一大块内存，并维护对应之自由串行（free-list）。下次若再有相同大小的内存需求，就直接从free-lists中拨出。如果客端释还小额区块，就由配置器回收到free-lists中—是的，别忘了，配置器除了负责配置，也负责回收。为了方便管理，SGI第二级配置器会主动将任何小额区块的内存需求量上调至8的倍数（例如客端要求 30 bytes，就自动调整为 32bytes），并维护 16 个 free-lists，各自管理大小分别为 8, 16, 24, 32, 40, 48, 56, 64, 72,80, 88, 96, 104, 112, 120, 128 bytes的小额区块。free-lists 的节点结构如下：

```c++
union obj {
    union obj * free_list_link;
    char client_data[1]; /* The client sees this. */
};
```

![《STL源码剖析》的笔记-secondadaptorfreelist.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-secondadaptorfreelist.png)

#### 空间配置函数allocate()

身 为 一 个 配 置 器 ， __default_alloc_template 拥 有 配 置 器 的 标 准 介 面 函 式
allocate() 。此函式首先判断区块大小，大于 128 bytes 就呼叫第一级配置器，小
于 128 bytes 就检查对应的 free list。如果free list之内有可用的区块，就直接拿来
用，如果没有可用区块，就将区块大小上调至 8 倍数边界，然后呼叫 refill() ，
准备为 free list 重新填充空间。 refill() 将于稍后介绍。

```c++
// n must be > 0
static void * allocate(size_t n)
{
    obj * volatile * my_free_list;
    obj * result;
    // 大于 128 就呼叫第一级配置器
    if (n > (size_t) __MAX_BYTES)
        return(malloc_alloc::allocate (n));
    }
    // 寻找 16 个 free lists 中适当的一个
    my_free_list = free_list + FREELIST_INDEX (n);
    result = *my_free_list;
    if (result == 0) {
        // 没找到可用的 free list ，准备重新填充 free list
        void *r = refill (ROUND_UP(n));
        return r;
    }
    // 调整 free list
    //下节详述
    *my_free_list = result -> free_list_link;
    return (result);
};
```

区块自free list拨出的动作，如图

![《STL源码剖析》的笔记-allocatefunction.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-allocatefunction.png)

#### 空间释放函数deallocate()

身 为 一 个 配 置 器 ， __default_alloc_template 拥 有 配 置 器 标 准 介 面 函 式deallocate() 。此函式首先判断区块大小，大于 128 bytes 就呼叫第一级配置器，小于 128 bytes 就找出对应的 free list，将区块回收。

```c++
// p 不可以是 0
static void deallocate (void *p, size_t n)
{
    obj *q = (obj *)p;
    obj * volatile * my_free_list;
    // 大于 128 就呼叫第一级配置器
    if (n > (size_t) __MAX_BYTES) {
        malloc_alloc::deallocate (p, n);
        return;
    }
    // 寻找对应的 free list
    my_free_list = free_list + FREELIST_INDEX(n);
    // 调整 free list ，回收区块
    q -> free_list_link = *my_free_list;
    *my_free_list = q;
    }
```

区块回收纳入free list的动作，如图

![《STL源码剖析》的笔记-deallocatefunction.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-deallocatefunction.png)

#### 重新填充free lists

回头讨论先前说过的 allocate() 。当它发现free list中没有可用区块了，就呼叫refill() 准 备 为free list重 新 填 充 空 间 。 新 的 空 间 将 取 自 记 忆 池 （ 经 由chunk_alloc() 完成）。预设取得20个新节点（新区块），但万一记忆池空间不足，获得的节点数（区块数）可能小于 20

#### 内存池/记忆池（memory pool）

举个例子，见图 2-7，假设程序一开始，客端就呼叫 chunk_alloc(32,20) ，于是malloc() 配置 40个 32bytes区块，其中第 1 个交出，另 19 个交给 free_list[3]维 护 ， 余20 个 留 给 记 忆 池 。 接 下 来 客 端 呼 叫 chunk_alloc(64,20) ， 此 时free_list[7] 空空如也，必须向记忆池要求支持。记忆池只够供应 (32*20)/64=10个 64bytes区块，就把这 10 个区块传回，第 1 个交给客端，余 9个由 free_list[7]维护。此时记忆池全空。接下来再呼叫 chunk_alloc(96, 20) ，此时 free_list[11]空空如也，必须向记忆池要求支持，而记忆池此时也是空的，于是以 malloc() 配置 40+n（附加量）个 96bytes 区块，其中第 1 个交出，另 19 个交给 free_list[11]维护，余 20+n（附加量）个区块留给记忆池……。万一山穷水尽，整个system heap 空间都不够了（以至无法为记忆池注入活水源头）， malloc() 行动失败， chunk_alloc() 就㆕处寻找有无「尚有未用区块，且区块够大」之free lists。找到的话就挖一块交出，找不到的话就呼叫第一级配置器。第一级配置器其实也是使用 malloc() 来配置内存，但它有 out-of-memory处理机制（类似 new-handler 机制），或许有机会释放其它的内存拿来此处使用。如果可以，就成功，否则发出bad_alloc异常。

![《STL源码剖析》的笔记-memorypool.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-memorypool.png)

SGI STL默认使用第二级配置器

### 内存基本处理工具

STL定义有五个全域函式，作用于未初始化空间上。这样的功能对于容器的实作很有帮助，我们会在第4章容器实作码中，看到它们的吃重演出。前两个函式是 2.2.3节说过，用于建构 的 construct() 和用于解构的 destroy() ，另三个函式是uninitialized_copy(),uninitialized_fill(),uninitialized_fill_n() 10，分别对应于高阶函式 copy() 、 fill() 、 fill_n()— 这些都是 STL 算法，将在第六章介绍。如果你要使用本节的三个低阶函式，应该含入 `<memory>` ，不过SGI 把它们实际定义于 `<stl_uninitialized>` 。

#### 1. uninitialized_copy

```c++
template <class InputIterator, class ForwardIterator>
ForwardIterator uninitialized_copy (InputIterator first, InputIterator last, ForwardIterator result);
```

uninitialized_copy() 使我们能够将内存的配置与对象的建构行为分离开来。如果做为输出目的地的 [result, result+(last-first)) 范围内的每一个迭 代 器 都 指 向 未 初 始 化 区 域 ， 则 uninitialized_copy() 会 使 用copy constructor，为身为输入来源之 [first,last) 范围内的每一个对象产生一份复制品，放进输出范围中。换句话说，针对输入范围内的每一个迭代器 i ，此函式会呼叫 construct(&*(result+(i-first)),*i) ，产生 *i 的复制品，放置于输出范围的相对位置上。式中的 construct() 已于 2.2.3 节讨论过。

如果你有需要实作一个容器， uninitialized_copy() 这样的函式会为你带来很大的帮助，因为容器的全范围建构式（range constructor）通常以两个步骤完成：

- 配置内存区块，足以包含范围内的所有元素。
- 使用 uninitialized_copy() ，在该内存区块上建构元素。

C++标准规格书要求 uninitialized_copy() 具有 "commit or rollback"语意，意思是要不就「建构出所有必要元素」，要不就（当有任何一个copy constructor失败时）「不建构任何东西」。

#### 2. uninitialized_fill

```c++
template <class ForwardIterator, class T>
void uninitialized_fill (ForwardIterator first, ForwardIterator last, const T& x);
```

uninitialized_fill() 也能够使我们将内存配置与对象的建构行为分离开来。如果 [first,last) 范围内的每个迭代器都指向未初始化的内存，那么uninitialized_fill() 会在该范围内产生 x （上式第三参数）的复制品。换句话 说 uninitialized_fill() 会 针 对 操 作 范 围 内 的 每 个 迭 代 器 i ， 呼 叫construct(&*i, x) ，在 i 所指之处产生 x 的复制品。式中的 construct() 已于 2.2.3节讨论过。

和 uninitialized_copy() 一样， uninitialized_fill() 必须具备 "commit orrollback"语意，换句话说它要不就产生出所有必要元素，要不就不产生任何元素。如果有任何一个copy constructor丢出异常（exception）， uninitialized_fill()必须能够将已产生之所有元素解构掉。

#### 3. uninitialized_fill_n

```c++
template <class ForwardIterator, class Size, class T>
ForwardIterator uninitialized_fill_n (ForwardIterator first, Size n, const T& x);
```

uninitialized_fill_n() 能够使我们将内存配置与对象建构行为分离开来。它会为指定范围内的所有元素设定相同的初值。如果 [first, first+n) 范围内的每一个迭代器都指向未初始化的内存，那么uninitialized_fill_n() 会呼叫copy constructor，在该范围内产生 x （上式第三参数）的复制品。也就是说面对 [first,first+n) 范围内的每个迭代器 i ，uninitialized_fill_n() 会呼叫 construct(&*i, x) ，在对应位置处产生 x 的复制品。式中的 construct() 已于 2.2.3 节讨论过。

uninitialized_fill_n() 也具有 "commit or rollback"语意：要不就产生所有必要的元素，否则就不产生任何元素。如果任何一个copy constructor丢出异常（exception）， uninitialized_fill_n() 必须解构已产生的所有元素。

以 下 分 别 介 绍 这 三 个 函 式 的 实 作 法 。 其 中 所 呈 现 的iterators （ 迭 代 器 ） 、value_type() 、 __type_traits 、 __true_type 、 __false_type 、 is_POD_type等实作技术，都将于第三章介绍。

#### 三个函数的实现

##### uninitialized_fill_n

首先是 uninitialized_fill_n() 的源码。本函式接受三个参数：

1. 迭代器 first 指向欲初始化空间的起始处
2. n 表示欲初始化空间的大小
3. x 表示初值

```c++
template <class ForwardIterator, class Size, class T>
inline ForwardIterator uninitialized_fill_n (ForwardIterator first,
Size n, const T&x ) {
    return __uninitialized_fill_n(first, n, x, value_type(first) );
    // 以上，利用 value_type() 取出 first的 value type.
}
```

这个函式的进行逻辑是，首先萃取出迭代器 first 的 value type（详见第三章），
然后判断该型别是否为 POD型别：

```c++
template <class ForwardIterator, class Size, class T, class T1>
inline ForwardIterator __uninitialized_fill_n (ForwardIterator first,
Size n, const T& x, T1*)
{
    // 以下 __type_traits<> 技法，详见 3.7 节
    typedef typename __type_traits <T1>::is_POD_type is_POD;
    return __uninitialized_fill_n_aux(first, n, x, is_POD());
}
```

POD意指 Plain Old Data，也就是纯量型别（scalar types）或传统的 C struct型别。POD型别必然拥有 trivialctor/dtor/copy/assignment函式，因此，我们可以对POD型别采取最有效率的初值填写手法，而对non-POD 型别采取最保险安全的作法：

```c++
//如果 copy construction 等同于 assignment, 而且
// destructor 是 trivial，以下就有效。
//如果是 POD型别，执行流程就会转进到以下函式。这是藉由 function template
//的自变量推导机制而得。
template <class ForwardIterator, class Size, class T>
inline ForwardIterator
__uninitialized_fill_n_aux (ForwardIterator first, Size n,
const T& x, __true_type ) {
    return fill_n (first, n, x); //交由高阶函式执行。见 6.4.2节。
}
// 如果不是 POD 型别，执行流程就会转进到以下函式。这是藉由 function template
//的自变量推导机制而得。
template <class ForwardIterator, class Size, class T>
ForwardIterator
__uninitialized_fill_n_aux (ForwardIterator first, Size n,
const T& x, __false_type ) {
    ForwardIterator cur = first;
    // 为求阅读顺畅，以下将原本该有的异常处理（ exception handling ）省略。
    for ( ; n > 0; --n, ++cur)
        construct(&*cur, x);
    return cur;
}
```

##### uninitialized_copy

下面列出 uninitialized_copy() 的源码。本函式接受三个参数：

- 迭代器 first 指向输入端的起始位置
- 迭代器 last 指向输入端的结束位置（前闭后开区间）
- 迭代器 result 指向输出端（欲初始化空间）的起始处

```c++
template <class InputIterator, class ForwardIterator>
inline ForwardIterator
uninitialized_copy (InputIterator first, InputIterator last,
ForwardIterator result) {
    return __uninitialized_copy(first, last, result,value_type(result) );
    // 以上，利用 value_type() 取出 first的 value type.
}
```

这个函式的进行逻辑是，首先萃取出迭代器 result 的 value type（详见第三章），
然后判断该型别是否为 POD型别：

```c++
template <class InputIterator, class ForwardIterator, class T>
inline ForwardIterator
__uninitialized_copy (InputIterator first, InputIterator last,
ForwardIterator result, T*) {
typedef typename __type_traits<T>::is_POD_type is_POD;
    return __uninitialized_copy_aux(first, last, result, is_POD());
    // 以上，企图利用 is_POD() 所获得的结果，让编译器做自变量推导。
}
```

POD意指 Plain Old Data，也就是纯量型别（scalar types）或传统的 C struct型别。POD型别必然拥有 trivialctor/dtor/copy/assignment函式，因此，我们可以对POD型别采取最有效率的复制手法，而对 non-POD 型别采取最保险安全的作法：

```c++
//如果 copy construction 等同于 assignment, 而且
// destructor 是 trivial，以下就有效。
//如果是 POD型别，执行流程就会转进到以下函式。这是藉由 function template
//的自变量推导机制而得。
template <class InputIterator, class ForwardIterator>
inline ForwardIterator
__uninitialized_copy_aux (InputIterator first, InputIterator last, ForwardIterator result, __true_type ) {
    return copy (first, last, result); //呼叫 STL算法 copy()
}

// 如果是 non-POD型别，执行流程就会转进到以下函式。这是藉由 function template
//的自变量推导机制而得。
template <class InputIterator, class ForwardIterator>
ForwardIterator
__uninitialized_copy_aux (InputIterator first, InputIterator last, ForwardIterator result, __false_type ) {
    ForwardIterator cur = result;
    // 为求阅读顺畅，以下将原本该有的异常处理（ exception handling ）省略。
    for ( ; first != last; ++first, ++cur)
        construct (&*cur, *first); //必须一个一个元素地建构，无法批量进行
    return cur;
}
```

针对 char* 和 wchar_t* 两种型别，可以最具效率的作法 memmove （直接搬移内存内容）来执行复制行为。因此 SGI 得以为这两种型别设计一份特化版本。

```c++
//以下是针对 const char* 的特化版本
inline char*uninitialized_copy (const char* first, const char* last,
char* result) {
memmove (result, first, last - first);
return result + (last - first);
}
//以下是针对 const wchar_t* 的特化版本
inline wchar_t* uninitialized_copy (const wchar_t* first, const wchar_t* last,
wchar_t* result) {
memmove (result, first, sizeof(wchar_t) * (last - first));
return result + (last - first);
}
```

##### uninitialized_fill

下面列出 uninitialized_fill() 的源码。本函式接受三个参数：

- 迭代器 first 指向输出端（欲初始化空间）的起始处
- 迭代器 last 指向输出端（欲初始化空间）的结束处（前闭后开区间）
- x 表示初值

```c++
template <class ForwardIterator, class T>
inline void uninitialized_fill (ForwardIterator first, ForwardIterator last,
const T& x) {
__uninitialized_fill(first, last, x, value_type(first) );
}
```

这个函式的进行逻辑是，首先萃取出迭代器 first 的 value type（详见第三章），然后判断该型别是否为 POD型别：

```c++
template <class ForwardIterator, class T, class T1>
inline void __uninitialized_fill (ForwardIterator first, ForwardIterator last,
const T& x, T1*) {
typedef typename __type_traits<T1>::is_POD_type is_POD;
__uninitialized_fill_aux(first, last, x, is_POD());
}
```

POD意指 Plain Old Data，也就是纯量型别（scalar types）或传统的 C struct型别。POD型别必然拥有 trivialctor/dtor/copy/assignment函式，因此，我们可以对POD型别采取最有效率的初值填写手法，而对non-POD 型别采取最保险安全的作法

```c++
//如果 copy construction 等同于 assignment, 而且
// destructor 是 trivial，以下就有效。
//如果是 POD型别，执行流程就会转进到以下函式。这是藉由 function template
//的自变量推导机制而得。
template <class ForwardIterator, class T>
inline void
__uninitialized_fill_aux (ForwardIterator first, ForwardIterator last,
const T& x, __true_type)
{
    fill (first, last, x); //呼叫 STL算法 fill()
}

// 如果是 non-POD型别，执行流程就会转进到以下函式。这是藉由 function template
//的自变量推导机制而得。
template <class ForwardIterator, class T>
void
__uninitialized_fill_aux (ForwardIterator first, ForwardIterator last,
const T& x, __false_type)
{
    ForwardIterator cur = first;
    // 为求阅读顺畅，以下将原本该有的异常处理（ exception handling ）省略。
    for ( ; cur != last; ++cur)
        construct(&*cur, x);//必须一个一个元素地建构，无法批量进行
}
```

本节三个函式对效率的特殊考虑，以图形显示。

![《STL源码剖析》的笔记-threememoryoperatationfunction.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-threememoryoperatationfunction.png)

#### 拓展：Placement new

[推荐阅读](https://www.cnblogs.com/luxiaoxun/archive/2012/08/10/2631812.html)

Placement new 的含义

placement new 是重载 operator new 的一个标准、全局的版本，它不能够被自定义的版本代替（不像普通版本的 operator new 和 operator delete 能够被替换）。

```c++
void *operator new( size_t, void * p ) throw() { return p; }
```

placement new 的执行忽略了 size_t 参数，只返还第二个参数。其结果是允许用户把一个对象放到一个特定的地方，达到调用构造函数的效果。和其他普通的 new 不同的是，它在括号里多了另外一个参数。比如：

```c++
Widget * p = new Widget;                    //ordinary new

pi = new (ptr) int; pi = new (ptr) int;     //placement new
```

括号里的参数 ptr 是一个指针，它指向一个内存缓冲器，placement new 将在这个缓冲器上分配一个对象。Placement new 的返回值是这个被构造对象的地址 (比如括号中的传递参数)。placement new 主要适用于：在对时间要求非常高的应用程序中，因为这些程序分配的时间是确定的；长时间运行而不被打断的程序；以及执行一个垃圾收集器 (garbage collector)。

Placement new只是 operator new 重载的一个版本。它并不分配内存，只是返回指向已经分配好的某段内存的一个指针。因此不能删除它，但需要调用对象的析构函数。

如果你想在已经分配的内存中创建一个对象，使用 new 时行不通的。也就是说 placement new 允许你在一个已经分配好的内存中（栈或者堆中）构造一个新的对象。原型中 void* p 实际上就是指向一个已经分配好的内存缓冲区的的首地址

#### 拓展：trivial destructor

如果用户不定义析构函数，而是用系统自带的，则说明，析构函数基本没有什么用（但默认会被调用）我们称之为 trivial destructor。反之，如果特定定义了析构函数，则说明需要在释放空间之前做一些事情，则这个析构函数称为 non-trivial destructor。如果某个类中只有基本类型的话是没有必要调用析构函数的，delelte p 的时候基本不会产生析构代码，

在 C++ 的类中如果只有基本的数据类型，也就不需要写显式的析构函数，即用默认析构函数就够用了，但是如果类中有个指向其他类的指针，并且在构造时候分配了新的空间，则在析构函数中必须显式释放这块空间，否则会产生内存泄露，

在 STL 中空间配置时候 destory（）函数会判断要释放的迭代器的指向的对象有没有 trivial destructor（STL 中有一个 has_trivial_destructor 函数，很容易实现检测），如果有 trivial destructor 则什么都不做，如果没有即需要执行一些操作，则执行真正的 destory 函数。

把trivial翻译为“无关痛痒”或“无意义的”，也就是说这个destructor是默认的，是无关痛痒的，是无意义的

#### 拓展：C++中的trivial解释

[C++ 中的 trivial 解释](https://www.cnblogs.com/shuaihanhungry/p/5764019.html)

#### 拓展：volatile

[谈谈 C/C++ 中的 volatile](https://liam.page/2018/01/18/volatile-in-C-and-Cpp/)

[C/C++ 中 volatile 关键字详解](https://www.cnblogs.com/yc_sunniwell/archive/2010/07/14/1777432.html)

## Chapter3 迭代器（iterators）概念与traits编程技法

迭代器：提供一种方法，俾得依序巡访某个聚合物（容器）所含的各个元素，而又无需曝露该聚合物的内部表述方式。

### 3.1 迭代器设计思维 — STL 关键所在

不论是泛型思维或 STL 的实际运用，迭代器（iterators）都扮演重要角色。STL 的中心思想在于，将数据容器（containers）和算法（algorithms）分开，彼此独立设计，最后再以一帖胶着剂将它们撮合在一起。容器和算法的泛型化，从技术角度来看并不困难，C++ 的 class templates 和 function templates可分别达成目标。如何设计出两者之间的良好胶着剂，才是大难题。以下是容器、算法、迭代器（iterator，扮演黏胶角色）的合作展示。以算法 find()为例，它接受两个迭代器和一个「搜寻标的」：

```c++
//摘自 SGI <stl_algo.h>
template <class InputIterator, class T>
InputIterator find (InputIterator first,
InputIterator last,
const T& value) {
    while (first != last && *first != value)
        ++first;
    return first;
}
```

只要给予不同的迭代器， find() 便能够对不同的容器做搜寻动作：

```c++
// file : 3find.cpp
#include <vector>
#include <list>
#include <deque>
#include <algorithm>
#include <iostream>
using namespace std;
int main()
{
    const int arraySize = 7;
    int ia[arraySize] = { 0,1,2,3,4,5,6 };
    vector <int> ivect(ia, ia+arraySize);
    list <int> ilist(ia, ia+arraySize);
    deque <int> ideque(ia, ia+arraySize); //注意：VC6[x]，未符合标准
    vector<int>::iterator it1 = find (ivect.begin(), ivect.end(), 4);
    if (it1 == ivect.end())
        cout << "4 not found." << endl;
    else
        cout << "4 found. " << *it1 << endl;
    // 执行结果：4 found. 4
    list<int>::iterator it2 =find (ilist.begin(), ilist.end(), 6);
    if (it2 == ilist.end())
        cout << "6 not found." << endl;
    else
        cout << "6 found. " << *it2 << endl;
    // 执行结果：6 found. 6
    deque<int>::iterator it3 = find (ideque.begin(), ideque.end(), 8);
    if (it3 == ideque.end())
        cout << "8 not found." << endl;
    else
        cout << "8 found. " << *it3 << endl;
    // 执行结果：8 not found.
}
```

### 3.2 迭代器 （iterator ） 是一种 smart pointer

迭代器是一种行为类似指针的对象，而指针的各种行为中最常见也最重要的便是内容提领（dereference）和成员取用（member access），因此迭代器最重要的编程工作就是对 operator* 和 operator-> 进行多载化（overloading）工程

为了完成一个针对 List 而设计的迭代器，我们无可避免地曝露了太多 List 实作细节：在 main() 之中为了制作 begin 和 end 两个迭代器，我们曝露了 ListItem ；在 ListIter class之中为了达成 operator++ 的目的，我们曝露了 ListItem 的操作函式 next() 。如果不是为了迭代器， ListItem原本应该完全隐藏起来不曝光的。换句话说，要设计出 ListIter ，首先必须对 List的实作细节有非常丰富的了解。既然这无可避免，干脆就把迭代器的开发工作交给 List 的设计者好了，如此一来所有实作细节反而得以封装起来不被使用者看到。**这正是为什么每一种 STL容器都提供有专属迭代器的缘故**。

### 3.3 迭代器相应型别/类别 （associated types ）

算法之中运用迭代器时，很可能会用到其相应型别（associated type）。什么是相应型别？迭代器所指之物的型别便是其一。假设算法中有必要宣告一个变量，以「迭代器所指对象的型别」为型别，如何是好？毕竟C++只支持sizeof() ，并未支持 typeof() ！即便动用 RTTI 性质中的 typeid() ，获得的也只是型别名称，不能拿来做变量宣告之用。

解决办法是：利用 function template 的**参数推导**（argument deducation）机制。

例如：

```c++
template <class I, class T>
void func_impl(I iter, T t)
{
    T tmp; // 这里解决了问题。T就是迭代器所指之物的型别，本例为 int
    // ... 这里做原本 func()应该做的全部工作
};
template <class I>
inline
void func(I iter)
{
    func_impl( iter,*iter);// func 的工作全部移往 func_impl
}
int main()
{
    int i;
    func(&i);
}
```

我们以 func() 为对外接口，却把实际动作全部置于 func_impl() 之中。由于func_impl() 是一个 function template，一旦被呼叫，编译器会自动进行 template自变量推导。于是导出型别 T ，顺利解决了问题。迭代器相应型别（associated types）不只是「迭代器所指对象的型别」一种而已。根据经验，最常用的相应型别有五种，然而并非任何情况下任何一种都可利用上述的 template自变量推导机制来取得。我们需要更全面的解法。

### 3.4 Traits 编程技法 — STL 源码门钥

迭代器所指物件的型别，称为该迭代器的value type。上述的自变量型别推导技巧虽然可用于 value type，却非全面可用：万一value type必须用于函式的传回值，就束手无策了，毕竟函式的「template 自变量推导机制」推而导之的只是自变量，无法推导函式的回返值型别。

我们需要其它方法。宣告巢状型别似乎是个好主意，像这样：

```c++
template <class T>
struct MyIter{
    typedef T value_type; // 巢状型别宣告（nested type）
    T* ptr;
    MyIter(T* p=0) : ptr(p) { }
    T& operator*() const { return *ptr; }
    // ...
};
template <class I>
typename I::value_type //这一整行是 func的回返值型别
func(I ite)
{ return *ite; }

// ...
MyIter<int> ite(new int(8));
cout << func (ite); //输出：8
```

注意， func() 的回返型别必须加上关键词 typename ，因为 T 是一个 template参数，在它被编译器具现化之前，编译器对 T 一无所悉，换句话说编译器此时并不知道 `MyIter<T>::value_type` 代表的是一个型别或是一个 member function或是一个 data member。关键词 typename 的用意在告诉编译器说这是一个型别，如此才能顺利通过编译。

看起来不错。但是有个隐晦的陷阱：**并不是所有迭代器都是 class type。原生指针就不是**！如果不是 class type，就无法为它定义巢状型别。但 STL（以及整个泛型思维）绝对必须接受原生指标做为一种迭代器，所以上面这样还不够。有没有办法可以让上述的一般化概念针对特定情况（例如针对原生指标）做特殊化处理呢？是的，template partial specialization可以做到。

#### Partial Specialization（偏特化）的意义

如果 class template拥有一个以上的 template参数，我们可以针对其中某个（或数个，但非全部）template参数进行特化工作。换句话说我们可以在泛化设计中提供一个特化版本（也就是将泛化版本中的某些template参数赋予明确的指定）。

假设有一个 class template如下：

```c++
template<typename U, typename V, typename T>
class C { ... };
```

partial specialization的字面意义容易误导我们以为，所谓「偏特化版」一定是对template参数 U 或 V 或 T （或某种组合）指定某个自变量值。事实不然，[Austern99]对于partial specialization的意义说得十分得体：**所谓partial specialization的意思是提供另一份 template定义式，而其本身仍为 templatized**。《泛型技术》一书对 partial specialization 的定义是：**针对（任何）template 参数更进一步的条件限制，所设计出来的一个特化版本**。

由此，面对以下这么一个 class template：

```c++
template<typename T>
class C { ... }; // 这个泛化版本允许（接受）T为任何型别
```

我们便很容易接受它有一个型式如下的partial specialization：

```c++
template<typename T>
class C<T*> { ... }; //这个特化版本仅适用于「T为原生指标」的情况
// 「T为原生指标」便是「T 为任何型别」的一个更进一步的条件限制
```

有了这项利器，我们便可以解决前述「巢状型别」未能解决的问题。先前的问题是，原生指标并非 class，因此无法为它们定义巢状型别。现在，我们可以针对「迭代器之 template自变量为指标」者，设计特化版的迭代器。

#### traits萃取

提高警觉，我们进入关键地带了。下面这个 class template专门用来「萃取」迭代器的特性，而 value type 正是迭代器的特性之一：

```c++
template <class I>
structiterator_traits { // traits 意为「特性」
typedef typename I ::value_type value_type;
};
```

这个所谓的traits，其意义是，如果 I 定义有自己的value type，那么透过这个traits的作用，萃取出来的 value_type 就是 I::value_type 。换句话说如果 I定义有自己的value type，先前那个 func() 可以改写成这样：

```c++
template <class I>
typename iterator_traits<I>::value_type // 这一整行是函式回返型别
func(I ite)
{ return *ite; }
```

但这除了多一层间接性，又带来什么好处？**好处是traits可以拥有特化版本**。现在，我们令 iterator_traites 拥有一个partial specializations 如下：

```c++
template <class T>
struct iterator_traits< T* > { //偏特化版—迭代器是个原生指标
    typedef T value_type;
};
```

于是，原生指标 int* 虽然不是一种 class type，亦可透过traits取其value type。这就解决了先前的问题。但是请注意，针对「指向常数对象的指针（pointer-to- const ）」，下面这个式子得到什么结果：

```c++
iterator_traits<const int*>::value_type
```

获得的是 const int 而非 int 。这是我们期望的吗？我们希望利用这种机制来宣告一个暂时变量，使其型别与迭代器的value type相同，而现在，宣告一个无法 赋 值 （ 因 const 之 故 ） 的 暂 时 变 数 ， 没 什 么 用 ！ 因 此 ， 如 果 迭 代 器 是 个pointer-to- const ，我们应该设法令其value type为一个 non- const 型别。没问题，只要另外设计一个特化版本，就能解决这个问题：

```c++
template <class T>
struct iterator_traits< const T* > { // 偏特化版—当迭代器是个 pointer-to-const
    typedef T value_type; // 萃取出来的型别应该是 T 而非 const T
};
```

现在，不论面对的是迭代器 MyIter ，或是原生指标 int* 或 const int* ，都可以透过traits取出正确的（我们所期望的）value type。图3-1说明traits所扮演的「特性萃取机」角色，萃取各个迭代器的特性。这里所谓的迭代器特性，指的是迭代器的相应型别（associated types）。当然，若要这个「特性萃取机」traits能够有效运作，每一个迭代器必须遵循约定，**自行以巢状型别定义（nested typedef）的方式定义出相应型别（associated types）**。这种一个约定，谁不遵守这个约定，谁就不能相容于 STL 这个大家庭。

根据经验，最常用到的迭代器相应型别有五种：value type, difference type, pointer, reference,iterator catagoly。如果你希望你所开发的容器能与 STL 水乳交融，一定要为你的容器的迭代器定义这五种相应型别。「特性萃取机」traits 会很忠实地将原汁原味榨取出来：

```c++
template <class I>
structiterator_traits {
typedef typename I::iterator_category iterator_category;
typedef typename I::value_type value_type;
typedef typename I::difference_type difference_type;
typedef typename I::pointer pointer;
typedef typename I::reference reference;
};
```

iterator_traits 必须针对传入之型别为 pointer 及 pointer-to-const 者，设计特化版本，稍后数节为你展示如何进行。

#### 3.4.1 迭代器相应型别之一 ：value type

所谓value type，是指迭代器所指对象的型别。任何一个打算与 STL算法有完美搭配的 class，都应该定义自己的 value type 巢状型别，作法就像上节所述。

#### 3.4.2 迭代器相应型别之二 ：difference type

difference type 用来表示两个迭代器之间的距离，也因此，它可以用来表示一个容器的最大容量，因为对于连续空间的容器而言，头尾之间的距离就是其最大容量。如果一个泛型算法提供计数功能，例如 STL的 count() ，其传回值就必须使用迭代器的 difference type：

```c++
template <class I, class T>
typename iterator_traits<I>::difference_type //这一整行是函式回返型别
count (I first, I last, const T& value) {
    typename iterator_traits<I>::difference_type n = 0;
    for ( ; first != last; ++first)
    if (*first == value)
        ++n;
    return n;
}
```

针对相应型别difference type，traits的两个（针对原生指标而写的）特化版本如下，以C++内建的 ptrdiff_t（ 定义于 `<cstddef>` 头文件）做为原生指标的difference type：

```c++
template <class I>
structiterator_traits {
    ...
    typedef typename I::difference_type difference_type;
};

//针对原生指标而设计的「偏特化（ partial specialization ）」版
template <class T>
structiterator_traits<T*> {
    ...
    typedef ptrdiff_t difference_type;
};

//针对原生的 pointer-to- const 而设计的「偏特化（ partial specialization ）」版
template <class T>
struct iterator_traits<const T*> {
    ...
    typedef ptrdiff_t difference_type;
};
```

现在，任何时候当我们需要任何迭代器 I的difference type，可以这么写：`typename iterator_traits<I>::difference_type`

#### 3.4.3 迭代器相应型别之三 ：reference type

从「迭代器所指之物的内容是否允许改变」的角度观之，迭代器分为两种：不允许改变「所指对象之内容」者，称为constant iterators，例如 constint* pic ；允许改变「所指对象之内容」者，称为 mutable iterators，例如 int* pi 。当我们对一个 mutable iterators做提领动作时，获得的不应该是个右值（rvalue），应该是个左值（lvalue），因为右值不允许赋值动作（assignment），左值才允许：

```c++
int* pi = new int(5);
const int* pci = new int(9);
*pi = 7; // 对 mutable iterator 做提领动作时，获得的应该是个左值，允许赋值。
*pci = 1; // 这个动作不允许，因为 pci是个 constant iterator，
// 提领 pci所得结果，是个右值，不允许被赋值。
```

在 C++中，**函式如果要传回左值，都是以by reference的方式进行**，所以当 p 是个 mutable iterators时，**如果其value type是 T ，那么 *p 的型别不应该是 T ，应该是 T&** 。将此道理扩充，如果 p 是一个 constant iterators，其value type是 T ，那么 *p 的型别不应该是 const T ，而应该是 const T& 。这里所讨论的 *p 的型别，即所谓的reference type。实作细节将在下一小节一并展示。

#### 3.4.4 迭代器相应型别之四 ：pointer type

pointers和 references 在 C++中有非常密切的关连。如果「传回一个左值，令它代表 p 所指之物」是可能的，那么「传回一个左值，令它代表 p 所指之物的位址」也一定可以。也就是说我们能够传回一个 pointer，指向迭代器所指之物。这些相应型别已在先前的 ListIter class中出现过：

```c++
Item& operator* () const { return *ptr; }
Item* operator-> () const { return ptr; }
```

Item& 便是 ListIter 的reference type而 Item* 便是其pointer type。

现在我们把 reference type和pointer type 这两个相应型别加入traits内：

```c++
template <class I>
structiterator_traits {
    ...
    typedef typename I::pointer pointer;
    typedef typename I::reference reference;
};

//针对原生指标而设计的「偏特化版（ partial specialization ）」
template <class T>
structiterator_traits<T*> {
    ...
    typedef T* pointer;
    typedef T& reference;
};

//针对原生的 pointer-to- const 而设计的「偏特化版（ partial specialization ）」
template <class T>
structiterator_traits<const T*> {
    ...
    typedef const T* pointer;
    typedef const T& reference;
};
```

#### 3.4.5 迭代器相应型别之五 ：iterator_category

根据移动特性与施行动作，迭代器被分为五类：

- Input Iterator：这种迭代器所指对象，不允许外界改变。只读（read only）。
- Output Iterator：唯写（write only）。
- Forward Iterator：允许「写入型」算法（例如 replace() ）在此种迭代器所形成的区间上做读写动作。
- Bidirectional Iterator：可双向移动。某些算法需要逆向走访某个迭代器区间（例如逆向拷贝某范围内的元素），就可以使用 Bidirectional Iterators。
- Random Access Iterator：前㆕种迭代器都只供应一部份指标算术能力（前三种支持 operator++ ，第四种再加上 operator-- ），第五种则涵盖所有指标算术能力，包括 p+n, p-n, p[n], p1-p2, p1<p2 。

这些迭代器的分类与从属关系，可以图3-2 表示。直线与箭头代表的并非 C++ 的继承关系，而是所谓concept（概念）与refinement（强化）的关系 。

![《STL源码剖析》的笔记-iteratorscategory.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-iteratorscategory.png)

设计算法时，如果可能，我们尽量针对图3-2中的某种迭代器提供一个明确定义，并针对更强化的某种迭代器提供另一种定义，这样才能在不同情况下提供最大效率。研究STL 的过程中，每一分每一秒我们都要念兹在兹，效率是个重要课题。假设有个算法可接受Forward Iterator，你以Random Access Iterator喂给它，它当然也会接受，因为一个Random Access Iterator必然是一个ForwardIterator（见图 3-2）。但是可用并不代表最佳！

##### 以 advanced()为例

拿 advance() 来说（这是许多算法内部常用的一个函式），此函式有两个参数，迭代器 p 和数值 n ；函式内部将 p 累进 n 次（前进 n 距离）。下面有三份定义，一份针对Input Iterator，一份针对Bidirectional Iterator，另一份针对Random Access Iterator。倒是没有针对ForwardIterator而设计的版本，因为那和针对InputIterator而设计的版本完全一致。

```c++
template <class InputIterator, class Distance>
voidadvance_II (InputIterator& i, Distance n)
{
// 单向，逐一前进
while (n--) ++i; //或写 for ( ; n > 0; --n, ++i );
}

template <class BidirectionalIterator, class Distance>
voidadvance_BI (BidirectionalIterator& i, Distance n)
{
    // 双向，逐一前进
    if (n >= 0)
        while (n--) ++i; //或写 for ( ; n > 0; --n, ++i );
    else
        while (n++) --i; //或写 for ( ; n < 0; ++n, --i );
}

template <class RandomAccessIterator, class Distance>
voidadvance_RAI (RandomAccessIterator& i, Distance n)
{
    // 双向，跳跃前进
    i += n;
}
```

现在，当程序呼叫 advance() ，应该选用（呼叫）哪一份函式定义呢？如果选择advance_II() ，对Random Access Iterator而言极度缺乏效率，原本O(1)的操作竟成为O(N)。如果选择 advance_RAI() ，则它无法接受Input Iterator。我们需要将三者合一，下面是一种作法：

```c++
template <class InputIterator, class Distance>
void advance (InputIterator& i, Distance n)
{
    if (is_random_access_iterator(i))
        advance_RAI(i, n); // 此函式有待设计
    else if (is_bidirectional_iterator (i))
        advance_BI(i, n); // 此函式有待设计
    else
        advance_II(i, n);
}
```

但是像这样在执行时期才决定使用哪一个版本，会影响程序效率。最好能够在编译期就选择正确的版本。多载化函式机制可以达成这个目标。前述三个 advance_xx() 都有两个函式参数，型别都未定（因为都是 template参数）。为了令其同名，形成多载化函式，我们必须加上一个型别已确定的函式参数，使函式多载化机制得以有效运作起来。设计考虑如下：如果traits有能力萃取出迭代器的种类，我们便可利用这个「迭代器类型」相应型别做为 advanced() 的第三参数。这个相应型别一定必须是个class type，不能只是数值号码类的东西，因为编译器需仰赖（一个型别）来进行多载化决议程序（overloaded resolution）。下面定义五个 classes，代表五种迭代器类型：

```c++
//五个做为标记用的型别（tag types）
structinput_iterator_tag { };
structoutput_iterator_tag { };
structforward_iterator_tag : public input_iterator_tag { };
structbidirectional_iterator_tag : public forward_iterator_tag { };
struct random_access_iterator_tag : publicbidirectional_iterator_tag { };
```

这些 classes只做为标记用，所以不需要任何成员。至于为什么运用继承机制，稍后再解释。现在重新设计 __ advance() （由于只在内部使用，所以函式名称加上特定的前导符），并加上第三参数，使它们形成多载化：

```c++
template <class InputIterator, class Distance>
inline void __advance (InputIterator& i, Distance n,
input_iterator_tag)
{
    // 单向，逐一前进
    while (n--) ++i;
}

//这是一个单纯的转呼叫函式（ trivial forwarding function ）。稍后讨论如何免除之。
template <class ForwardIterator, class Distance>
inline void __advance (ForwardIterator& i, Distance n,
forward_iterator_tag)
{
    advance(i, n, input_iterator_tag());
}

// 单纯地进行转呼叫（ forwarding ）
template <class BidiectionalIterator, class Distance>
inline void __advance (BidiectionalIterator& i, Distance n,
bidirectional_iterator_tag)
{
    // 双向，逐一前进
    if (n >= 0)
        while (n--) ++i;
    else
        while (n++) --i;
}

template <class RandomAccessIterator, class Distance>
inline void __advance (RandomAccessIterator& i, Distance n,
random_access_iterator_tag)
{
    // 双向，跳跃前进
    i += n;
}
```

注意上述语法，每个 __ advance() 的最后一个参数都只宣告型别，并未指定参数名称，因为它纯粹只是用来启动多载化机制，函式之中根本不使用该参数。如果硬要加上参数名称也可以，画蛇添足罢了。

行 进 至 此 ， 还 需 要 一 个 对 外 开 放 的 上 层 控 制 介 面 ， 呼 叫 上 述 各 个 多 载 化 的__ advance() 。 此 一 上 层 介 面 只 需 两 个 参 数 ， 当 它 准 备 将 工 作 转 给 上 述 的__ advance() 时，才自行加上第三自变量：迭代器类型。因此，这个上层函式必须有能力从它所获得的迭代器中推导出其类型—这份工作自然是交给 traits 机制：

```c++
template <class InputIterator, class Distance>
inline void advance (InputIterator& i, Distance n)
{
    __advance(i, n, iterator_traits< InputIterator >::iterator_category()); 3
}
```

注意上述语法， `iterator_traits<Iterator>::iterator_category()` 将产生一个暂时对象（道理就像 int() 会产生一个 int 暂时对象一样），其型别应该隶属前述五个迭代器类型之一。然后，根据这个型别，编译器才决定呼叫哪一个__ advance() 多载函式。因此，为了满足上述行为，traits必须再增加一个相应型别：

```c++
template <class I>
struct iterator_traits {
    ...
    typedef typename I::iterator_category iterator_category;
};

//针对原生指标而设计的「偏特化版（ partial specialization ）」
template <class T>
struct iterator_traits<T*> {
    ...
    // 注意，原生指标是一种 Random Access Iterator
    typedef random_access_iterator_tag iterator_category;
};

//针对原生的 pointer-to- const 而设计的「偏特化版（ partial specialization ）」
template <class T>
struct iterator_traits<const T*>
    ...
    // 注意，原生的 pointer-to- const是一种 Random Access Iterator
    typedef random_access_iterator_tag iterator_category;
};
```

任何一个迭代器，其类型永远应该落在「该迭代器所隶属之各种类型中，最强化的那个」。例如 int* 既是Random Access Iterator又是 Bidirectional Iterator，同时也是Forward Iterator，而且也是Input Iterator，那么，其类型应该归属为
random_access_iterator_tag。你是否注意到 advance() 的 template参数名称取得好像不怎么理想：

```c++
template <class InputIterator , class Distance>
inline void advance (InputIterator& i, Distance n);
```

按说 advanced() 既然可以接受各种类型的迭代器，就不应将其型别参数命名为InputIterator 。这其实是 STL 算法的一个命名规则：**以算法所能接受之最低阶迭代器类型，来为其迭代器型别参数命名**。

##### 消除“单纯传递调用的函数”

以 class 来定义迭代器的各种分类标签，不唯可以促成多载化机制的成功运作（使
编译器得以正确执行多载化决议程序，overloaded resolution），另一个好处是，透过继承，我们可以不必再写「单纯只做转呼叫」的函式（例如前述的 advance()
ForwardIterator版）。为什么能够如此？考虑下面这个小例子，从其输出结果可
以看出端倪：

```c++
// file: 3tag-test.cpp
//模拟测试 tag types 继承关系所带来的影响。
#include <iostream>
using namespace std;
struct B { }; // B 可比拟为 InputIterator
struct D1 : public B { }; // D1 可比拟为 ForwardIterator
struct D2 : public D1 { }; // D2 可比拟为 BidirectionalIterator

template <class I>
func(I& p, B)
{ cout << "B version" << endl; }

template <class I>
func(I& p, D2)
{ cout << "D2 version" << endl; }

int main()
{
    int* p;
    func(p, B()); // 参数与自变量完全吻合。输出 : "B version"
    func(p, D1()); // 参数与自变量未能完全吻合；因继承关系而自动转呼叫。
                    //输出:"B version"
    func(p, D2()); // 参数与自变量完全吻合。输出 : "D2 version"
}
```

##### 以 distance()为例

关于「迭代器类型标签」的应用，以下再举一例。 distance() 也是常用的一个迭代器操作函式，用来计算两个迭代器之间的距离。针对不同的迭代器类型，它
可以有不同的计算方式，带来不同的效率。整个设计模式和前述的 advance() 如
出一辙：

```c++
template <class InputIterator>
inline iterator_traits<InputIterator>::difference_type
__distance (InputIterator first, InputIterator last,
input_iterator_tag ) {
iterator_traits<InputIterator>::difference_type n = 0;
    // 逐一累计距离
    while (first != last) {
        ++first; ++n;
    }
    return n;
}

template <class RandomAccessIterator>
inline iterator_traits<RandomAccessIterator>::difference_type
__distance (RandomAccessIterator first, RandomAccessIterator last,
random_access_iterator_tag ) {
    // 直接计算差距
    return last - first;
}

template <class InputIterator>
inline iterator_traits<InputIterator>::difference_type
distance (InputIterator first, InputIterator last) {
typedef typename iterator_traits<InputIterator>::iterator_category category;
    return __distance(first, last, category());
}
```

注意， distance() 可接受任何类型的迭代器；其 template型别参数之所以命名为 InputIterator ，是为了遵循STL 算法的命名规则：**以算法所能接受之最初级类型来为其迭代器型别参数命名**。此外也请注意，由于迭代器类型之间存在着继承关系，「转呼叫（forwarding）」的行为模式因此自然存在—这一点我已在前一节讨论过。换句话说，当客端呼叫 distance() 并使用 Output Iterators或 Forward Iterators 或Bidirectional Iterators，统统都会转呼叫 Input Iterator版的那个 __distance() 函式。

### std::iterator的保证

为了符合规范，任何迭代器都应该提供五个巢状相应型别，以利traits萃取，否则便是自外于整个STL架构，可能无法与其它 STL 组件顺利搭配。然而写码难免挂一漏万，谁也不能保证不会有粗心大意的时候。如果能够将事情简化，就好多了。STL提供了一个 iterators class如下，如果每个新设计的迭代器都继承自它，就保证符合 STL 所需之规范：

```c++
template <class Category,
    class T,
    class Distance = ptrdiff_t,
    class Pointer = T*,
    class Reference = T&>
struct iterator {
    typedef Category iterator_category;
    typedef T value_type;
    typedef Distance difference_type;
    typedef Pointer pointer;
    typedef Reference reference;
};
```

iterator class不含任何成员，纯粹只是型别定义，所以继承它并不会招致任何额外负担。由于后三个参数皆有默认值，新的迭代器只需提供前两个参数即可。

比如自己写的ListIter，如果采用这种写法，应该这样写

```c++
template <class Item>
struct ListIter :
public std::iterator<std::forward_iterator_tag, Item>
{ ... }
```

### 总结

设计适当的相应型别（associated types），是迭代器的责任。设计适当的迭代器，则是容器的责任。唯容器本身，才知道该设计出怎样的迭代器来走访自己，并执行迭代器该有的各种行为（前进、后退、取值、取用成员…）。至于算法，完全可以独立于容器和迭代器之外自行发展，只要设计时以迭代器为对外接口就行。

traits编程技法，大量运用于 STL 实作品中。它利用「巢状型别」的写码技巧与编译器的template自变量推导功能，补强 C++未能提供的关于型别认证方面的能力，补强 C++不为强型（strong typed）语言的遗憾。了解traits编程技法，就像获得「芝麻开门」口诀一样，从此得以一窥 STL 源码堂奥。

### 3.6 iterator 源码完整重列

### 3.7 SGI STL 的私房菜 ： __type_traits

traits编程技法很棒，适度弥补了 C++ 语言本身的不足。STL只对迭代器加以规范，制定出 iterator_traits 这样的东西。SGI 把这种技法进一步扩大到迭代器以外的世界，于是有了所谓的 __type_traits 。双底线前缀词意指这是SGISTL 内部所用的东西，不在 STL 标准规范之内。

**iterator_traits 负责萃取迭代器的特性， __type_traits 则负责萃取型别（type）的特性**。此处我们所关注的型别特性是指：这个型别是否具备non-trivial defalt ctor ？是否具备 non-trivial copy ctor？是否具备 non-trivial assignment operator？是否具备 non-trivialdtor？如果答案是否定的，我们在对这个型别进行建构、解构、拷贝、赋值等动作时，就可以采用最有效率的措施（例如根本不唤起尸位素餐的那些constructor, destructor），而采用内存直接处理动作 如malloc() 、 memcpy() 等等，获得最高效率。这对于大规模而动作频繁的容器，有着显著的效率提升。

定义于 SGI <type_traits.h> 中的 __type_traits ，提供了一种机制，允许针对不同的型别属性（type attributes），在编译时期完成函式派送决定（function dispatch）。这对于撰写 template很有帮助，例如，当我们准备对一个「元素型别未知」的数组执行 copy 动作时，如果我们能事先知道其元素型别是否有一个 trivialcopy constructor ， 便 能 够 帮 助 我 们 决 定 是 否 可 使 用 快 速 的 memcpy() 或memmove() 。从 iterator_traits 得 来 的 经 验 ， 我 们 希 望 ， 程 式 之 中 可 以 这 样 运 用

```c++
__type_traits<T> ， T 代表任意型别：
__type_traits<T>::has_trivial_default_constructor
__type_traits<T>::has_trivial_copy_constructor
__type_traits<T>::has_trivial_assignment_operator
__type_traits<T>::has_trivial_destructor
__type_traits<T>::is_POD_type // POD : Plain Old Data
```

我们希望上述式子响应我们「真」或「假」（以便我们决定采取什么策略），但其结果不应该只是个 bool 值，应该是个有着真/假性质的「对象」，因为我们希望利用其响应结果来进行自变量推导，而编译器只有面对 class object形式的自变量，才会做自变量推导。为此，上述式子应该传回这样的东西：

```c++struct__true_type { };
struct__false_type { };
```

这两个空白 classes没有任何成员，不会带来额外负担，却又能够标示真假，满足我们所需。

为了达成上述五个式子， __type_traits 内必须定义一些typedefs，其值不是__true_type 就是 __false_type 。下面是 SGI的作法：

```c++
template <class type>
struct__type_traits
typedef __true_type this_dummy_member_must_be_first ;
    /* 不要移除这个成员。它通知「有能力自动将 __type_traits 特化」
    的编译器说，我们现在所看到的这个 __type_traits template 是特
    殊的。这是为了确保万一编译器也使用一个名为 __type_traits 而其
    实与此处定义并无任何关联的 template 时，所有事情都仍将顺利运作。
    */
/* 以下条件应被遵守，因为编译器有可能自动为各型别产生专属的 __type_traits
    特化版本：
    - 你可以重新排列以下的成员次序
    - 你可以移除以下任何成员
    - 绝对不可以将以下成员重新命名而却没有改变编译器中的对应名称
    - 新加入的成员会被视为一般成员，除非你在编译器中加上适当支持。*/
typedef __false_type has_trivial_default_constructor ;
typedef __false_type has_trivial_copy_constructor ;
typedef __false_type has_trivial_assignment_operator ;
typedef __false_type has_trivial_destructor ;
typedef __false_type is_POD_type;
};
```

为什么SGI 把所有巢状型别都定义为 __false _type 呢？是的，SGI 定义出最保守的值，然后（稍后可见）再针对每一个纯量型别（scalar types）设计适当的__type_traits 特化版本，这样就解决了问题。上述 __type_traits 可以接受任何型别的自变量，五个typedefs将经由以下管道获得实值：

- 一般具现体（general instantiation），内含对所有型别都必定有效的保守值。上述各个 has_trivial_xxx 型别都被定义为 __false_type ，就是对所有型别都必定有效的保守值。
- 经过宣告的特化版本，例如 <type_traits.h> 内对所有 C++纯量型别（scalar types）提供了对映的特化宣告。稍后展示。
- 某些编译器（如 Silicon Graphics N32和 N64编译器）会自动为所有型别提供适当的特化版本。（这真是了不起的技术。不过我对其精确程度存疑）

__types_traits 在SGI STL中的应用很广。下面我举几个实例。第一个例子是出现于本书 2.3.3节的 uninitialized_fill_n() 全域函式：

```c++
template <class ForwardIterator, class Size, class T>
inline ForwardIterator uninitialized_fill_n (ForwardIterator first,
Size n , const T& x ) {
    return __uninitialized_fill_n(first, n, x, value_type(first) );
}
```

此函式以 x为蓝本，自迭代器 first 开始建构 n个元素。为求取最大效率，首先 以 value_type() （ 3.6节 ） 萃 取 出 迭 代 器 first 的value type， 再 利 用__type_traits 判断该型别是否为 POD型别：

```c++
template <class ForwardIterator, class Size, class T, class T1>
inline ForwardIterator __uninitialized_fill_n (ForwardIterator first,
Size n, const T& x, T1*)
{
    typedef typename __type_traits<T1>::is_POD_type is_POD;
    return __uninitialized_fill_n_aux(first, n, x, is_POD());
}
```

以下就「是否为 POD型别」采取最适当的措施：

```c++
//如果不是 POD型别，就会派送（ dispatch ）到这里
template <class ForwardIterator, class Size, class T>
ForwardIterator
__uninitialized_fill_n_aux (ForwardIterator first, Size n,
const T& x, __false_type ) {
ForwardIterator cur = first;
    // 为求阅读顺畅简化，以下将原本有的异常处理（ exception handling ）去除。
    for ( ; n > 0; --n, ++cur)
        construct(&*cur, x); //见 2.2.3节
    return cur;
}

//如果是 POD型别，就会派送（ dispatch ）到这里。下两行是原文件所附注解。
//如果 copy construction 等同于 assignment ，而且有 trivial destructor ，
//以下就有效。
template <class ForwardIterator, class Size, class T>
inline ForwardIterator
__uninitialized_fill_n_aux (ForwardIterator first, Size n,
const T& x, __true_type ) {
    return fill_n (first, n, x); //交由高阶函式执行，如下所示。
}

//以下是定义于 <stl_algobase.h> 中的 fill_n()
template <class OutputIterator, class Size, class T>
OutputIterator fill_n (OutputIterator first, Size n, const T& value) {
    for ( ; n > 0; --n, ++first)
        *first = value;
    return first;
}
```

第二个例子是负责对象解构的 destroy() 全域函式。此函式之源码及解说在 2.2.3节有完整的说明。

第三个例子是出现于本书第6章的 copy() 全域函式（泛型算法之一）。这个函式有非常多的特化（specialization）与强化（refinement）版本，殚精竭虑，全都是为了效率考虑，希望在适当的情况下采用最「雷霆万钧」的手段。最基本的想法是这样：

```c++
//拷贝一个数组，其元素为任意型别，视情况采用最有效率的拷贝手段。
template <class T> inline void copy (T* source,T* destination,int n) {
    copy(source,destination,n,
        typename __type_traits<T>::has_trivial_copy_constructor() );
}

//拷贝一个数组，其元素型别拥有 non-trivial copy constructors 。
template <class T> void copy (T* source,T* destination,int n, __false_type)
{ ... }

//拷贝一个数组，其元素型别拥有 trivial copy constructors 。
//可借助 memcpy() 完成工作
template <class T> void copy (T* source,T* destination,int n, __true_type)
{ ... }
```

以上只是针对「函式参数为原生指标」的情况而做的设计。第6章的 copy() 演算法是个泛型版本，情况又复杂许多。详见 6.4.3节。

因 此 如 果 你 是 SGI STL的 使 用 者 ， 你 可 以 在 自 己 的 程 式 中 充 份 运 用 这 个__type_traits。加入编译器足够厉害，可以萃取出来我的Shape是否有 trivial defalt ctor或trivial copy ctor或trivial assignment operator或 trivial dtor，但如果不够厉害，萃取出来的都是__false_type，这样的结果未免过于保守，所以最保险的方法是自行设计一个__type_traits特化版本，明白地告诉编译器：

```c++
template<>struct __type_traits<Shape> {
    typedef __true_type has_trivial_default_constructor;
    typedef __false_type has_trivial_copy_constructor;
    typedef __false_type has_trivial_assignment_operator;
    typedef __false_type has_trivial_destructor;
    typedef __false_type is_POD_type;
};
```

究竟一个 class什么时候该有自己的 non-trivial default constructor, non-trivial copy constructor, non-trivial assignment operator, non-trivial destructor 呢？一个简单的判断准则是：如果 class 内含指标成员，并且对它进行内存动态配置，那么这个class就需要实作出自己的 non-trivial-xxx。

即使你无法全面针对你自己定义的型别，设计 __type_traits 特化版本，无论如何，至少，有了这个 __type_traits 之后，当我们设计新的泛型算法时，面对C++纯量型别，便有足够的信息决定采用最有效的拷贝动作或赋值动作—因为每一个纯量型别都有对应的 __type_traits 特化版本，其中每一个 typedef 的值都是 __true_type 。

> type traits 类型特征
从字面上理解，Type Traits 就是” 类型的特征” 的意思。在 C++ 元编程中，程序员不少时候都需要了解一些类型的特征信息，并根据这些类型信息选择应有的操作。Type Traits 有助于编写通用、可复用的代码。

[推荐阅读](https://blog.csdn.net/mogoweb/article/details/79264925)

## Chapter4 序列式容器 sequence containers

### 4.1 容器的概观与分类

容器，置物之所也。

序列式容器sequence containers  研究数据的特定排列方式，以利搜寻或排序或其它特殊目的，这一专门学科我们称为数据结构（Data Structures）。大学信息相关教育里头，与编程最有直接关系的科目，首推数据结构与算法（Algorithms）。几乎可以说，任何特定的数据结构都是为了实现某种特定的算法。STL 容器即是将运用最广的一些数据结构实作出来（图4-1）。未来，在每五年召开一次的C++标准委员会中，STL容器的数量还有可能增加。众所周知，常用的数据结构不外乎 array（数组）、list（串行）、tree（树）、stack（堆栈）、queue（队列）、hash table（杂凑表）、set（集合）、map（映像表）…等等。根据「资料在容器中的排列」特性，这些数据结构分为序列式（sequence）和关系型（associative）两种。本章探讨序列式容器，下一章探讨关系型容器。

![《STL源码剖析》的笔记-stlcontainers.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-stlcontainers.png)

这里所谓的衍生，并非继承（inheritance）关系，而是内含（containment）关系。例如 heap 内含一个 vector，priority-queue 内含一个 heap，stack 和 queue 都含一个deque，set/map/multiset/multimap 都内含一个 RB-tree，hast_x都内含一个 hashtable。

### 4.2 vector

#### 4.2.1 vector 概述

vector 的数据安排以及操作方式，与 array 非常像似。两者的唯一差别在于空间的运用弹性。 **array 是静态空间**，一旦配置了就不能改变；要换个大（或小）一点的房子，可以，一切细琐得由客端自己来：首先配置一块新空间，然后将元素从旧址一一搬往新址，然后再把原来的空间释还给系统。 **vector 是动态空间**，随着元素的加入，它的内部机制会自行扩充空间以容纳新元素。因此， vector 的运用对于内存的樽节与运用弹性有很大的帮助，我们再也不必因为害怕空间不足而一开始就要求一个大块头 array 了，我们可以安心使用 vector ，吃多少用多少。

vector 的实作技术，关键在于其对大小的控制以及重新配置时的数据搬移效率。一旦 vector 旧有空间满载，如果客端每新增一个元素， vector 内部只是扩充一个元素的空间，实为不智，因为所谓扩充空间（不论多大），一如稍早所说，是**配置新空间 /数据搬移 /释还旧空间**的大工程，时间成本很高，应该加入某种未雨绸缪的考虑。稍后我们便可看到 SGI vector 的空间配置策略。

#### 4.2.2 vector 定义式摘要

以下是 vector 定义式的源码摘录。虽然 STL规定，欲使用 vector 者必须先含入 `<vector>` ，但 SGI STL 将 vector 实作于更底层的 `<stl_vector.h>` 。

```c++
// alloc是 SGI STL的空间配置器，见第二章。
template <class T, class Alloc = alloc>
class vector {
public:
// vector 的巢状型别定义
typedef T value_type;
typedef value_type* pointer;
typedef value_type* iterator;
typedef value_type& reference;
typedef size_t size_type;
typedef ptrdiff_tdifference_type;
protected:
// 以下， simple_alloc 是 SGI STL的空间配置器，见 2.2.4节。
typedef simple_alloc<value_type,Alloc>data_allocator;
iterator start; //表示目前使用空间的头
iterator finish; //表示目前使用空间的尾
iterator end_of_storage; //表示目前可用空间的尾
void insert_aux (iterator position, const T& x);
void deallocate () {
    if (start)
        data_allocator::deallocate(start, end_of_storage - start);
}
void fill_initialize (size_type n, const T& value) {
    start =allocate_and_fill (n, value);
    finish = start + n;
    end_of_storage = finish;
}
public:
iterator begin () { return start; }
iterator end () { return finish; }
size_type size () const { return size_type(end() - begin()); }
size_type capacity () const {
return size_type(end_of_storage - begin()); }
bool empty () const { return begin() == end(); }
reference operator[] (size_type n) { return *(begin() + n); }
vector () : start(0), finish(0), end_of_storage(0) {}
vector (size_type n, const T& value) {fill_initialize(n, value); }
vector (int n, const T& value) { fill_initialize(n, value); }
vector (long n, const T& value) { fill_initialize(n, value); }
explicit vector (size_type n) { fill_initialize(n, T()); }
~vector()
    destroy (start, finish); //全域函式，见 2.2.3节。
    deallocate(); // 这是 vector 的一个 member function
}
reference front () { return * begin() ; } //第一个元素
reference back () { return *( end() - 1); } //最后一个元素
void push_back (const T& x) {    //将元素安插至最尾端
    if (finish != end_of_storage) {
        construct (finish, x); //全域函式，见 2.2.3节。
        ++finish;
    }
    else
        insert_aux(end(), x); // 这是 vector 的一个 member function
}
void pop_back () {    //将最尾端元素取出
    --finish;
    destroy(finish); //全域函式，见 2.2.3节。
}
iterator erase (iterator position) { //清除某位置上的元素
    if (position + 1 != end())
        copy (position + 1, finish, position); //后续元素往前搬移
    --finish;
    destroy(finish); //全域函式，见 2.2.3节。
    return position;
}
void resize (size_type new_size, const T& x) {
    if (new_size < size())
        erase (begin() + new_size, end());
    else
        insert (end(), new_size - size(), x);
}
void resize (size_type new_size) { resize (new_size, T()); }
void clear () { erase(begin(), end()); }
protected:
// 配置空间并填满内容
iterator allocate_and_fill (size_type n, const T& x) {
    iterator result =data_allocator::allocate (n);
    uninitialized_fill_n(result, n, x); // 全域函式，见 2.3 节
    return result;
}
```

#### 4.2.3 vector 的迭代器

vector 维护的是一个连续线性空间，所以不论其元素型别为何，原生指标都可以做为 vector 的迭代器而满足所有必要条件，因为 vector 迭代器所需要的操作行为如 operator*,operator->,operator++,operator--,operator+, operator-, operator+=,operator-= ，原生指标天生就具备。 vector 支援随机存取，而原生指标正有着这样的能力。所以，**vector 提供的是 Random Access Iterators**。

```c++
template <class T, class Alloc = alloc>
class vector {
public:
typedef T value_type;
typedef value_type* iterator;
...
};
```

根据上述定义，如果客端写出这样的代码：

```c++
// vector 的迭代器是原生指标
vector<int>::iterator ivite;
vector<Shape>::iterator svite;
```

ivite 的型别其实就是 int* ， svite 的型别其实就是 Shape* 。

#### 4.2.4 vector 的数据结构

vector 所采用的数据结构非常简单：线性连续空间。它以两个迭代器 start 和 finish 分 别 指 向 配 置 得 来 的 连 续 空 间 中 目 前 已 被 使 用 的 范 围 ， 并 以 迭 代 器 end_of_storage 指向整块连续空间（含备用空间）的尾端：

```c++
template <class T, class Alloc = alloc>
class vector {
...
protected:
iterator start;
iterator finish;
//表示目前使用空间的头
//表示目前使用空间的尾
iterator end_of_storage; //表示目前可用空间的尾
...
};
```

为了降低空间配置时的速度成本， vector 实际配置的大小可能比客端需求量更大一些，以备将来可能的扩充。这便是**容量（capacity）**的观念。换句话说**一个vector的容量永远大于或等于其大小**。一旦容量等于大小，便是满载，下次再有新增元素，整个 vector 就得另觅居所。见图 4-2。

运用 start, finish, end_of_storage 三个迭代器，便可轻易提供首尾标示、大小、容量、空容器判断、注标（ [ ] ）运算子、最前端元素值、最后端元素值…等机能：

```c++
template <class T, class Alloc = alloc>
class vector {
...
public:
iterator begin () { return start ; }
iterator end () { return finish ; }
size_type size () const { return size_type(end() - begin()); }
size_type capacity () const {
    return size_type(end_of_storage - begin()); }
bool empty () const { return begin() == end(); }
reference operator[] (size_type n) { return*(begin() + n); }
reference front () { return *begin() ; }
reference back () { return *(end() - 1) ; }
...
};
```

![《STL源码剖析》的笔记-vectords.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-vectords.png)

#### 4.2.5 vector 的建构与内存管理 ： constructor, push_back

下面是个小小的测试程序，我的观察重点在建构的方式、元素的添加，以及大小、容量的变化：

```c++
// filename : 4vector-test.cpp
#include <vector>
#include <iostream>
#include <algorithm>
using namespace std;
int main()
{
    int i;
    vector<int> iv(2,9);
    cout << "size=" << iv. size() << endl; // size=2
    cout << "capacity=" << iv. capacity() << endl; // capacity=2
    iv.push_back(1);
    cout << "size=" << iv.size() << endl; // size=3
    cout << "capacity=" << iv.capacity() << endl; // capacity=4
    iv.push_back(2);
    cout << "size=" << iv.size() << endl; // size=4
    cout << "capacity=" << iv.capacity() << endl; // capacity=4
    iv.push_back(3);
    cout << "size=" << iv.size() << endl; // size=5
    cout << "capacity=" << iv.capacity() << endl; // capacity=8
    iv.push_back(4);
    cout << "size=" << iv.size() << endl; // size=6
    cout << "capacity=" << iv.capacity() << endl; // capacity=8

    for(i=0; i<iv.size(); ++i)
        cout << iv[i] << ' ';     // 9 9 1 2 3 4
    cout << endl;

    iv.push_back(5);
    cout << "size=" << iv.size() << endl; // size=7
    cout << "capacity=" << iv.capacity() << endl; // capacity=8

    for(i=0; i<iv.size(); ++i)
        cout << iv[i] << ' ';    // 9 9 1 2 3 4 5
    cout << endl;

    iv.pop_back();
    iv.pop_back();
    cout << "size=" << iv.size() << endl; // size=5
    cout << "capacity=" << iv.capacity() << endl; // capacity=8
    iv.pop_back();
    cout << "size=" << iv.size() << endl; // size=4
    cout << "capacity=" << iv.capacity() << endl; // capacity=8
    vector<int>::iterator ivite = find (iv.begin(), iv.end(), 1);
    if (ivite) iv.erase(ivite);
    cout << "size=" << iv.size() << endl; // size=3
    cout << "capacity=" << iv.capacity() << endl; // capacity=8

    for(i=0; i<iv.size(); ++i)
        cout << iv[i] << ' ';     // 9 9 2
    cout << endl;

    ite = find (ivec.begin(), ivec.end(), 2);
    if (ite) ivec. insert(ite,3,7);
    cout << "size=" << iv.size() << endl; // size=6
    cout << "capacity=" << iv.capacity() << endl; // capacity=8

    for(int i=0; i<ivec.size(); ++i)
        cout << ivec[i] << ' '; // 9 9 7 7 7 2
    cout << endl;

    iv.clear();
    cout << "size=" << iv.size() << endl; // size=0
    cout << "capacity=" << iv.capacity() << endl; // capacity=8
}
```

vector 预设使用 alloc （第二章）做为空间配置器，并据此另外定义了一个data_allocator ，为的是更方便以元素大小为配置单位

```c++
template <class T, class Alloc = alloc>
class vector {
protected:
    // simple_alloc<> 见 2.2.4 节
    typedef simple_alloc<value_type,Alloc>data_allocator;
...
};
```

于是， data_allocator::allocate(n) 表示配置 n 个元素空间。vector 提供许多constructors，其中一个允许我们指定空间大小及初值：

```c++
// 建构式，允许指定 vector 大小 n和初值 value
vector (size_type n, const T& value) {fill_initialize(n, value); }
// 充填并予初始化
void fill_initialize (size_type n, const T& value) {
    start =allocate_and_fill (n, value);
    finish = start + n;
    end_of_storage = finish;
}
// 配置而后充填
iterator allocate_and_fill (size_type n, const T& x) {
iterator result =data_allocator::allocate(n) ; // 配置 n 个元素空间
uninitialized_fill_n(result, n, x); // 全域函式，见 2.3 节
return result;
}
```

uninitialized_fill_n() 会根据第一参数的型别特性（type traits，3.7 节），决定使用算法 fill_n() 或反复呼叫 construct() 来完成任务（见 2.3 节描述）。当我们以 push_back() 将新元素安插于 vector 尾端，该函式首先检查是否还有备用空间？如果有就直接在备用空间上建构元素，并调整迭代器 finish ，使 vector变大。如果没有备用空间了，就扩充空间（重新配置、搬移数据、释放原空间）：

```c++
void push_back (const T& x) {
    if (finish != end_of_storage) { //还有备用空间
        construct (finish, x);//全域函式，见 2.2.3节。
        ++finish;//调整水位高度
}
else
    //已无备用空间
    insert_aux(end(), x); // vector member function ，见以下列表
}

template <class T, class Alloc>
void vector<T, Alloc>:: insert_aux (iterator position, const T& x) {
    if (finish != end_of_storage) { //还有备用空间
        // 在备用空间起始处建构一个元素，并以 vector 最后一个元素值为其初值。
        construct (finish, *(finish - 1));
        // 调整水位。
        ++finish;
        T x_copy = x;
        copy_backward(position, finish - 2, finish - 1);
        *position = x_copy;
    }
    else { //已无备用空间
        const size_type old_size = size();
        const size_type len = old_size != 0 ? 2 * old_size : 1;
        // 以上配置原则：如果原大小为 0，则配置 1（个元素大小）；
        // 如果原大小不为 0，则配置原大小的两倍，
        // 前半段用来放置原资料，后半段准备用来放置新资料。
        iterator new_start =data_allocator::allocate (len); //实际配置
        iterator new_finish = new_start;
        try {
            // 将原 vector 的内容拷贝到新 vector。start是vector的成员，是一个iterator，new_start也是一个iterator，指向刚allocate出来的地址空间首位
            new_finish = uninitialized_copy(start, position, new_start);
            // 为新元素设定初值 x
            construct (new_finish, x);
            // 调整水位。
            ++new_finish;
            // 将原 vector 的备用空间中的内容也忠实拷贝过来（侯捷疑惑：啥用途？）（简体书解答：本函数有可能被insert(p,x)调用）（个人理解：如果插入位置position不等于finish，则会用到这里，需要把position后面的元素copy到新地址去）
            new_finish =uninitialized_copy(position, finish, new_finish);
        }
        catch(...) {
            // "commit or rollback" semantics.
            destroy (new_start, new_finish);
            data_allocator::deallocate(new_start, len);
            throw;
        }
        // 解构并释放原 vector
        destroy (begin(), end());
        deallocate();
        // 调整迭代器，指向新 vector
        start = new_start;
        finish = new_finish;
        end_of_storage = new_start + len;
    }
}
```

注意，所谓动态增加大小，并不是在原空间之后接续新空间（因为无法保证原空间之后尚有可供配置的空间），而是以原大小的两倍另外配置一块较大空间，然后将原内容拷贝过来，然后才开始在原内容之后建构新元素，并释放原空间。因此，对 vector 的任何操作，一旦引起空间重新配置，指向原 vector 的所有迭代器就都失效了。这是程序员易犯的一个错误，务需小心。

#### 4.2.6 vector 的元素操作 ： pop_back, erase, clear, insert

vector 所提供的元素操作动作很多，无法在有限篇幅中一一讲解—其实也没有这种必要。为搭配先前对空间配置的讨论，我挑选数个相关函式做为解说对象。这些函式也出现在先前的测试程序中。

```c++
// 将尾端元素拿掉，并调整大小。
void pop_back () {
    --finish; //将尾端标记往前移一格，表示将放弃尾端元素。
    destroy(finish); // destroy是全域函式，见第 2 章
}
// 清除 [first,last) 中的所有元素
iterator erase (iterator first, iterator last) {
    iterator i = copy (last, finish, first); // copy 是全域函式，第 6 章
    destroy(i, finish);// destroy是全域函式，第 2 章
    finish = finish - (last - first);
    return first;
}
// 清除某个位置上的元素
iterator erase (iterator position) {
    if (position + 1 != end())
        copy (position + 1, finish, position); // copy 是全域函式，第 6 章
    --finish;
    destroy(finish); // destroy是全域函式，2.2.3 节
    return position;
}
void clear () { erase (begin(), end()); } // erase()就定义在上面
```

图 4-3a 展示 erase(first, last) 的动作。

![《STL源码剖析》的笔记-vectoreraserange.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-vectoreraserange.png)

下面是 vector::insert() 实作内容：

```c++
//从 position 开始，安插 n个元素，元素初值为 x
template <class T, class Alloc>
void vector<T, Alloc>:: insert (iterator position, size_type n, const T& x)
{
    if (n != 0) { // 当 n != 0 才进行以下所有动作
        if (size_type(end_of_storage - finish) >= n)
            // 备用空间大于等于「新增元素个数」
            T x_copy = x;
            // 以下计算安插点之后的现有元素个数
            const size_type elems_after = finish - position;
            iterator old_finish = finish;
            if (elems_after > n)
                // 「安插点之后的现有元素个数」大于「新增元素个数」
                uninitialized_copy(finish - n, finish, finish);
                finish += n; //将 vector 尾端标记后移
                copy_backward(position, old_finish - n, old_finish);
                fill (position, position + n, x_copy); //从安插点开始填入新值
            }
            else {
                // 「安插点之后的现有元素个数」小于等于「新增元素个数」
                uninitialized_fill_n(finish, n - elems_after, x_copy);
                finish += n - elems_after;
                uninitialized_copy(position, old_finish, finish);
                finish += elems_after;
                fill (position, old_finish, x_copy);
            }
        }
        else {
            // 备用空间小于「新增元素个数」（那就必须配置额外的内存）
            // 首先决定新长度：旧长度的两倍，或旧长度+新增元素个数。
            const size_type old_size = size();
            const size_type len = old_size + max(old_size, n);
            // 以下配置新的 vector 空间
            iterator new_start = data_allocator::allocate (len);
            iterator new_finish = new_start;
            __STL_TRY {
                // 以下首先将旧 vector的安插点之前的元素复制到新空间。
                new_finish = uninitialized_copy(start, position, new_start);
                // 以下再将新增元素（初值皆为 n）填入新空间。
                new_finish = uninitialized_fill_n(new_finish, n, x);
                // 以下再将旧 vector 的安插点之后的元素复制到新空间。
                new_finish = uninitialized_copy(position, finish, new_finish);
            }
            # ifdef __STL_USE_EXCEPTIONS
                catch(...) {
                // 如有异常发生，实现 "commit or rollback" semantics.
                destroy (new_start, new_finish);
                data_allocator::deallocate(new_start, len);
                throw;
            }
            # endif /* __STL_USE_EXCEPTIONS */
            // 以下清除并释放旧的 vector
            destroy (start, finish);
            deallocate();
            // 以下调整水位标记
            start = new_start;
            finish = new_finish;
            end_of_storage = new_start + len;
        }
    }
}
```

注意，安插完成后，新节点将位于标兵迭代器（上例之 position ，标示出安插点）所指之节点的前方—这是STL对于「安插动作」的标准规范。图4-3b展示insert(position,n,x) 的动作。

![《STL源码剖析》的笔记-vectorinsert1.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-vectorinsert1.png)

![《STL源码剖析》的笔记-vectorinsert2.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-vectorinsert2.png)

![《STL源码剖析》的笔记-vectorinsert3.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-vectorinsert3.png)

#### 拓展：copy_backward

std::copy_backward

Copy range of elements backward
Copies the elements in the range [first,last) starting from the end into the range terminating at result.

The function returns an iterator to the first element in the destination range.

The resulting range has the elements in the exact same order as [first,last). To reverse their order, see reverse_copy.

The function begins by copying *(last-1) into *(result-1), and then follows backward by the elements preceding these, until first is reached (and including it).

The ranges shall not overlap in such a way that result (which is the past-the-end element in the destination range) points to an element in the range (first,last]. For such cases, see copy.

The behavior of this function template is equivalent to:

```c++
template<class BidirectionalIterator1, class BidirectionalIterator2>
  BidirectionalIterator2 copy_backward ( BidirectionalIterator1 first,
                                         BidirectionalIterator1 last,
                                         BidirectionalIterator2 result )
{
  while (last!=first) *(--result) = *(--last);
  return result;
}
```

### 4.3 list

#### 4.3.1 list 概述

相较于 vector 的连续线性空间， list 就显得复杂许多，它的好处是每次安插
或删除一个元素，就配置或释放一个元素空间。因此， list 对于空间的运用有绝
对的精准，一点也不浪费。而且，对于任何位置的元素安插或元素移除， list 永
远是常数时间。

#### 4.3.2 list 的节点 （node ）

每一个设计过 list的人都知道，list本身和 list的节点是不同的结构，需要分开
设计。以下是 STL list 的节点（node）结构：

```c++
template <class T>
struct__list_node {
typedef void* void_pointer;
void_pointer prev; //型别为 void*。其实可设为 __list_node<T>*
void_pointer next;
Tdata;
};
```

显然这是一个双向串行

#### 4.3.3 list 的迭代器

list 不再能够像 vector 一样以原生指标做为迭代器，因为其节点不保证在储存空间中连续存在。 list 迭代器必须有能力指向 list 的节点，并有能力做正确的递增、递减、取值、成员存取…等动作。所谓「 list 迭代器正确的递增、递减、取值、成员取用」动作是指，递增时指向下一个节点，递减时指向上一个节点，取值时取的是节点的资料值，成员取用时取用的是节点的成员，如图 4-4。由于STL list 是一个双向串行（double linked-list），迭代器必须具备前移、后移的能力。**所以 list 提供的是Bidirectional Iterators**。list 有一个重要性质：安插动作（insert）和接合动作（splice）都不会造成原有的 list 迭代器失效。这在 vector 是不成立的，因为 vector 的安插动作可能造成记忆体重新配置，导致原有的迭代器全部失效。甚至 list 的元素删除动作（erase），也只有「指向被删除元素」的那个迭代器失效，其它迭代器不受任何影响。

![《STL源码剖析》的笔记-listiterator.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-listiterator.png)

以下是 list 迭代器的设计：

```c++
template<class T, class Ref, class Ptr>
struct__list_iterator {
    typedef __list_iterator<T, T&, T*> iterator;
    typedef __list_iterator<T, Ref, Ptr> self;
    typedef bidirectional_iterator_tag iterator_category ;
    typedef T value_type;
    typedef Ptr pointer;
    typedef Ref reference;
    typedef __list_node<T>* link_type;
    typedef size_t size_type;
    typedef ptrdiff_tdifference_type ;
    link_type node; //迭代器内部当然要有一个原生指标，指向 list 的节点
    // constructor
    __list_iterator(link_type x) : node(x) {}
    __list_iterator() {}
    __list_iterator(const iterator& x) : node(x.node) {}
    bool operator== (const self& x) const { return node == x.node; }
    bool operator!= (const self& x) const { return node != x.node; }
    // 以下对迭代器取值（ dereference ），取的是节点的资料值。
    reference operator* () const { return (*node). data ; }
    // 以下是迭代器的成员存取（ member access ）运算子的标准作法。
    pointer operator-> () const { return &(operator*()); }
    // 对迭代器累加 1，就是前进一个节点
    self& operator++()
        node = (link_type)((*node). next);
        return *this;
    }
    self operator++(int)
        self tmp = *this;
        ++*this;
        return tmp;
    }
    // 对迭代器递减 1，就是后退一个节点
    self& operator--()
        node = (link_type)((*node). prev);
        return *this;
    }
    self operator--(int)
        self tmp = *this;
        --*this;
        return tmp;
    }
};
```

#### 4.3.4 list 的数据结构

SGI list 不仅是一个双向串行，而且还是一个环状双向串行。所以它只需要一个指标，便可以完整表现整个串行：

```c++
template <class T, class Alloc = alloc>// 预设使用 alloc 为配置器
class list {
protected:
    typedef __list_node <T>list_node;
public:
    typedef list_node* link_type;
protected:
    link_type node;// 只要一个指标，便可表示整个环状双向串行
    ...
};
```

如果让指标 node 指向刻意置于尾端的一个空白节点， node 便能符合 STL对于
「前闭后开」区间的要求，成为 last 迭代器，如图 4-5。这么一来，以下几个函
式便都可以轻易完成：

```c++
iterator begin () { return (link_type)((*node).next); }
iterator end () { return node; }
bool empty () const { return node->next == node; }
size_type size () const {
    size_type result = 0;
    distance (begin(), end(), result); // 全域函式，第 3 章。
    return result;
}
// 取头节点的内容（元素值）。
reference front () { return *begin(); }
// 取尾节点的内容（元素值）。
reference back () { return *(--end()); }
```

![《STL源码剖析》的笔记-listcircle.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-listcircle.png)

#### 4.3.5 list 的建构与内存管理 ：constructor, push_back, insert

下面是一个测试程序，我的观察重点在建构的方式以及大小的变化：

```c++
// filename : 4list-test.cpp
#include <list>
#include <iostream>
#include <algorithm>
using namespace std;
int main()
{
    int i;
    list<int> ilist;
    cout << "size=" << ilist. size () << endl; // size=0
    ilist. push_back(0);
    ilist.push_back(1);
    ilist.push_back(2);
    ilist.push_back(3);
    ilist.push_back(4);
    cout << "size=" << ilist.size() << endl; // size=5
    list<int>::iterator ite;
    for(ite = ilist. begin (); ite != ilist. end(); ++ite)
        cout << * ite << ' '; // 0 1 2 3 4
    cout << endl;
    ite = find (ilist.begin(), ilist.end(), 3);
    if (ite!=0)
        ilist. insert(ite, 99);
    cout << "size=" << ilist.size() << endl; // size=6
    cout << *ite << endl; // 3  insert之后，ite仍指向3，这里没用到insert的返回值（指向9的迭代器）
    for(ite = ilist.begin(); ite != ilist.end(); ++ite)
        cout << *ite << ' '; // 0 1 2 99 3 4
    cout << endl;
    ite = find (ilist.begin(), ilist.end(), 1);
    if (ite!=0)
        cout << *(ilist. erase (ite)) << endl; // 2 这里用到了erase的返回值，返回待删除节点后面那个节点
    for(ite = ilist.begin(); ite != ilist.end(); ++ite)
        cout << *ite << ' '; // 0 2 99 3 4
    cout << endl;
}
```

list 预设 使用 alloc （2.2.4节 ）做为 空间 配置 器 ， 并据此 另外 定义 了一 个list_node_allocator ，为的是更方便地以节点大小为配置单位：

```c++
template <class T, class Alloc = alloc>// 预设使用 alloc 为配置器
class list {
protected:
    typedef __list_node<T>list_node;
    // 专属之空间配置器，每次配置一个节点大小：
    typedef simple_alloc<list_node, Alloc>list_node_allocator;
    ...
};
```

于是， list_node_allocator(n) 表示配置n个节点空间。以下㆕个函式，分别
用来配置、释放、建构、摧毁一个节点：

```c++
protected:
// 配置一个节点并传回
link_type get_node () { return list_node_allocator::allocate(); }
// 释放一个节点
void put_node (link_type p) { list_node_allocator::deallocate(p); }
// 产生（配置并建构）一个节点，带有元素值
link_type create_node (const T& x) {
    link_type p = get_node();
    construct (&p->data, x); //全域函式，建构/解构基本工具。
    return p;
}
// 摧毁（解构并释放）一个节点
void destroy_node (link_type p) {
}
destroy(&p->data); //全域函式，建构/解构基本工具。
put_node(p);
}
```

list 提供有许多constructors，其中一个是default constructor，允许我们不指定任何参数做出一个空的 list 出来：

```c++
public:
list() {empty_initialize(); } //产生一个空串行。
protected:
void empty_initialize()
    node =get_node(); //配置一个节点空间，令 node 指向它。
    node ->next = node; //令 node头尾都指向自己，不设元素值。
    node ->prev = node;
}
```

![《STL源码剖析》的笔记-emptylist.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-emptylist.png)

当我们以 push_back() 将新元素安插于 list 尾端，此函式内部呼叫 insert() ：

```c++
void push_back (const T& x) { insert (end(), x); }
```

insert() 是一个多载化函式，有多种型式，其中最简单的一种如下，符合以上所需。首先配置并建构一个节点，然后在尾端做适当的指标动作，将新节点安插进去，**注意返回值指向新插入的节点**：

```c++
// 函式目的：在迭代器 position 所指位置安插一个节点，内容为 x。
iterator insert (iterator position, const T& x) {
    link_type tmp = create_node(x); //产生一个节点（设妥内容为 x）
    // 调整双向指标，使 tmp安插进去。
    tmp->next = position.node;
    tmp->prev = position.node->prev;
    (link_type(position.node->prev))->next = tmp;
    position.node->prev = tmp;
    return tmp;
}
```

于是，先前测试程序连续安插了五个节点（其值为 0 1 2 3 4）之后， list 的状态如图 4-5。如果我们希望在 list 内的某处安插新节点，首先必须确定安插位置，例如我希望在资料值为 3的节点处安插一个数据值为 99 的节点，可以这么做：

```c++
ilite = find (il.begin(), il.end(), 3);
if (ilite!=0)
il. insert(ilite, 99);
```

find() 动作稍后再做说明。安插之后的 list 状态如图 4-6。注意，安插完成后，新节点将位于标兵迭代器（标示出安插点）所指之节点的前方—这是STL对于「安插动作」的标准规范。由于 list 不像 vector 那样有可能在空间不足时做重新配置、数据搬移的动作，所以安插前的所有迭代器在安插动作之后都仍然有效。

![《STL源码剖析》的笔记-listinsert.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-listinsert.png)

#### 4.3.6 list 的元素操作 ：push_front, push_back, erase, pop_front, pop_back, clear, remove, unique, splice, merge, reverse, sort

list 所提供的元素操作动作很多，无法在有限的篇幅中一一讲解—其实也没有这种必要。

```c++
// 安插一个节点，做为头节点
void push_front (const T& x) { insert (begin(), x); }
// 安插一个节点，做为尾节点（上一小节才介绍过）
void push_back (const T& x) { insert (end(), x); }
// 移除迭代器 position 所指节点
iterator erase (iterator position) {
    link_type next_node = link_type(position.node->next);
    link_type prev_node = link_type(position.node->prev);
    prev_node->next = next_node;
    next_node->prev = prev_node;
    destroy_node (position.node);
    return iterator(next_node);
}
// 移除头节点
void pop_front () { erase (begin()); }
// 移除尾节点
void pop_back()
    iterator tmp = end();
    erase(--tmp);
}

//清除所有节点（整个串行）
template <class T, class Alloc>
void list<T, Alloc>:: clear()
{
    link_type cur = (link_type) node->next; // begin()
    while (cur != node ) { //巡访每一个节点
        link_type tmp = cur;
        cur = (link_type) cur->next;
        destroy_node(tmp); //摧毁（解构并释放）一个节点
    }
    // 恢复 node 原始状态
    node->next = node;
    node->prev = node;
}

//将数值为 value之所有元素移除
template <class T, class Alloc>
void list<T, Alloc>:: remove (const T& value) {
    iterator first = begin();
    iterator last = end();
    while (first != last) { //巡访每一个节点
        iterator next = first;
        ++next;
        if (*first == value) erase(first); //找到就移除
        first = next;
    }
}
//移除数值相同的连续元素。注意，只有「连续而相同的元素」，才会被移除剩一个。
template <class T, class Alloc>
void list<T, Alloc>:: unique () {
    iterator first = begin();
    iterator last = end();
    if (first == last) return; //空串行，什么都不必做。
    iterator next = first;
    while (++next != last) { //巡访每一个节点
        if (*first == *next) //如果在此区段中有相同的元素
            erase(next);        //移除之
        else
            first = next; // 调整指针
        next = first; // 修正区段范围
    }
}
```

由于 list 是一个双向环状串行，只要我们把边际条件处理好，那么，在头部或尾部安插元素（ push_front 和 push_back ），动作几乎是一样的，在头部或尾部移除元素（ pop_front 和 pop_back ），动作也几乎是一样的。移除（ erase ）某个迭代器所指元素，只是做一些指标搬移动作而已，并不复杂。如果图4-6再经以下搜寻并移除的动作，状况将如图 4-7。

```c++
ite = find(ilist.begin(), ilist.end(), 1);
if (ite!=0)
    cout << *(ilist.erase(ite)) << endl;
```

![《STL源码剖析》的笔记-listerase.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-listerase.png)

list 内部提供一个所谓的迁移动作（ transfer ）：将某连续范围的元素迁移到某个特定位置之前（好像C++标准中没有这个函数）。技术上很简单，节点间的指标移动而已。这个动作为其它的复杂动作如 splice, sort, merge 等奠定良好的基础。下面是 transfer 的源码：

```c++
protected:
// 将 [first,last) 内的所有元素搬移到 position 之前。
void transfer (iterator position, iterator first, iterator last) {
    if (position != last) {
        (*(link_type((*last.node).prev))).next = position.node;    // (1)
        (*(link_type((*first.node).prev))).next = last.node;    // (2)
        (*(link_type((*position.node).prev))).next = first.node; // (3)
        link_type tmp = link_type((*position.node).prev);       // (4)
        (*position.node).prev = (*last.node).prev;              // (5)
        (*last.node).prev = (*first.node).prev;                 // (6)
        (*first.node).prev = tmp;                               // (7)
    }
}
```

以上七个动作，一步一步地显示于图 4-8a。

![《STL源码剖析》的笔记-listtransfer.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-listtransfer.png)

transfer 所接受的 [first,last) 区间，是否可以在同一个 list 之中呢？答案是可以。你只要想象图4-8a所画的两条 lists其实是同一个 list 的两个区段，就不难得到答案了。

上述的 transfer 并非公开界面。 list 公开提供的是所谓的接合动作（ splice ）：将某连续范围的元素从一个 list 搬移到另一个（或同一个） list 的某个定点。如果接续先前 4list-test.cpp 程序的最后执行点，继续执行以下 splice 动作：

```c++
int iv[5] = { 5,6,7,8,9 };
list<int> ilist2(iv, iv+5);
// 目前，ilist的内容为 0 2 99 3 4
ite = find(ilist.begin(), ilist.end(), 99);
ilist. splice (ite,ilist2); // 0 2 5 6 7 8 9 99 3 4
ilist. reverse();           // 4 3 99 9 8 7 6 5 2 0
ilist. sort();              // 0 2 3 4 5 6 7 8 9 99
```

很容易便可看出效果。图4-8b显示接合动作。技术上很简单，只是节点间的指标移动而已，这些动作已完全由 transfer() 做掉了。

![《STL源码剖析》的笔记-listsplice.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-listsplice.png)

为了提供各种接口弹性， `list<T>::splice` 有许多版本：

```c++
public:
// 将 x接合于 position 所指位置之前。x必须不同于 *this。
void splice (iterator position, list& x) {
    if (!x.empty())
        transfer (position, x.begin(), x.end());
}
// 将 i 所指元素接合于 position 所指位置之前。position 和 i 可指向同一个 list。
void splice (iterator position, list&, iterator i) {
    iterator j = i;
    ++j;
    if (position == i || position == j) return;
    transfer (position, i, j);
}
// 将 [first,last) 内的所有元素接合于 position 所指位置之前。
// position 和[first,last)可指向同一个 list，
// 但 position 不能位于[first,last)之内。
void splice (iterator position, list&, iterator first, iterator last) {
    if (first != last)
        transfer (position, first, last);
}
```

以下是 merge(), reverse(), sort() 的源码。有了 transfer() 在手，这些
动作都不难完成。

```c++
// merge()将 x合并到 *this身上。两个 lists 的内容都必须先经过递增排序。
template <class T, class Alloc>
void list<T, Alloc>:: merge (list<T, Alloc>& x) {
    iterator first1 = begin();
    iterator last1 = end();
    iterator first2 = x.begin();
    iterator last2 = x.end();
    // 注意：前提是，两个 lists都已经过递增排序，
    while (first1 != last1 && first2 != last2)
        if (*first2 < *first1) {
            iterator next = first2;
            transfer (first1, first2, ++next);
            first2 = next;
        }
        else
            ++first1;
    if (first2 != last2) transfer (last1, first2, last2);
}

// reverse()将 *this的内容逆向重置
template <class T, class Alloc>
void list<T, Alloc>:: reverse () {
    // 以下判断，如果是空白串行，或仅有一个元素，就不做任何动作。
    // 使用 size() == 0 || size() == 1 来判断，虽然也可以，但是比较慢。
    if (node->next == node || link_type(node->next)->next == node) return;
    iterator first = begin();
    ++first;
    while (first != end()) {
        iterator old = first;
        ++first;
        transfer (begin(), old, first);
    }
}

// list 不能使用 STL 算法 sort()，必须使用自己的 sort() member function ，
//因为 STL 算法 sort() 只接受 RamdonAccessIterator.
//本函式采用 quick sort.
template <class T, class Alloc>
void list<T, Alloc>:: sort () {
    // 以下判断，如果是空白串行，或仅有一个元素，就不做任何动作。
    // 使用 size() == 0 || size() == 1 来判断，虽然也可以，但是比较慢。
    if (node->next == node || link_type(node->next)->next == node) return;
    // 一些新的 lists，做为中介数据存放区
    list<T, Alloc> carry;
    list<T, Alloc> counter[64];
    int fill = 0;
    while (!empty()) {
        carry. splice (carry.begin(), *this, begin());
        int i = 0;
        while(i < fill && !counter[i].empty()) {
            counter[i]. merge(carry);
            carry. swap (counter[i++]);
        }
        carry. swap (counter[i]);
        if (i == fill) ++fill;
    }
    for (int i = 1; i < fill; ++i)
        counter[i]. merge (counter[i-1]);
    swap (counter[fill-1]);
}
```

### 4.4 deque

vector 是单向开口的连续线性空间， deque 则是一种双向开口的连续线性空间。所谓双向开口，意思是可以在头尾两端分别做元素的安插和删除动作，如图4-9。vector 当然也可以在头尾两端做动作（从技术观点），但是其头部动作效率奇差，无法被接受。

![《STL源码剖析》的笔记-deque.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-deque.png)

deque 和 vector 的最大差异，一在于 deque 允许于常数时间内对起头端进行元素的安插或移除动作，二在于 deque 没有所谓容量（ capacity ）观念，因为它是动态地以分段连续空间组合而成，随时可以增加一段新的空间并链接起来。换句话说，像 vector 那样「因旧空间不足而重新配置一块更大空间，然后复制元素，再释放旧空间」这样的事情在 deque 是不会发生的。也因此， deque 没有必要提供所谓的空间保留（ reserve ）功能。虽然 deque 也提供Ramdon Access Iterator，但它的迭代器并不是原生指标，其复杂度和 vector 不可以道里计（稍后看到源码，你便知道），这当然在在影响了各个运算层面。因此，除非必要，我们应尽可能选择使用 vector 而非 deque 。对 deque 进行的排序动作，为了最高效率，可将 deque 先完整复制到一个 vector身上，将 vector 排序后（利用 STL sort 算法），再复制回 deque 。

#### deque的中控器

deque 是连续空间（至少逻辑看来如此），连续线性空间总令我们联想到 array或 vector 。 array 无法成长， vector 虽可成长，却只能向尾端成长，而且其所谓成长原是个假象，事实上是 (1)另觅更大空间、(2)将原数据复制过去、(3)释放原空间三部曲。如果不是 vector 每次配置新空间时都有留下一些余裕，其「成长」假象所带来的代价将是相当高昂。

deque 系由一段一段的定量连续空间构成。一旦有必要在 deque 的前端或尾端增加新空间，便配置一段定量连续空间，串接在整个 deque 的头端或尾端。 deque 的最大任务，便是在这些分段的定量连续空间上，维护其整体连续的假象，并提供随机存取的界面。避开了「重新配置、复制、释放」的轮回，代价则是复杂的迭代器架构。

受 到 分 段 连 续 线 性 空 间 的 字 面 影 响 ， 我 们 可 能 以 为 deque 的 实 作 复 杂 度 和vector 相比虽不中亦不远矣，其实不然。主要因为，既曰分段连续线性空间，就必须有中央控制，而为了维护整体连续的假象，数据结构的设计及迭代器前进后退等动作都颇为繁琐。 deque 的实作码份量远比 vector 或 list 都多得多。deque 采用一块所谓的map（注意，不是 STL 的 map 容器）做为主控。这里所谓map是一小块连续空间，其中每个元素（此处称为一个节点，node）都是指标，指向另一段（较大的）连续线性空间，称为缓冲区。缓冲区才是 deque 的储存空间主体。SGI STL允许我们指定缓冲区大小，默认值 0表示将使用 512 bytes缓冲区。

```c++
template <class T, class Alloc = alloc, size_t BufSiz = 0>
class deque {
public: // Basic types
    typedef T value_type;
    typedef value_type* pointer;
    ...
protected: // Internal typedefs
    // 元素的指针的指针（ pointer of pointer of T ）
    typedef pointer* map_pointer;
protected: // Data members
    map_pointer map; //指向 map，map 是块连续空间，其内的每个元素
            // 都是一个指标（称为节点），指向一块缓冲区。
    size_type map_size;// map 内可容纳多少指标。
    ...
};
```

map 其实是一个 T** ，也就是说它是一个指标，所指之物又是一个指标，指向型别为 T 的一块空间，如图 4-10。

![《STL源码剖析》的笔记-dequemap.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-dequemap.png)

#### 4.4.3 deque 的迭代器

deque 是 分段连续空间 。维 护其「整体连续」假象 的任务 ，着落在迭代器的operator++ 和 operator-- 两个运算子身上。

让我们思考一下， deque 迭代器应该具备什么结构。首先，它必须能够指出分段连续空间（亦即缓冲区）在哪里，其次它必须能够判断自己是否已经处于其所在缓冲区的边缘，如果是，一旦前进或后退时就必须跳跃至下一个或上一个缓冲区。为了能够正确跳跃， deque 必须随时掌握管控中心（map）。下面这种实作方式符合需求：

```c++
template <class T, class Ref, class Ptr, size_t BufSiz >
struct__deque_iterator { //未继承 std::iterator
    typedef __deque_iterator<T, T&, T*, BufSiz> iterator;
    typedef __deque_iterator<T, const T&, const T*, BufSiz> const_iterator;
    static size_t buffer_size () {return__deque_buf_size (BufSiz , sizeof(T)); }
    // 未继承 std::iterator ，所以必须自行撰写五个必要的迭代器相应型别（第 3 章）
    typedef random_access _iterator_tagiterator_category ; // (1)
    typedef T value_type;    // (2)
    typedef Ptr pointer;    // (3)
    typedef Ref reference;    // (4)
    typedef size_t size_type;
    typedef ptrdiff_tdifference_type ;// (5)
    typedef T** map_pointer;
    typedef __deque_iterator self;
    // 保持与容器的联结
    T* cur;//此迭代器所指之缓冲区中的现行（current）元素
    T* first;//此迭代器所指之缓冲区的头
    T* last;//此迭代器所指之缓冲区的尾（含备用空间）
    map_pointernode;//指向管控中心
    ...
};
```

其中用来决定缓冲区大小的函式 buffer_size() ，呼叫 __deque_buf_size() ，后者是个全域函式，定义如下：

```c++
//如果 n不为 0，传回 n，表示 buffer size 由使用者自定。
//如果 n为 0，表示 buffer size使用默认值，那么
// 如果 sz（元素大小， sizeof(value_type) ）小于 512，传回 512/sz，
// 如果 sz 不小于 512，传回 1。
inlinesize_t __deque_buf_size (size_t n, size_t sz)
{
    return n != 0 ? n : (sz < 512 ? size_t(512 / sz) : size_t(1));
}
```

![《STL源码剖析》的笔记-dequemapbufferiterator.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-dequemapbufferiterator.png)

假设现在我们产生一个 `deque<int>` ，并令其缓冲区大小为32，于是每个缓冲区可容纳 32/sizeof(int)=4 个元素。经过某些操作之后， deque 拥有 20 个元素，那么其 begin() 和 end() 所传回的两个迭代器应该如图4-12。这两个迭代器事实上一直保持在 deque 内，名为 start 和 finish ，稍后在 deque 数据结构中便可看到）。

20个元素需要 20/8 = 3 个缓冲区，所以map之内运用了三个节点。迭代器 start内的 cur 指标当然指向缓冲区的第一个元素，迭代器 finish 内的 cur 指标当然指向缓冲区的最后元素（的下一位置）。注意，最后一个缓冲区尚有备用空间。稍后如果有新元素要安插于尾端，可直接拿此备用空间来使用。

![《STL源码剖析》的笔记-dequebeginend.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-dequebeginend.png)

下面是 deque 迭代器的几个关键行为。由于迭代器内对各种指标运算都做了多载化动作，所以各种指标运算如加、减、前进、后退…都不能直观视之。其中最重点的关键就是：一旦行进时遇到缓冲区边缘，要特别当心，视前进或后退而定，可能需要呼叫 set_node() 跳一个缓冲区：

```c++
void set_node (map_pointer new_node) {
    node = new_node;
    first = *new_node;
    last = first + difference_type(buffer_size());
}
//以下各个多载化运算子是 __deque_iterator<> 成功运作的关键。
reference operator* () const { return *cur ; }
pointer operator-> () const { return &( operator* ()); }
difference_type operator- (const self& x) const {
    return difference_type(buffer_size()) * (node - x.node - 1) +
        (cur - first) + (x.last - x.cur);
}
// 参考 More Effective C++ , item6: Distinguish between prefix and
// postfix forms of increment and decrement operators.
self& operator++ () {
    ++cur;
    if (cur == last) {
        //切换至下一个元素。
        //如果已达所在缓冲区的尾端，
        set_node(node + 1);//就切换至下一节点（亦即缓冲区）
        cur = first;        // 的第一个元素。
    }
    return *this;
}
self operator++ (int) { //后置式，标准写法
    self tmp = *this;
    ++*this;
    return tmp;
}
self& operator-- () {
    if (cur == first) { //如果已达所在缓冲区的头端，
        set_node(node - 1);//就切换至前一节点（亦即缓冲区）
        cur = last;
    }
    --cur;
    return *this;
}
// 的最后一个元素。
//切换至前一个元素。
self operator-- (int) { //后置式，标准写法
    self tmp = *this;
    --*this;
    return tmp;
}
// 以下实现随机存取。迭代器可以直接跳跃 n个距离。
self& operator+= (difference_type n) {
    difference_type offset = n + (cur - first);
    if (offset >= 0 && offset < difference_type(buffer_size()))
        // 标的位置在同一缓冲区内
        cur += n;
    else {
        // 标的位置不在同一缓冲区内
        difference_type node_offset =
            offset > 0 ? offset / difference_type(buffer_size()) : -difference_type((-offset - 1) / buffer_size()) - 1;
        // 切换至正确的节点（亦即缓冲区）
        set_node (node + node_offset);
        // 切换至正确的元素
        cur = first + (offset - node_offset * difference_type( buffer_size()));
    }
    return *this;
}
// 参考 More Effective C++ , item22: Consider using op= instead of
// stand-alone op.
self operator+ (difference_type n) const {
    self tmp = *this;
    return tmp += n; // 唤起 operator+=
}
self& operator-= (difference_type n) { return *this += -n; }
// 以上利用 operator+= 来完成 operator-=

// 参考 More Effective C++ , item22: Consider using op= instead of
// stand-alone op.
self operator- (difference_type n) const {
    self tmp = *this;
    return tmp -= n; // 唤起 operator-=
}
// 以下实现随机存取。迭代器可以直接跳跃 n个距离。
reference operator[] (difference_type n) const { return *( *this + n); }
// 以上唤起 operator*, operator+
bool operator== (const self& x) const { return cur == x.cur; }
bool operator!= (const self& x) const { return !(*this == x); }
bool operator< (const self& x) const {
    return (node == x.node) ? (cur < x.cur) : (node < x.node);
}
```

#### 4.4.4 deque 的数据结构

deque 除了维护一个先前说过的指向map的指标外，也维护 start, finish 两个迭代器，分别指向第一缓冲区的第一个元素和最后缓冲区的最后一个元素（的下一位置）。此外它当然也必须记住目前的map大小。因为一旦map所提供的节点不足，就必须重新配置更大的一块map。

deque 除了维护一个先前说过的指向map的指标外，也维护 start, finish 两个迭代器，分别指向第一缓冲区的第一个元素和最后缓冲区的最后一个元素（的下一位置）。此外它当然也必须记住目前的map大小。因为一旦map所提供的节点不足，就必须重新配置更大的一块map。

```c++
//见 __deque_buf_size() 。BufSize 默认值为 0的唯一理由是为了闪避某些
//编译器在处理常数算式（ constant expressions ）时的臭虫。
//预设使用 alloc为配置器。
template <class T, class Alloc = alloc, size_t BufSiz = 0>
class deque {
public: // Basic types
    typedef T value_type;
    typedef value_type* pointer;
    typedef size_t size_type;
public: // Iterators
    typedef__deque_iterator <T, T&, T*,BufSiz> iterator;
protected: // Internal typedefs
    // 元素的指针的指针（ pointer of pointer of T ）
    typedef pointer* map_pointer;
protected: // Data members
    iterator start;
    iterator finish;
    map_pointer map;
    //表现第一个节点。
    //表现最后一个节点。
    //指向 map，map 是块连续空间，
    // 其每个元素都是个指针，指向一个节点（缓冲区）。
    size_type map_size;// map 内有多少指标。
    ...
};
```

有了上述结构，以下数个机能便可轻易完成：

```c++
public: // Basic accessors
iterator begin () { return start ; }
iterator end () { return finish ; }
reference operator[] (size_type n) {
    return start [difference_type(n)]; //唤起 __deque_iterator<>::operator[]
}
reference front () { return *start ; } // 唤起 __deque_iterator<>::operator*
reference back () {
    iterator tmp = finish;
    --tmp; //唤起 __deque_iterator<>::operator--
    return *tmp; //唤起 __deque_iterator<>::operator*
    // 以上三行何不改为： return *(finish-1);
    // 因为 __deque_iterator<> 没有为 (finish-1) 定义运算子?!（个人疑惑：不是定义了op-吗？？？？）
}
// 下行最后有两个 ‘;’，虽奇怪但合乎语法。
size_type size () const { return finish - start;; }
// 以上唤起 iterator::operator-
size_type max_size () const { return size_type(-1); }
bool empty () const { return finish == start; }
```

#### 4.4.5 deque 的建构与内存管理 ctor, push_back, push_front

deque 的缓冲区扩充动作相当琐碎繁杂，以下将以分解动作的方式一步一步图解说明。程序一开始宣告了一个 deque ：

```c++
deque<int,alloc,32> ideq(20,9);
```

其缓冲区大小为 32 bytes，并令其保留 20 个元素空间，每个元素初值为 9。为了指定 deque 的第三个 template参数（缓冲区大小），我们必须将前两个参数都指明出来（这是 C++语法规则），因此必须明确指定 alloc （第二章）为空间配置器。

在尾部新增元素的deque示意图如下

![《STL源码剖析》的笔记-dequepushback.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-dequepushback.png)

再加一个元素，则要引发新缓冲区的配置

![《STL源码剖析》的笔记-dequepushbacknewbuffer.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-dequepushbacknewbuffer.png)

在头部插入也是类似的

![《STL源码剖析》的笔记-dequepushfrontnewbuffer.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-dequepushfrontnewbuffer.png)

如果头部缓冲区仍有备用空间，则直接往前加即可

![《STL源码剖析》的笔记-20191229194315.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-20191229194315.png)

#### 4.4.6 deque 的元素操作：pop_back, pop_front, clear, erase, insert

pop操作正好是push操作的相反，不赘述，对deque进行find，返回的是deque的迭代器（类），该迭代器（类）主要有四个迭代器（指针）构成

![《STL源码剖析》的笔记-dequefind.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-dequefind.png)

对于erase和insert操作，涉及到元素的移动，不同于vector只对position后面的元素移动，deque的移动可以双向的，如果position离start更近，则移动pisition之前的元素，如果position离finish更近，则移动position之后的元素

### 4.5 stack

#### 4.5.1 stack 概述

stack 是一种先进后出（First In Last Out，FILO）的数据结构。它只有一个出口，型式如图4-18。 stack 允许新增元素、移除元素、取得最顶端元素。但除了最顶端外，没有任何其它方法可以存取 stack 的其它元素。换言之 stack 不允许有走访行为。将元素推入 stack 的动作称为 push，将元素推出 stack 的动作称为pop。

![《STL源码剖析》的笔记-stack.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-stack.png)

#### 4.5.2 stack 定义式完整列表

以某种既有容器做为底部结构，将其接口改变，使符合「先进后出」的特性，形成一个 stack ，是很容易做到的。 deque 是双向开口的数据结构，若以 deque 为底部结构并封闭其头端开口，便轻而易举地形成了一个 stack 。因此，SGI STL 便以 deque 做为预设情况下的 stack 底部结构， stack 的实作因而非常简单，源码十分简短，本处完整列出。

由于 stack 系以底部容器完成其所有工作，而具有这种「修改某物接口，形成另一种风貌」之性质者，称为adapter（配接器），因此 STL stack 往往不被归类为 container（容器），而被归类为container adapter。

#### 4.5.3 stack 没有迭代器

stack 所有元素的进出都必须符合「先进后出」的条件，只有 stack 顶端的元素，才有机会被外界取用。 stack 不提供走访功能，也不提供迭代器。

#### 4.5.4 以 list 做为 stack 的底层容器

除了 deque 之外， list 也是双向开口的数据结构。上述 stack 源码中使用的底层容器的函式有 empty, size, back, push_back, pop_back ，凡此种种 list都具备。因此若以 list 为底部结构并封闭其头端开口，一样能够轻易形成一个stack 。

```c++
// file : 4stack-test.cpp
#include <stack>
#include <list>
#include <iostream>
#include <algorithm>
using namespace std;
int main()
{
    stack<int, list<int> > istack;
    istack.push(1);
    istack.push(3);
    istack.push(5);
    istack.push(7);
    cout << istack.size() << endl; // 4
    cout << istack.top() << endl; // 7
    istack.pop(); cout << istack.top() << endl; // 5
    istack.pop(); cout << istack.top() << endl; // 3
    istack.pop(); cout << istack.top() << endl; // 1
    cout << istack.size() << endl; // 1
}
```

### 4.6 queue

#### 4.6.1 queue 概述

queue 是一种先进先出（First In First Out，FIFO）的数据结构。它有两个出口，
型式如图4-19。 queue 允许新增元素、移除元素、从最底端加入元素、取得最顶
端元素。但除了最底端可以加入、最顶端可以取出，没有任何其它方法可以存取
queue 的其它元素。换言之 queue 不允许有走访行为。

将元素推入 queue 的动作称为 push，将元素推出 queue 的动作称为pop。

![《STL源码剖析》的笔记-queue.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-queue.png)

#### 4.6.2 queue 定义式完整列表

以某种既有容器为底部结构，将其接口改变，使符合「先进先出」的特性，形成一个 queue ，是很容易做到的。 deque 是双向开口的数据结构，若以 deque 为底部结构并封闭其底端的出口和前端的入口，便轻而易举地形成了一个 queue 。因此，SGI STL 便以 deque 做为预设情况下的 queue 底部结构， queue 的实作因而非常简单，源码十分简短，本处完整列出。

由于 queue 系以底部容器完成其所有工作，而具有这种「修改某物接口，形成另一种风貌」之性质者，称为adapter（配接器），因此 STL queue 往往不被归类为 container（容器），而被归类为container adapter。

#### 4.6.3 queue 没有迭代器

queue 所有元素的进出都必须符合「先进先出」的条件，只有 queue 顶端的元素，才有机会被外界取用。 queue 不提供走访功能，也不提供迭代器。

#### 4.6.4 以 list 做为 queue 的底层容器

除了 deque 之外， list 也是双向开口的数据结构。上述 queue 源码中使用的底层容器的函式有 empty, size, back, push_back, pop_back ，凡此种种 list都具备。因此若以 list 为底部结构并封闭其头端开口，一样能够轻易形成一个queue 。下面是作法示范。

```c++
// file : 4queue-test.cpp
#include <queue>
#include <list>
#include <iostream>
#include <algorithm>
using namespace std;
int main()
{
    queue<int, list<int> > iqueue;
    iqueue.push(1);
    iqueue.push(3);
    iqueue.push(5);
    iqueue.push(7);
    cout << iqueue.size() << endl; // 4
    cout << iqueue.front() << endl; // 1
    iqueue.pop(); cout << iqueue.front() << endl; // 3
    iqueue.pop(); cout << iqueue.front() << endl; // 5
    iqueue.pop(); cout << iqueue.front() << endl; // 7
    cout << iqueue.size() << endl; // 1
}
```

### 4.7 heap （ 隐性表述 ，implicit representation ）

#### 4.7.1 heap 概述

heap 并不归属于STL 容器组件，它是个幕后英雄，扮演 priority queue （4.8节）的推手。顾名思义， priority queue 允许使用者以任何次序将任何元素推入容器内，但取出时一定是从优先权最高（也就是数值最高）之元素开始取。 binary max heap 正是具有这样的特性，适合做为 priority queue 的底层机制。让我们做点分析。如果使用 4.3 节的 list 做为 priorityqueue 的底层机制，元素安插动作可享常数时间。但是要找到 list 中的极值，却需要对整个 list 进行线性扫描。我们也可以改个作法，让元素安插前先经过排序这一关，使得 list 的元素值总是由小到大（或由大到小），但这么一来，收之东隅却失之桑榆：虽然取得极值以及元素删除动作达到最高效率，元素的安插却只有线性表现。

比较麻辣的作法是以binary search tree（如5.1节的 RB-tree ）做为 priority queue 的底层机制。这么一来元素的安插和极值的取得就有O(logN)的表现。但杀鸡用牛刀，未免小题大作，一来binary search tree的输入需要足够的随机性，二来binary search tree并不容易实作。 priorityqueue 的复杂度，最好介于queue 和binary search tree之间，才算适得其所。binary heap便是这种条件下的适当候选人。

所谓binary heap就是一种complete binary tree（完全二元树） ，也就是说，整棵binary tree除了最底层的叶节点 (s) 之外，是填满的，而最底层的叶节点 (s) 由左至右又不得有空隙。图 4-20 是一个 complete binary tree。

![《STL源码剖析》的笔记-completebinarytreeandarray.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-completebinarytreeandarray.png)

complete binary tree整棵树内没有任何节点漏洞，这带来一个极大好处：我们可以利用 array 来储存所有节点。假设动用一个小技巧（SGI STL 提供的 heap 并未使用此一小技巧） ，将 array 的 #0元素保留（或设为无限大值或无限小值），那么当complete binary tree中的某个节点位于 array 的 i 处，其左子节点必位于 array 的 2i 处，其右子节点必位于 array 的2i+1 处，其父节点必位于「 i/2 」处（此处的「」权且代表高斯符号，取其整数）。通过这么简单的位置规则， array 可以轻易实作出complete binary tree。这种以array 表述 tree 的方式，我们称为**隐式表述法（implicit representation）**。

这么一来，我们需要的工具就很简单了：一个 array 和一组 heap 算法（用来安插元素、删除元素、取极值、将某一整组数据排列成一个 heap ）。 array 的缺点是无法动态改变大小，而 heap 却需要这项功能，因此以 vector （4.2节）代替 array 是更好的选择。

根据元素排列方式， heap 可分为 max-heap 和 min-heap 两种，前者每个节点的键值（key）都大于或等于其子节点键值，后者的每个节点键值（key）都小于或等于其子节点键值。因此， max-heap 的最大值在根节点，并总是位于底层 array 或vector 的起头处； min-heap 的最小值在根节点，亦总是位于底层 array 或vector 的起头处。STL 供应的是 max-heap ，因此以下我说 heap 时，指的是max-heap 。

#### 4.7.2 heap 算法

##### push_heap 算法

为了满足 complete binary tree的条件，新加入的元素一定要放在最下一层做为叶节点，并填补在由左至右的第一个空格，也就是把新元素安插在底层 vector 的 end() 处。

新元素是否适合于其现有位置呢？为满足 max-heap 的条件（每个节点的键值都大于或等于其子节点键值），我们执行一个所谓的percolate up（上溯）程序：将新节点拿来与其父节点比较，如果其键值（key）比父节点大，就父子对换位置。如此一直上溯，直到不需对换或直到根节点为止。

![《STL源码剖析》的笔记-pushheap.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-pushheap.png)

##### pop_heap 算法

图4-22是 pop_heap 算法的实际操演情况。既然身为 max-heap ，最大值必然在根节点。 pop 动作取走根节点（其实是移至底部容器 vector 的最后一个元素）之后，为了满足 complete binary tree的条件，必须将最下一层最右边的叶节点拿掉，现在我们的任务是为这个被拿掉的节点找一个适当的位置。

为满足 max-heap 的条件（每个节点的键值都大于或等于其子节点键值），我们执行一个所谓的percolate down（下放）程序：将根节点（最大值被取走后，形成一个「洞」）填入上述那个失去生存空间的叶节点值，再将它拿来和其两个子节点比较键值（key），并与较大子节点对调位置。如此一直下放，直到这个「洞」的键值大于左右两个子节点，或直到下放至叶节点为止。

注意， pop_heap 之后，最大元素只是被置放于底部容器的最尾端，尚未被取走。如果要取其值，可使用底部容器（ vector ）所提供的 back() 操作函式。如果要移除它，可使用底部容器（ vector ）所提供的 pop_back() 操作函式。

![《STL源码剖析》的笔记-popheap.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-popheap.png)

##### sort_heap 算法

既然每次 pop_heap 可获得 heap 之中键值最大的元素，如果持续对整个 heap 做pop_heap 动作，每次将操作范围从后向前缩减一个元素（因为 pop_heap 会把键值最大的元素放在底部容器的最尾端），当整个程序执行完毕，我们便有了一个递增序列，显然排序过后原来的heap就不再是合法的heap了。图 4-23 是 sort_heap 的实际操演情况。

![《STL源码剖析》的笔记-sortheap.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-sortheap.png)

#### 4.7.3 heap 没有迭代器

heap 的所有元素都必须遵循特别的（complete binary tree）排列规则，所以 heap 不提供走访功能，也不提供迭代器。

#### 4.7.4 heap 测试实例

```c++
// file: 4heap-test.cpp
#include <vector>
#include <iostream>
#include <algorithm> // heap algorithms
using namespace std;
int main()
{
    {
    // test heap ( 底层以 vector完成)
    int ia[9] = {0,1,2,3,4,8,9,3,5};
    vector<int> ivec(ia, ia+9);
    make_heap (ivec.begin(), ivec.end());
    for(int i=0; i<ivec.size(); ++i)
        cout << ivec[i] << ' '; // 9 5 8 3 4 0 2 3 1
    cout << endl;
    ivec.push_back(7);
    push_heap (ivec.begin(), ivec.end()); // 新元素7已经位于vector尾端
    for(int i=0; i<ivec.size(); ++i)
        cout << ivec[i] << ' '; // 9 7 8 3 5 0 2 3 1 4
    cout << endl;
    pop_heap (ivec.begin(), ivec.end());
    cout << ivec.back() << endl; // 9. return but no remove.仍在vector中，处于尾端
    ivec.pop_back(); // remove last elem and no return
    for(int i=0; i<ivec.size(); ++i)
        cout << ivec[i] << ' '; // 8 7 4 3 5 0 2 3 1
    cout << endl;
    sort_heap (ivec.begin(), ivec.end());
    for(int i=0; i<ivec.size(); ++i)
        cout << ivec[i] << ' '; // 0 1 2 3 3 4 5 7 8
    cout << endl;
    }
    {
    // test heap ( 底层以 array 完成)
    int ia[9] = {0,1,2,3,4,8,9,3,5};
    make_heap(ia, ia+9);
    // array 无法动态改变大小，因此不可以对满载的 array 做 push_heap() 动作。
    // 因为那得先在 array尾端增加一个元素。
    // 如果对于一个满载的array执行push_heap()，该函数会将最后一个元素视为新增元素，
    // 并将其余元素视为一个完整的heap结构（实际上它们的确是），因此执行结果等于原先的heap
    sort_heap(ia, ia+9);
    for(int i=0; i<9; ++i)
    cout << ia[i] << ' '; // 0 1 2 3 3 4 5 8 9
    cout << endl;
    // 经过排序之后的 heap，不再是个合法的 heap
    // 重新再做一个 heap
    make_heap(ia, ia+9);
    pop_heap(ia, ia+9);
    cout << ia[8] << endl; // 9
    }
    {
    // test heap ( 底层以 array 完成)
    int ia[6] = {4,1,7,6,2,5};
    make_heap(ia, ia+6);
    for(int i=0; i<6; ++i)
    cout << ia[i] << ' '; // 7 6 5 1 2 4
    cout << endl;
    }
}
```

### 4.8 priority_queue

#### 4.8.1 priority_queue 概述

顾名思义， priority_queue 是一个拥有权值观念的 queue ，它允许加入新元素、移除旧元素，审视元素值等功能。由于这是一个 queue ，所以只允许在底端加入元素，并从顶端取出元素，除此之外别无其它存取元素的途径。

priority_queue 带有权值观念，其内的元素并非依照被推入的次序排列，而是自动依照元素的权值排列（通常权值以实值表示）。权值最高者，排在最前面。

预设情况下 priority_queue 系利用一个 max-heap 完成，后者是一个以 vector表现的 complete binary tree（4.7 节）。 max-heap 可以满足 priority_queue 所需要的「依权值高低自动递增排序」的特性。

![《STL源码剖析》的笔记-priorityheap.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-priorityheap.png)

###　4.8.2 priority_queue 定义式完整列表

由于 priority_queue 完全以底部容器为根据，再加上 heap 处理规则，所以其
实作非常简单。预设情况下是以 vector 为底部容器。源码很简短，此处完整列
出。

queue 以底部容器完成其所有工作。具有这种「修改某物接口，形成另一种风貌」
之性质者，称为 adapter（配接器），因此 STL priority_queue 往往不被归类为
container（容器），而被归类为container adapter。

```c++
template <class T, class Sequence = vector<T>,
class Compare = less<typename Sequence::value_type> >
class priority_queue {
public:
    typedef typename Sequence::value_type value_type;
    typedef typename Sequence::size_type size_type;
    typedef typename Sequence::reference reference;
    typedef typename Sequence::const_referenceconst_reference ;
protected:
    Sequence c; //底层容器
    Compare comp;//元素大小比较标准
public:
    priority_queue () : c() {}
    explicit priority_queue (const Compare& x) : c(), comp(x) {}

    //以下用到的 make_heap(), push_heap(), pop_heap() 都是泛型算法
    //注意，任一个建构式都立刻于底层容器内产生一个 implicit representation heap 。
    template <class InputIterator>
    priority_queue (InputIterator first, InputIterator last, const Compare& x)
    : c(first, last), comp(x) { make_heap (c.begin(), c.end(), comp); }
    template <class InputIterator>
    priority_queue (InputIterator first, InputIterator last)
    : c(first, last) { make_heap (c.begin(), c.end(), comp); }

    bool empty () const { return c.empty (); }
    size_type size () const { return c.size (); }
    const_reference top () const { return c.front (); }

    void push (const value_type& x) {
        __STL_TRY {
            // push_heap是泛型算法，先利用底层容器的 push_back() 将新元素
            // 推入末端，再重排 heap。见 C++ Primer p.1195 。
            c.push_back(x);
            push_heap (c.begin(), c.end(), comp); // push_heap是泛型算法
        }
        __STL_UNWIND(c.clear());
    }
    void pop () {
        __STL_TRY {
            // pop_heap 是泛型算法，从 heap 内取出一个元素。它并不是真正将元素
            // 弹出，而是重排 heap，然后再以底层容器的 pop_back() 取得被弹出
            // 的元素。见 C++ Primer p.1195 。
            pop_heap (c.begin(), c.end(), comp);
            c.pop_back();
        }
        __STL_UNWIND(c.clear());
    }
};
```

#### 4.8.3 priority_queue 没有迭代器

priority_queue 的所有元素，进出都有一定的规则，只有 queue 顶端的元素（权值最高者），才有机会被外界取用。 priority_queue 不提供走访功能，也不提供迭代器。

#### 4.8.4 priority_queue 测试实例

```c++
// file: 4pqueue-test.cpp
#include <queue>
#include <iostream>
#include <algorithm>
using namespace std;
int main()
{
    // test priority queue...
    int ia[9] = {0,1,2,3,4,8,9,3,5};
    priority_queue <int> ipq(ia, ia+9);
    cout << "size=" << ipq.size() << endl; // size=9
    for(int i=0; i<ipq. size(); ++i)
        cout << ipq. top () << ' '; // 9 9 9 9 9 9 9 9 9
    cout << endl;
    while(!ipq.empty()) {
        cout << ipq. top () << ' '; // 9 8 5 4 3 3 2 1 0
        ipq. pop();
    }
    cout << endl;
}
```

### 4.9 slist（仅存在于SGI STL）

#### 4.9.1 slist 概述

STL list 是个双向串行（double linked list）。SGI STL 另提供了一个单向串行（single linked list），名为 slist 。这个容器并不在标准规格之内，不过多做一些剖析，多看多学一些实作技巧也不错，所以我把它纳入本书范围。slist 和 list 的主要差别在于，前者的迭代器属于单向的 Forward Iterator，后者的迭代器属于双向的Bidirectional Iterator。为此， slist 的功能自然也就受到许多限制。不过，单向串行所耗用的空间更小，某些动作更快，不失为另一种选择。

slist 和 list 共同具有的一个相同特色是，它们的安插（insert）、移除（erase）、接合（splice）等动作并不会造成原有的迭代器失效（当然啦，指向被移除元素的那个迭代器，在移除动作发生之后肯定是会失效的）。注意，根据STL的习惯，**安插动作会将新元素安插于指定位置之前，而非之后**。然而做为一个单向串行， slist 没有任何方便的办法可以回头定出前一个位置，因此它必须从头找起。换句话说，除了 slist 起始处附近的区域之外，在其它位置上采用 insert 或 erase 操作函式，都是不智之举（但还是可以实现的，只是相比于list，slist的insert效率很低，时间O(n)）。这便是 slist 相较于 list之下的大缺点。为此， slist 特别提供了 insert_after() 和 erase_after() 供弹性运用。

基于同样的（效率）考虑， slist 不提供 push_back() ，只提供 push_front() 。因此 slist 的元素次序会和元素安插进来的次序相反。

#### 4.9.2 slist 的节点

slist 节点和其迭代器的设计，架构上比 list 复杂许多，运用了继承关系，因此在型别转换上有复杂的表现。这种设计方式在第5章 RB-tree 将再一次出现 。图 4-25 概述了 slist 节点和其迭代器的设计架构。

![《STL源码剖析》的笔记-slistnodeanditerator.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-slistnodeanditerator.png)

```c++
//单向串行的节点基本结构
struct__slist_node_base
{
    __slist_node_base* next;
};
//单向串行的节点结构
template <class T>
struct__slist_node : public __slist_node_base
{
    T data;
};
//全域函式：已知某一节点，安插新节点于其后。
inline __slist_node_base* __slist_make_link(
__slist_node_base* prev_node,
__slist_node_base* new_node)
{
    // 令 new节点的下一节点为 prev节点的下一节点
    new_node->next = prev_node->next;
    prev_node->next = new_node; //令 prev 节点的下一节点指向 new 节点
    return new_node;
}
//全域函式：单向串行的大小（元素个数）
inline size_t __slist_size (__slist_node_base* node)
{
    size_t result = 0;
    for ( ; node != 0; node = node->next)
    ++result;   //一个一个累计
    return result;
}
```

#### 4.9.3 slist 的迭代器

![《STL源码剖析》的笔记-slistiterator.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-slistiterator.png)

实际构造如下。请注意它和节点的关系（见图 4-25）。

```c++
//单向串行的迭代器基本结构
struct__slist_iterator_base
{
    typedef size_t size_type;
    typedef ptrdiff_tdifference_type ;
    typedef forward_iterator_tag iterator_category ;//注意，单向
    __slist_node_base* node;//指向节点基本结构
    __slist_iterator_base (__slist_node_base* x) : node (x) {}
    void incr () { node = node->next ; } // 前进一个节点
    bool operator== (const __slist_iterator_base& x) const {
        return node == x.node ;
    }
    bool operator!= (const __slist_iterator_base& x) const {
        return node != x.node;
    }
};
//单向串行的迭代器结构
template <class T, class Ref, class Ptr>
struct__slist_iterator : public __slist_iterator_base
{
    typedef __slist_iterator<T, T&, T*> iterator;
    typedef __slist_iterator<T, const T&, const T*> const_iterator ;
    typedef __slist_iterator<T, Ref, Ptr> self;
    typedef T value_type;
    typedef Ptr pointer;
    typedef Ref reference;
    typedef __slist_node<T> list_node;
    __slist_iterator(list_node* x ) : __slist_iterator_base( x ) {}
    // 呼叫 slist<T>::end() 时会造成 __slist_iterator(0) ，于是唤起上述函式。
    __slist_iterator () : __slist_iterator_base(0) {}
    __slist_iterator(const iterator&x ) : __slist_iterator_base( x.node ) {}
    reference operator* () const { return ((list_node*) node) ->data ; }
    pointer operator-> () const { return &( operator*() ); }
    self& operator++()
    {
        incr();//前进一个节点
        return *this;
    }
    self operator++(int)
    {
        self tmp = *this;
        incr(); //前进一个节点
        return tmp;
    }
    //没有实作 operator-- ，因为这是一个 forward iterator
};
```

注意，比较两个 slist 迭代器是否等同时（例如我们常在循环中比较某个迭代器是否等同于 slist.end() ），由于 __slist_iterator 并未对 operator== 实施多载化，所以会唤起 __slist_iterator_base::operator== 。根据其中之定义，我们知道，两个 slist 迭代器是否等同，视其 __slist_node_base* node 是否等同而定。

#### 4.9.4 slist 的数据结构

下面是 slist 源码摘要，我把焦点放在「单向串行之形成」的一些关键点上。

```c++
template <class T, class Alloc = alloc>
class slist
{
public:
    typedef T value_type;
    typedef value_type* pointer;
    typedef const value_type* const_pointer ;
    typedef value_type& reference;
    typedef const value_type& const_reference ;
    typedef size_t size_type;
    typedef ptrdiff_tdifference_type ;
    typedef __slist_iterator<T, T&, T*> iterator;
    typedef __slist_iterator<T, const T&, const T*> const_iterator ;
private:
    typedef __slist_node<T> list_node;
    typedef __slist_node_base list_node_base ;
    typedef __slist_iterator_base iterator_base ;
    typedef simple_alloc<list_node, Alloc> list_node_allocator ;
    static list_node* create_node (const value_type& x) {
        list_node* node =list_node_allocator::allocate ();//配置空间
        __STL_TRY {
            construct (&node->data, x);    //建构元素
            node->next = 0 ;
        }
        __STL_UNWIND(list_node_allocator::deallocate (node));
        return node;
    }
    static void destroy_node (list_node* node) {
        destroy (&node->data); //将元素解构
        list_node_allocator::deallocate (node); //释还空间
    }
private:
    list_node_base head ; // 头部。注意，它不是指标，是实物。
public:
    slist () { head.next = 0 ; }
    ~slist () { clear() ; }
public:
    iterator begin () { return iterator((list_node*) head.next ); }
    iterator end () { return iterator(0) ; }
    size_type size () const { return __slist_size(head.next ); }
    bool empty () const { return head.next == 0; }
    // 两个 slist互换：只要将 head 交换互指即可。
    void swap(slist& L)
    {
        list_node_base* tmp = head.next;
        head.next = L.head.next;
        L.head.next = tmp;
    }
public:
    // 取头部元素
    reference front () { return ((list_node*) head.next)->data ; }
    // 从头部安插元素（新元素成为 slist 的第一个元素）
    void push_front (const value_type& x) {
        __slist_make_link (&head,create_node(x));
    }
    // 注意，没有 push_back()
    // 从头部取走元素（删除之）。修改 head。
    void pop_front () {
        list_node* node = (list_node*) head.next;
        head.next = node->next;
        destroy_node(node);
    }
    ...
};
```

#### 4.9.5 slist 的元素操作

下面是一个小小练习：

```c++
// file: 4slist-test.cpp
#include <slist>
#include <iostream>
#include <algorithm>
using namespace std;
int main()
{
    int i;
    slist <int> islist;
    cout << "size=" << islist.size() << endl; // size=0
    islist. push_front(9);
    islist.push_front(1);
    islist.push_front(2);
    islist.push_front(3);
    islist.push_front(4);
    cout << "size=" << islist. size () << endl; // size=5
    slist<int>::iterator ite =islist. begin();
    slist<int>::iterator ite2=islist. end();
    for(; ite != ite2; ++ite)
        cout << *ite << ' '; // 4 3 2 1 9
    cout << endl;
    ite = find (islist.begin(), islist.end(), 1);
    if (ite!=0)
        islist. insert(ite, 99);
    cout << "size=" << islist.size() << endl; // size=6
    cout << *ite << endl; // 1
    ite =islist.begin();
    ite2=islist.end();
    for(; ite != ite2; ++ite)
        cout << *ite << ' '; // 4 3 2 99 1 9
    cout << endl;
    ite = find(islist.begin(), islist.end(), 3);
    if (ite!=0)
    cout << *(islist.erase(ite)) << endl; // 2
    ite =islist.begin();
    ite2=islist.end();
    for(; ite != ite2; ++ite)
        cout << *ite << ' '; // 4 2 99 1 9
    cout << endl;
}
```

首先依次序把元素 9,1,2,3,4安插到 slist ，实际结构呈现如图 4-26。接下来搜寻元素1，并将新元素 99安插进去，如图 4-27。注意，新元素被安插在插入点（元素 1）的前面而不是后面。接下来搜寻元素 3，并将该元素移除，如图 4-28。

![《STL源码剖析》的笔记-slist91234.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-slist91234.png)

![《STL源码剖析》的笔记-slistinsert.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-slistinsert.png)

![《STL源码剖析》的笔记-slisterase.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-slisterase.png)

如果你对于图4-26、图4-27、图4-28中的 end() 的画法感到奇怪，这里我要做一些说明。请注意，练习程序中一再以循环巡访整个 slist ，并以迭代器是否等于 slist.end() 做为循环结束条件，这其中有一些容易疏忽的地方，我必须特别提醒你。当我们呼叫 end() 企图做出一个指向尾端（下一位置）的迭代器，STL源码是这么进行的：

```c++
iterator end () { return iterator(0) ; }
```

这会因为源码中如下的定义：

```c++
typedef __slist_iterator<T, T&, T*> iterator;
```

而形成这样的结果：

```c++
__slist_iterator<T, T&, T*>(0);
```

从而因为源码中如下的定义：

```c++
//产生一个暂时对象，引发 ctor
__slist_iterator(list_node* x ) : __slist_iterator_base( x ) {}
```

而导致基础类别的建构：

```c++
__slist_iterator_base(0);
```

并因为源码中这样的定义：

```c++
struct__slist_iterator_base
{
    __slist_node_base* node;//指向节点基本结构
    __slist_iterator_base (__slist_node_base* x) : node (x) {}
    ...
};
```

而导致：

```c++
node(0);
```

因此我在图 4-26、图 4-27、图 4-28 中皆以悬空的方式表现 end()

## Chapter5 关联式容器（associative containers）

[C++ STL 中的 map 用红黑树实现，搜索效率是 O (lgN), 为什么不像 python 一样用散列表从而获得常数级搜索效率呢？](https://www.zhihu.com/question/24506208)

标准的STL关联式容器分为set（集合）和map（映射表）两大类，以及这两大类的衍生体 multiset（多键集合）和multimap（多键映射表）。这些容器的底层机制均以RB-tree（红黑树）完成，RB-tree也是一个独立容器，但并不开放给外界使用。

此外，SGI STL还提供了一个不在标准规格之列的关联式容器：hash table（散列表），以及以此 hashtable为底层机制而完成的 hash_set（散列集合）、hash_map（散列映射表）、hash_multiset（散列多键集合）、hash_multimap（散列多键映射表）

注：散列表很重要，它们在C++98未被纳入标准的原因是太迟了，C++11已经有了以散列表为基础的unordered_set和unordered_map，对应SGI STL的hash_set和hash_map

![《STL源码剖析》的笔记-sgistlcontainers.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-sgistlcontainers.png)

所谓关联式容器，观念上类似关联式数据库（实际上则简单许多）：每笔数据（每个元素）都有一个键值（key）和一个实值（value），set的键就是就是值，map可以键值分开，形成一种映射关系。当元素被插入到关联式容器中时，容器内部结构（可能是RB-tree，也可能是hash-table）便依照其键值大小，以某种特定规则将这个元素放置于适当位置。关联式容器没有所谓头尾（只有最大元素和最小元素），所以不会有所谓 push_back（）、push front（）、pop back（）、pop_front（）、begin（）、end（）这样的操作行为。

一般而言，关联式容器的内部结构是一个 balanced binary tree（平衡二又树），以便获得良好的搜寻效率。balanced binary tree有许多种类型，包括AvL-tree、RB-tree、AA-tree，其中最广泛应用于STL中的是RB-tree

### 5.1 树的导览

> 以下数据结构知识可以在专门讲数据结构的书中找到

树的定义与术语

二叉查找树

AVL树

单旋转与双旋转

### 5.2 RB-tree（红黑树）

红黑树满足的四条规则：

1. 每个节点不是红色就是黑色。
2. 根节点为黑色，NULL节点为黑色（有时也成NULL节点为树叶节点）
3. 如果节点为红，其子节点必须为黑
4. 任一节点至NULL（树尾端）的任何路径，所含之黑节点数必须相同。

#### 5.2.1 插入节点

假设插入节点是红色的，这样就不会违背规则：任意节点至NULL节点的任何路径所含黑色节点数目相同，这样只需要考虑是否违背规则：如果节点为红，其子节点必须为黑

RB-tree的插入后的旋转与AVL树的旋转很像，分为外侧和内侧，在内侧有需要双旋转，旋转不外乎左旋和右旋，旋转完之后重新染色，有可能会上溯至祖父节点，甚至更高

#### 5.2.4 RB-tree的迭代器

要成功地将RB-tree实现为一个泛型容器，迭代器的设计是一个关键。首先我们要考虑它的类别（category），然后要考虑它的前进（increment）、后退（decrement）、提领（dereference）、成员访问（member access）等操作

RB-tree迭代器属于双向迭代器，但不具备随机定位能力，其提领操作和成员访问操作与1st十分近似，较为特殊的是其前进和后退操作。注意，RB-tree迭代器的前进操作 operator++（）调用了基层迭代器的 increment（），RB-tree迭代器的后退操作 operator-（）则调用了基层迭代器的 decrement（），前进或后退的举止行为完全依据二叉搜索树的节点排列法则。比如某节点的increment()是找那个比当前节点更大的最小节点，它有可能在当前节点的右子树里面，也有可能某位祖先的右子树里面

#### 5.2.7 RB-tree的元素操作

本节主要只谈元素（节点）的插入和搜寻。

RB-tree提供两种插入操作insert_unique（）和 insert_equal（），前者表示被插入节点的键值（key）在整棵树中必须独一无二（因此，如果树中已存在相同的键值，插人操作就不会真正进行），后者表示被插入节点的键值在整棵树中可以重复，因此，无论如何插入都会成功（除非空间不足导致配置失败）。这里的insert_unique（）和 insert_equal（）就是STL中的set/map与multiset/multimap的实现基础。这两个函数都有数个版本，所有版本在插入完之后，会再调用一个平衡操作，用来旋转和改变颜色。

RB-tree是一个二叉搜索树，元素的搜寻正是其拿手项目。所以搜寻起来非常迅速，只需要在二叉搜索树中不断搜索即可，用时O(logN)

### 5.3 set

set的特性是，所有元素都会根据元素的键值自动被排序。这是因为RB-tree是二叉搜索树，迭代器的递增会指向比当前节点更大的最小节点，当用迭代器遍历整个set时，即为有序（升序）

set的元素不像map那样可以同时拥有实值（value）和键值（key），set元素的键值就是实值，实值就是键值。set不允许两个元素有相同的键值（而multiset允许）。

我们可以通过set的迭代器改变set的元素值吗？不行，因为set元素值就是其键值，关系到set元素的排列规则。如果任意改变set元素值，会严重破坏set组织。这就像我们不能在RB-tree中修改某节点的值一样，的确我们没有在哪篇教程中看到修改操作，一般只会说insert与erase操作。

稍后你会在set源代码之中看到，`set<T>::iterator`被定义为底层RB-tree的 const iterator，杜绝写人操作。换句话说，set iterators是一种constant iterators（相对于 mutable iterators）

set拥有与1ist相同的某些性质：当客户端对它进行元素新增操作（insert）或删除操作（erase）时，操作之前的所有迭代器，在操作完成之后都依然有效。当然，被删除的那个元素的迭代器必然是个例外。

由于RB-tree是一种平衡二又搜索树自动排序的效果很不错，所以标准的STL set即以RB-tree为底层机制。又由于set所开放的各种操作接口，RB-tree也都提供了，所以几乎所有的set操作行为，都只是转调用RB-tree的操作行为而已

**我们一般用set内部的find()成员方法，而不用泛型算法find()**，因为泛型算法find()内使用迭代器遍历，而set底层的RB-tree的迭代器的increment()略微复杂，从根节点开始，每次都需要找比当前节点更大的最小节点，用时O(n)，而find()成员方法从根节点开始，类似二分查找，用时O(logn)

下面是小小的set测试程序，

```c++
int main(){
    int i;
    int ia[5] = {0,2,3,4,5};
    set<int>::iterator ite1 = iset.begin();
    set<int>::iterator ite2 = iset.end();
    for(; ite1 != ite2; ++ite1){
        cout << *ite1;  // 0  2  3  4  5
    }
    //使用STL算法find（）来搜寻元素，可以有效运作，但不是好办法
    if(find(iset.begin(), iset.end(), 3) != iset.end()){
        cout << "3 found" << endl;  // 3 found
    }
    //面对关联式容器，应该使用其所提供的find函数来搜寻元素，会比
    //使用STL算法find（）更有效率，因为STL算法find（）只是循序搜寻
    if(iset.find(3) != iset.end()){
        cout << "3 found" << endl;  // 3 found
    }
    //企图通过迭代器来改变set元素，是不被允许的
    *iset.find(3) = 9;  //error，assignment of read-only location
}
```

### 5.4 map

map的特性是，所有元素都会根据元素的键值自动被排序。map的所有元素都是pair，同时拥有实值（value）和键值（key）。pair的第一元素被视为键值，第二元素被视为实值，map不允许两个元素拥有相同的键值（而multimap允许）。

我们可以通过map的迭代器改变map的元素内容吗？如果想要修正元素的键值，答案是不行，因为map元素的键值关系到map元素的排列规则。任意改变map元素键值将会严重破坏map组织。这就像我们不能在RB-tree中修改某节点的值一样，的确我们没有在哪篇教程中看到修改操作，一般只会说insert与erase操作。**但如果想要修正元素的实值，答案是可以，因为map元素的实值并不影响map元素的排列规则**。因此，map iterators既不是一种constant iterators，也不是一种 mutable iterators。

与set一样，map拥有和list相同的某些性质：当客户端对它进行元素新增操作（insert）或删除操作（erase）时，操作之前的所有迭代器，在操作完成之后都依然有效。当然，被删除的那个元素的迭代器必然是个例外。

与set一样，由于RB-tree是一种平衡二叉搜索树，自动排序的效果很不错，所以标准的STL map即以RB-tree为底层机制.又由于map所开放的各种操作接口，RB-tree也都提供了、所以几乎所有的map操作行为，都只是转调用RB-tree的操作行为而已。

**与set一样，我们一般用map内部的find()成员方法，而不用泛型算法find()**，因为泛型算法find()内使用迭代器遍历，而map底层的RB-tree的迭代器的increment()略微复杂，从根节点开始，每次都需要找比当前节点更大的最小节点，用时O(n)，而find()成员方法从根节点开始，类似二分查找，用时O(logn)

这里着重强调map的subscript（下标）操作，用法有两种，可能作为左值运用（内容可被修改），也可能作为右值运用（内容不可被修改），左值右值都适用的关键在于，返回值采用by reference传递形式。下标操作底层调用insert函数，若没有这个键，则直接创建，把键值对插入map中，并返回值，若有这个键，则返回值

### 5.5 multiset

multiset的特性以及用法和set完全相同，唯一的差别在于它允许键值重复，因此它的插入操作采用的是底层机制RB-tree的 lnsert equa1（）而非insert unique（）。

### 5.6 multimap

multimap的特性以及用法与map完全相同，唯一的差别在于它允许键值重复，因此它的插人操作采用的是底层机制RB-tree的 insert equa1（）而非insert unique（）。

### 5.7 hashtable

二又搜索树具有对数平均时间（logarithmic average time）的表现，但这样的表现构造在一个假设上：输入数据有足够的随机性。这一节要介绍一种名为hashtable（散列表）的数据结构，这种结构在插人、删除、搜寻等操作上也具有“常数平均时间”的表现，而且这种表现是以统计为基础，不需仰赖输入元素的随机性

#### 5.7.1 hashtable概述

使用某种映射函数，将大数映射为小数，散列函数就是这个作用，负责将某一元素映射为一个“大小可接受之索引”，使用散列函数会带来散列冲突/碰撞的问题

负载系数

线性探测，主集团/一次聚集

平方探测，次集团/二次聚集

双散列，消除次集团问题

再散列，一般取大于原来散列表两倍的第一个质数

开链/分离链表法，负载系数大于1

SGI STL的hashtable用的就是开链法

![《STL源码剖析》的笔记-separatechaining.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-separatechaining.png)

#### 5.7.2 hashtable的桶子（buckets）与节点（nodes）

SGI STL将hashtable的元素成为桶子，因为桶子里面可能会有多个元素（开链法），buckets是用vector实现的，桶子内的linked list并不是STL的list，而是自己实现并且维护的list。

#### 5.7.3 hashtable的迭代器

#### 5.7.4 hashtable的数据结构

虽然开链法并不要求哈希表大小必须为质数，但是SGI STL仍以质数设计大小，并且将28个质数（逐渐呈现大约两倍的关系）计算好了，存在于源码中，并提供了函数，用来查询在这28个质数中，最接近某数并大于某数的质数

#### 5.7.4 hashtable的构造与内存管理

resize即为再散列（rehashing），SGI STL对于resize的实现略微奇特，如果总元素个数（把新增元素也计入）大于buckets vector的大小，就resize，新的buckets vector大小为下一个质数（在28个质数中的）

resize的主要流程如下：

1. 建立下一个质数大小的tmp hashtable
2. 对原hashtable中的每个元素，重新哈希（rehashing），放入tmp中
3. swap原hashtable与tmp hashtable，

#### 5.7.7 hash functions

对于 string、double、float等类型，SGI hashtable没有定义散列函数，所以要用的话必须自己设计

### 5.8 hash_set

虽然STL只规范复杂度与接口，并不规范实现方法，但 STL set多半以RB-tree为底层机制.SGI则是在STL标准规格之外另又提供了一个所谓的hash set，以 hashtable为底层机制。由于 hash set所供应的操作接口hashtable都提供了，所以几乎所有的 hash set操作行为，都只是转调用hashtab1e的操作行为而已。

运用set，为的是能够快速搜寻元素。这一点，不论其底层是RB-tree或是hash table，都可以达成任务。但是请注意，RB-tree有自动排序功能而 hashtab1e没有，反应出来的结果就是，set的元素有自动排序功能而 hash set没有

set的元素不像map那样可以同时拥有实值（value）和键值（key），set元素的键值就是实值，实值就是键值。这一点在 hash set中也是一样的hash set的使用方式，与set完全相同。

### 5.9 hash_map

sGI在STL标准规格之外，另提供了一个所谓的hash_map，以 hashtab1e为底层机制。由于 hash_map所供应的操作接口，hashtable都提供了，所以几乎所有的hash_map操作行为，都只是转调用 hashtable的操作行为而已。

运用map，为的是能够根据键值快速搜寻元素这一点，不论其底层是 RB-tree或是 hashtab1e，都可以达成任务。但是请注意，RB-tree有自动排序功能而hashtable没有，反应出来的结果就是，map的元素有自动排序功能而 hash_map没有

map的特性是，每一个元素都同时拥有一个实值（value）和一个键值（key）。这一点在 hash_map中也是一样的 hash_map的使用方式，和map完全相同

### 5.10 hash_multiset

hash multiset的特性与 multiset完全相同，唯一的差别在于它的底层机制是 hashtable。也因此，hash multiset的元素并不会被自动排序

hash_multiset和 hash set实现上的唯一差别在于，前者的元素插入操作采用底层机制 hashtab1e的 insert_equa1（），后者则是采用 insert.unique（）

### 5.11 hash_multimap

hash_multimap的特性与 multimap完全相同，唯一的差别在于它的底层机制是 hashtab1e。也因此，hash_mu map的元素并不会被自动排序

hash_multimap和 hash_map实现上的唯一差别在于，前者的元素插入操作采用底层机制 hashtable的 insert_equal（），后者则是采用 insert_unique（）

## Chapter6 算法（algorithm）

### 6.1 算法概览

算法，问题之解法也。

以有限的步骤，解决逻辑或数学上的问题，这一专门科目我们称为算法（algorithm）。

#### 6.1.2 STL 算法总览

图6-1将所有的STL算法（以及一些非标准的 SGI STL算法）的名称、用途文件分布等等，依算法名称的字母顺序列表。表格之中凡是不在STL标准规格之列的SGI专属算法，都以☆加以标示（以下“质变”栏意指 mutating，意思是“会改变其操作对象之内容”）

![《STL源码剖析》的笔记-algorithm1.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-algorithm1.png)

![《STL源码剖析》的笔记-algorithm2.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-algorithm2.png)

![《STL源码剖析》的笔记-algorithm3.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-algorithm3.png)

![《STL源码剖析》的笔记-algorithm4.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-algorithm4.png)

#### 6.1.3 质变算法 mutating algorithms—会改变操作对象之值

所有的STL算法都作用在由迭代器[first，last)所标示出来的区间上。所谓“质变算法”，是指运算过程中会更改区间内（迭代器所指）的元素内容。诸如拷贝（copy）、互换（swap）、替换（replace）、填写（fill）、删除（remove）、排列组合（permutation）、分割（partition）、随机重排（random shuffling）、排序（sort）等算法，都属此类 。

#### 6.1.4 非质变算法 nonmutating algorithms—不改变操作对象之值

所有的STL算法都作用在由迭代器【first,last）所标示出来的区间上。所谓“非质变算法”，是指运算过程中不会更改区间内（迭代器所指）的元素内容诸如查找（find）、匹配（search）、计数（count）、巡访（for each）、比较（equal,mismatch）、寻找极值（max,min）等算法，都属此类。但是如果你在 for_each（巡访每个元素）算法身上应用一个会改变元素内容的仿函数（functor）。

#### 6.1.5 STL 算法的一般形式

所有泛型算法的前两个参数都是一对迭代器（iterators），通常称为 first和last，用以标示算法的操作区间。STL习惯采用前闭后开区间（或称左涵盖区间）表示法，写成【first,last），表示区间涵盖 first至last（不含last）之间的所有元素。当 first=last时，上述所表现的便是一个空区间这个【first,last）区间的必要条件是，必须能够经由 increment（累加）操作符的反复运用，从 first到达last。编译器本身无法强求这一点。如果这个条件不成立，会导致未可预期的结果

每一个STL算法的声明，都表现出它所需要的最低程度的迭代器类型。

将无效的迭代器传给某个算法，虽然是一种错误，却不保证能够在编译时期就被捕捉出来，因为所谓“迭代器类型”并不是真实的型别，它们只是 function template的一种型别参数（type parameters）

许多STL算法不只支持一个版本。这一类算法的某个版本采用缺省运算行为，另一个版本提供额外参数，接受外界传入一个仿函数（functor），以便采用其他策略。例如 unique（）缺省情况下使用 equality操作符来比较两个相邻元素，但如果这些元素的型别并未供应 equality操作符，或如果用户希望定义自己的 equality操作符，便可以传一个仿函数（functor）给另一版本的 unique（）。

所有的数值（numerIc）算法，包括 adjacent__difference（），accumulate（）inner_product（），partial_sum（）等等，都实现于SGI`<stl_numeric.h>`之中，这是个内部文件，STL规定用户必须包含的是上层的`<numeric>`，其他STL算法都实现于SGI的`<stl_algo>`和`<stl_algobase.h>`文件中，也都是内部文件；欲使用这些算法，必须先包含上层相关头文件`<algorithm>`

### 6.2 算法的泛化过程

如何将算法独立于其所处理的数据结构之外，不受数据结构的羁绊，思想层面就不是那么简单了。如何设计一个算法，使它适用于任何（或大多数）数据结构呢？换个说法，我们如何在即将处理的未知的数据结构（也许是 array，也许是 vector，也许是1ist，也许是 deque…）上，正确地实现所有操作呢？

关键在于，只要把操作对象的型别加以抽象化，把操作对象的标示法和区间目标的移动行为抽象化，整个算法也就在一个抽象层面上工作了。整个过程称为算法的泛型化（generalized），简称泛化。

使用template，可以接收不同类型的参数，由指针抽象成迭代器，迭代器是一种行为类似指针的对象，是一种smart pointers，于是可以得到STL的泛型算法

### 6.3 数值算法 <stl_numeric>

accumulate 累加，第一版本用加法定义，第二版本用外界提供的二元仿函数，两个版本都有一个形参用来传入初值，这是为了在区间为空时仍有返回值，第二版本的二元操作符不必满足交换律和结合律，因为所有运算顺序都有明确规定

adjacent_difference 储存第一元素值，然后储存后继元素之差值，与adjacent_difference互为逆运算，第一版本用减法定义差值，第二版本采用外界提供的二元仿函数

inner_product: 能够计算一般内积，两个版本都有一个形参用来传入初值，第二版本有两个仿函数用来代替加法和乘法，不必满足交换律和结合律

partial_sum 计算局部总和，与adjacent_difference互为逆运算，如果令输出的位置为区间本身的首位，则是就地（in-place）计算，这种情况下是质变算法（mutating algorithm）

power 计算某数的n幂次方，推荐文章[C++ 中位运算的使用方法](https://blog.csdn.net/a1351937368/article/details/77746574/)、[STL 源码分析之 power 算法](https://blog.csdn.net/u012062760/article/details/46401115)、[STL 系列之七 快速计算 x 的 n 次幂 power () 的实现](https://blog.csdn.net/MoreWindows/article/details/7174143)

power: 计算指数 (C++ 中位运算的使用方法 ; STL 源码分析之 power 算法 ; 快速计算 x 的 n 次幂)

itoa：SGI专属算法，不在STL标准之列，用来设定某个区间的内容，使其内每个元素从指定的value值开始，呈现递增状态，改变了区间内容，所以是质变算法

### 6.4 基本算法 <stl_algobase.h>

STL标准规格中并没有区分基本算法或复杂算法，然而SGI却把常用的一些算法定义于`<stl_algobase.h>`之中，其它算法定义于`<stl_algo.h>`中。

equal,fill,fill_n,iter_swap,lexicographical_compare,max,min,mismatch,swap

有些要保证第一序列长度大于第二序列长度，否则会发生不可预期的行为

主要讲一讲copy，这是一个为了提高效率的绝佳代码实现

#### 6.4.3 copy——强化效率无所不用其极

不论是对客端程序或对STL内部而言，copy（）都是一个常常被调用的函数由于copy进行的是复制操作，而复制操作不外乎运用 assignment operator或copy constructor（copy算法用的是前者），但是某些元素型别拥有的是 trivialassignment operator，因此，如果能够使用内存直接复制行为（例如C标准函数memmove或 memcpy），便能够节省大量时间。为此，SGI STL的copy算法用尽各种办法，包括函数重载（function overloading）、型别特性（type traits）、偏特化（partial specialization）等编程技巧，无所不用其极地加强效率。图62表示整个coy（）操作的脉络。配合稍后出现的源代码，可收一目了然之效

![《STL源码剖析》的笔记-copy.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-copy.png)

copy算法可将输人区间【first,last）内的元素复制到输出区间【resultresult+（last-first））内。也就是说，它会执行赋值操作*result=*first*（result+1）=*（first+1），…依此类推。返回一个迭代器：
result+（last-first）。copy对其 template参数所要求的条件非常宽松。其输入区间只需由 inputiterators构成即可，输出区间只需由 Outputiterator构成即可这意味着你可以使用copy算法，将任何容器的任何一段区间的内容，复制到任何容器的任何一段区间上，如图6-3所示对于每个从0到last-first（不含）的整数n，copy执行赋值操作*（result+n）=*（first+n）。赋值操作是向前（亦即累加n）推进的。

如果输人区间和输出区间完全没有重叠，当然毫无问题，否则便需特别注意为什么图6-3第二种情况（可能）会产生错误？从稍后即将显示的源代码可知，copY算法是一一进行元素的赋值操作，如果输出区间的起点位于输入区间内，copy算法便（可能）会在输入区间的（某些）元素尚未被复制之前，就覆盖其值，导致错误结果。在这里我一再使用“可能”这个字眼，是因为，如果copy算法根据其所接收的迭代器的特性决定调用 memmove（）来执行任务，就不会造成上述错误，因为 memmove（）会先将整个输人区间的内容复制下来，没有被覆盖的危险

![《STL源码剖析》的笔记-copyeg.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-copyeg.png)

copy 改变的是 [result,result+(last-first)) 中的迭代器所指对象，而并非更改迭代器本身。它会为输出区间的元素赋予新值，而不是产生新的元素。它不能改变输出区间的迭代器个数。因此不能将元素插入空容器中。如果真想把元素（而非赋值）序列之内，要么使用序列容器的insert成员函数，要么使用copy算法并搭配insert_iterator（8.3.1节）

#### 6.4.4 copy_backward

这个算法的考虑以及实现上的技巧与copy（）十分类似，其操作示意于图6-4，将【first，last）区间内的每一个元素，以逆行的方向复制到以 result-1为起点，方向亦为逆行的区间上。换句话说，copy-backward算法会执行赋值操作*（result-1）=*（last-1），*（result-2）=*（last-2），…依此类推。返回一个迭代器：result-（last-first）。copy backward所接受的迭代器必须是 Bidirection| Iterators，才能够“倒行逆施”。你可以使用 copy backward算法，将任何容器的任何一段区间的内容，复制到任何容器的任何一段区间上。如果输人区间和输出区间完全没有重叠，当然毫无问题，否则便需特别注意，如图6-4

![《STL源码剖析》的笔记-copybackward.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-copybackward.png)

### 6.5 set 相关算法

STL一共提供了四种与set（集合）相关的算法，分别是并集（union）、交集（Intersection）、差集（difference）、对称差集（symmetric difference）

所谓set，可细分为数学上的定义和STL的定义两种，数学上的set允许元素重复而未经排序，例如{1，1，4, 6，3}，STL的定义（也就是set容器，见5。3节）则要求元素不得重复，并且经过排序，例如{1，3，4，6}。**本节的四个算法所接受的set，必须是有序区间（sorted range），元素值得重复出现**。换句话说，它们可以接受STL的set/multiset容器作为输入区间。

SGI STL另外提供有 hash set/hash multiset两种容器，以 hashtab1e为底层机制（见5。8节、5.10节），其内的元素并未呈现排序状态，所以虽然名称之中也有set字样，却不可以应用于本节的四个算法

**本节四个算法都至少有四个参数，分别表现两个set区间**。以下所有说明都以s1代表第一区间【irst1，last1），以s2代表第二区间【first2，last2）。每个set算法都提供两个版本（但稍后展示的源代码只列出第一版本），第二版本允许用户指定“`a<b`”的意义，因为这些算法判断两个元素是否相等的依据，完全靠“小于”运算。是的，知道何谓“小于”，就可以推导出何谓“等于”

请注意，当集合（set）允许重复元素的存在时，并集、交集、差集、对称差集的定义，都与直观定义有些微的不同

#### 6.5.1 set_union

算法set_union可构造s1、s2之并集。也就是说，它能构造出集合s1 U s2，此集合内含s1或s2内的每一个元素。s1、s2及其并集都是以排序区间表示。返回值为一个迭代器，指向输出区间的尾端。

由于s1和s2内的每个元素都不需唯一，因此，如果某个值在s1出现n次，在S2出现m次，那么该值在输出区间中会出现max（m,n）次，其中n个来自s1，其余来自s2。在 STL set容器内，m<=1且n≤1。

set union是一种稳定（stable）操作，意思是输人区间内的每个元素的相对顺序都不会改变。set union有两个版本，差别在于如何定义某个元素小于另个元素。第一版本使用 operator<进行比较，第二版本采用仿函数comp进行比较

![《STL源码剖析》的笔记-setunion.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-setunion.png)

#### 6.5.2 set_intersection

算法set_intersection可构造s1、s2之交集。也就是说，它能构造出集合s1 ∩ s2，此集合内含同时出现于s1和s2内的每一个元素。s1、s2及其交集都是以排序区间表示。返回值为一个迭代器，指向输出区间的尾端。

由于sl和s2内的每个元素都不需唯一，因此，如果某个值在s1出现n次，在s2出现m次，那么该值在输出区间中会出现min（m,n）次，并且全部来自s1。在 STL set容器内，m≤1且n≤1。

set_intersection是一种稳定（stable）操作，意思是输出区间内的每个元素的相对顺序都和s1内的相对顺序相同。它有两个版本，差别在于如何定义某个元素小于另一个元素。第一版本使用 operator<进行比较，第二版本采用仿函数comp进行比较

![《STL源码剖析》的笔记-setintersection.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-setintersection.png)

#### 6.5.3 set_difference

算法set_difference可构造s1、s2之差集。也就是说，它能构造出集合s1-s2，此集合内含“出现于s1但不出现于s2”的每一个元素。S1、s2及其交集都是以排序区间表示。返回值为一个迭代器，指向输出区间的尾端。

由于s1和s2内的每个元素都不需唯一，因此如果某个值在s1出现n次，在s2出现m次，那么该值在输出区间中会出现max（n-m，0）次，并且全部来自s1。在 STL set容器内，m≤1且n≤1。

set_difference是一种稳定（stable）操作，意思是输出区间内的每个元素的相对顺序都和s1内的相对顺序相同。它有两个版本，差别在于如何定义某个元素小于另一个元素第一版本使用 operator<进行比较，第二版本采用仿函数comp进行比较。

![《STL源码剖析》的笔记-setdifference.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-setdifference.png)

#### 6.5.4 set_symmetric_difference

算法 set symmetric difference可构造s1、s2之对称差集。也就是说，它能构造出集合（s1-s2）U（s2-S1），此集合内含“出现于s1但不出现于s2以及”出现于S2但不出现于s1“的每一个元素。s1、s2及其交集都是以排序区间表示。返回值为一个迭代器，指向输出区间的尾端。

由于s1和S2内的每个元素都不需唯一，因此如果某个值在s1出现n次在s2出现m次，那么该值在输出区间中会出现|n-m|次。如果n>m，输出区间内的最后n-m个元素将由s1复制而来，如果`n<m`则输出区间内的最后m-n个元素将由s2复制而来。在 STL set容器内，m≤1且n≤1。

set_symmetric_difference是一种稳定（stable）操作，意思是输入区间内的元素相对顺序不会被改变。它有两个版本，差别在于如何定义某个元素小于另一个元素。第一版本使用 operator<进行比较，第二版本采用仿函数comp

![《STL源码剖析》的笔记-setsymmetricdifference.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-setsymmetricdifference.png)

### 6.6 heap算法

四个heap相关算法已于4.7.2节的`<stl_heap.h>`介绍过：make_heap（），pop_heap（），push heap（），sort_heap（）。喔，是的，当然，你猜对了，SGI STI算法所在的头文件`<stl_algo.h>`内包含了`<stl_heap.h>`

### 6.7 其它算法定

定义于SGI`<stl_algo.h>`内的所有算法，除了set/heap相关算法已于前两节介绍过，其余都安排在这一节。其中有很单纯的循环遍历，也有很复杂的快速排序。运作逻辑相对单纯者，被我安排在6.7.1小节，其它相对比较复杂者，每个算法各安排一个小节

#### 6.7.1 单纯的数据处理

这一小节所列的算法，都只进行单纯的数据移动、线性查找、计数、循环遍历逐一对元素施行指定运算等操作。它们的运作逻辑都相对单纯，直观而易懂。我把对它们的说明直接写成源代码注释，必要时另加图片示意。

深入源代码之前，先观察每一个算法的表现，是个比较好的学习方式。以下程序示范本节每一个算法的用法。程序中有时使用STL内建的仿函数（functors，如less，greater，equeal_to）和配接器（adapters，如bind2nd），有时使用自定义的仿函数（如 display，even_by_two）；仿函数相关技术请参考1.9.6节和第7章，配接器相关技术请参考第8章。

adjacent_find 找出第一组满足条件的相邻元素。这里所谓的条件，在版本一中是指“两元素相等”，在版本二中允许用户指定一个二元运算，两个操作数分别是相邻的第元素和第二元素。

count 运用 equality操作符，将【first，last）区间内的每一个元素拿来和指定值value比较，并返回与va1ue相等的元素个数

count_if 将指定操作（一个仿函数）pred实施于【irst,last）区间内的每一个元素身上，并将“造成pred之计算结果为true”的所有元素的个数返回

find 根据 equality操作符，循序查找【first,last）内的所有元素，找出第一个匹配“等同（equality）条件”者。如果找到，就返回一个 Inputiterator指向该元素，否则返回迭代器last

find_if 根据指定的prea运算条件（以仿函数表示），循序査找【first,ast）内的所有元素，找出第一个令pred运算结果为true者。如果找到就返回一个Inputiterator指向该元素，否则返回迭代器last

find_end 在序列一【first1，last1）所涵盖的区间中，查找序列二 first2，last2）的最后一次出现点。如果序列一之内不存在“完全匹配序列二”的子序列，便返回迭代器last1.此算法有两个版本，版本一使用元素型别所提供的 equality操作符，版本二允许用户指定某个二元运算（以仿函数呈现），作为判断元素相等与否的依据。

find_first_of 本算法以【first2，last2）区间内的某些元素作为查找目标，寻找它们在irst1，last1）区间内的第一次出现地点。举个例子，假设我们希望找出字符序列 synesthesia的第一个元音，我们可以定义第二序列为 aeiou。此算法会返回一个Forwardlterator，指向元音序列中任一元素首次出现于第一序列的地点，此例将指向字符序列的第一个e。如果第一序列并未内含第二序列的任何元素，返回的将是last1.本算法第一个版本使用元素型别所提供的 equality操作符，第二个版本允许用户指定一个二元运算pred

![《STL源码剖析》的笔记-findendfindfirstof.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-findendfindfirstof.png)

for_each 将仿函数f施行于【first，last）区间内的每一个元素身上.f不可以改变元素内容，因为 first和last都是 Inputiterators，不保证接受赋值行为（assignment）·如果想要一一修改元素内容，应该使用算法 trans form（）。f可返回一个值，但该值会被忽略。

generate 将仿函数gen的运算结果填写在【first，last）区间内的所有元素身上。所谓填写，用的是迭代器所指元素之 assignment操作符。

generate_n 将仿函数gen的运算结果填写在从迭代器 first开始的n个元素身上，所谓填写，用的是迭代器所指元素的 assignment操作符

includes（应用于有序区间）判断序列二S2是否“涵盖于”序列一S1.Sl和S2都必须是有序集合，其中的元素都可重复（不必唯一）。所谓涵盖，意思是“S2的每一个元素都出现于s1”。由于判断两个元素是否相等，必须以less或 greater运算为依据（当Sl元素不小于S2元素且S2元素不小于S1元素，两者即相等；或说当S1元素不大于S2元素且S2元素不大于S1元素，两者即相等），因此配合着两个序列S1和S2的排序方式（递增或递减），includes算法可供用户选择采用less或 greater进行两元素的大小比较（comparison）

![《STL源码剖析》的笔记-includes.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-includes.png)

max_element 这个算法返回一个迭代器，指向序列之中数值最大的元素。

min_element 这个算法返回一个迭代器，指向序列之中数值最小的元素。

merge（应用于有序区间）将两个经过排序的集合S1和S2，合并起来置于另一段空间。所得结果也是一个有序（sred）序列。返回一个迭代器，指向最后结果序列的最后一个元素的下位置。图6.6c展示 merge算法的工作原理。

![《STL源码剖析》的笔记-mergealgo.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-mergealgo.png)

partition 会将区间【first，last）中的元素重新排列。所有被一元条件运算prea判定为true的元素，都会被放在区间的前段，被判定为 false的元素，都会被放在区间的后段。这个算法并不保证保留元素的原始相对位置。代码实现使用了双指针法。如果需要保留原始相对位置，应使用 stable_partition

![《STL源码剖析》的笔记-partition.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-partition.png)

remove　移除【first,last）之中所有与value相等的元素。**这一算法并不真正从容器中删除那些元素（换句话说容器大小并未改变），而是将每一个不与value相等（也就是我们并不打算移除）的元素轮番赋值给 first之后的空间**。返回值Forwarditerator标示出重新整理后的最后元素的下一位置。例如序列{0，10，2，0，3，0，4}，如果我们执行 remove（），希望移除所有0值元素，执行结果将是{1，2，3，4，0，3，0，4}。每一个与0不相等的元素，1，2，3，4，分别被拷贝到第一、二、三、四个位置上。第四个位置以后不动，换句话说是第四个位置之后是这一算法留下的**残余数据**。返回值 Forwarditerator指向第五个位置。如果要删除那些残余数据，可将返回的迭代器交给区间所在之容器的 erase（）member function，注意，array不适合使用 remove（）和 remove_if（），因为 array无法缩小尺寸，导致残余数据永远存在。对aray而言，较受欢迎的算法是 remove_copy（）和remove_copy_if（）。

![《STL源码剖析》的笔记-removealgo.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-removealgo.png)

remove_copy 移除【first，last）区间内所有与value相等的元素。它并不真正从容器中删除那些元素（换句话说，原容器没有任何改变），而是将结果复制到一个以 result标示起始位置的容器身上新容器可以和原容器重叠，但如对新容器实际给值时，超越了旧容器的大小，会产生无法预期的结果。返回值 Outputiterator指出被复制的最后元素的下一位置

remove_if 移除【first，last区间内所有被仿函数pred核定为true的元素。它并不真正从容器中删除那些元素（换句话说，容器大小并未改变，请参考 remove（）每一个不符合pre条件的元素都会被轮番赋值给 first之后的空间。返回值Forwarditerator标示出重新整理后的最后元素的下一位置。此算法会留有一些残余数据，如果要删除那些残余数据，可将返回的迭代器交给区间所在之容器的 erase（）member function注意，array不适合使用 remove（）和 remove_if（），因为 array无法缩小尺寸，导致残余数据永远存在。对 array而言，较受欢迎的算法是remove_copy（）和 remove_copy_if（）

remove__if 移除[first,1ast)区间内所有被仿函数pred评估为true的元素.它并不真正从容器中删除那些元素(换句话说原容器没有任何改变),而是将结果复制到一个以 result标示起始位置的容器身上.新容器可以和原容器重叠,但如果针对新容器实际给值时,超越了旧容器的大小,会产生无法预期的结果.返回值Outputterator指出被复制的最后元素的下一位置.

replace

replace_copy

replace_if

replace_copy_if

reverse 将序列【first,last）的元素在原容器中颠倒重排。例如序列{0，1，1，3，5}颠倒重排后为（53，，1，0}。迭代器的双向或随机定位能力，影响了这个算法的效率，所以设计为双层架构（呵呵，老把戏了）

reverse_copy

rotate 将[first,middle)内的元素和[middle,last)内的元素互换.middle所指的元素会成为容器的第一个元素.如果有个数字序列{1,2,3,4,5,6,7),对元素3做旋转操作,会形成{3,4,6,7,1,2).看起来这和 swap ranges()功能颇为近似,但swap-ranges()只能交换两个长度相同的区间,rotate()可以交换两个长度不同的区间,如图6-6g所示.迭代器的移动能力，影响了这个算法的效率，所以设计为双层架构（呵呵，老把戏

![《STL源码剖析》的笔记-rotatealgo.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-rotatealgo.png)

![《STL源码剖析》的笔记-rotatebytwoiterators.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-rotatebytwoiterators.png)

rotate_copy 行为类似 rotate(),但产生出来的新序列会被置于 result所指出的容器中返回值 Outputiterator指向新产生的最后元素的下一位置.原序列没有任何改变由于它不需要就地(in-place)在原容器中调整内容,实现上也就简单得多旋转操作其实只是两段元素彼此交换,所以只要先把后段复制到新容器的前端,再把前段接续复制到新容器,即可

search 在序列一[first1,last1)所涵盖的区间中,查找序列二[first2,1ast2)的首次出现点.如果序列一内不存在与序列二完全匹配的子序列,便返回迭代器last1·版本一使用元素型别所提供的 equality操作符,版本二允许用户指定某个二元运算(以仿函数呈现),作为判断相等与否的依据.以下只列出版本一的源代码.

search_n 在序列[First,last)所涵盖的区间中,查找"连续 count个符合条件之元素"所形成的子序列,并返回一个迭代器指向该子序列起始处.如果找不到这样的子序列,就返回迭代器1ast.上述所谓的"某条件",在 search_n版本一指的是相等条件"equality",在 search_n版本二指的是用户指定的某个二元运算(以仿函数呈现

例如，面对序列（10，8，8，7，2，8，7，2，2，8，7，0}，查找“连续两个8”所形成的子序列起点，可以这么写：

```c++
iter1 = search_n(iv.begin(), iv.end(), 2, 8);
```

查找“连续三个小于8的元素”所形成的子序列起点，可以这么写：

```c++
iter2 = search_n(iv.begin(), iv.end(), 3, 8, less<int>());
```

![《STL源码剖析》的笔记-searchn.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-searchn.png)

swap_ranges 将[first1,1ast1)区间内的元素与"从 first2开始、个数相同"的元素互相交换.这两个序列可位于同一容器中,也可位于不同的容器中.如果第二序列的长度小于第一序列,或是两序列在同一容器中且彼此重叠,执行结果未可预期. 此箅法返回一个迭代器,指向第二序列中的最后一个被交换元素的下一位置.

transform 第一版本以仿函数op作用于[first,1ast)中的每一个元素身上,并以其结果产生出一个新序列.第二版本以仿函数 binary_op作用于一双元素身上(其中一个元素来自[first1,last),另一个元素来自"从 first2开始的序列"),并以其结果产生出一个新序列.如果第二序列的元素少于第一序列,执行结果未可预期trans form()的两个版本都把执行结果放进迭代器 result所标示的容器中result也可以指向源端容器,那么 trans form()的运算结果就会取代该容器内的元素.返回值 Outputiterator将指向结果序列的最后元素的下一位置

unique 能够移除(remove)重复的元素.每当在[first,last]内遇有重复元素群,它便移除该元素群中第一个以后的所有元素.注意,unique只移除相邻的重复元素,如果你想要移除所有(包括不相邻的)重复元素,必须先将序列排序,使所有重复元素都相邻.unique会返回一个迭代器指向新区间的尾端,新区间之内不含相邻的重复元素.这个算法是稳定的(stable),亦即所有保留下来的元素,其原始相对次序不变.事实上unique并不会改变[first,last)的元素个数,有一些残余数据会留下来,如图6-61所示.情况类似remove算法,请参考先前对remove的讨论.unique有两个版本，因为所谓“相邻元素是否重复”可有不同的定义。第一版本使用简单的相等（equality）测试，第二版本使用一个Binary Predicate binary_pred做为测试准则。

![《STL源码剖析》的笔记-unique.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-unique.png)

unique_copy 算法unique_copy可从[first,last)中将元素复制到以result开头的区间上:如果面对相邻重复元素群,只会复制其中第一个元素.返回的迭代器指向以result开头的区间的尾端.与其它名为*_copy的算法一样,unique-copy乃是unique的一个复制式版本,所以它的特性与unique完全相同(请参考图6-6L),只不过是将结果输出到另一个区间而已.unique_copy有两个版本,因为所谓"相邻元素是否重复"可有不同的定义.第一版本使用简单的相等(equality)测试,第二版本使用一个Binary Predicate binary-pred作为测试准则.

#### 6.7.2 lower_bound（应用于有序区间）

这是二分查找(binary search)的一种版本,试图在已排序的(first,last)中寻找元素value.如果[first,last)具有与value相等的元素(s),便返回一个迭代器,指向其中第一个元素.如果没有这样的元素存在,便返回"假设这样的元素存在时应该出现的位置".也就是说,它会返回一个迭代器,指向第一个"不小于value"的元素.如果value大于[first,last)内的任何一个元素,则返回last.以稍许不同的观点来看1ower_bouna,其返回值是"在不破坏排序状态的原则下,可插入value的第一个位置".见图6-7.

![《STL源码剖析》的笔记-lowerboundandupperbound.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-lowerboundandupperbound.png)

#### 6.7.3 upper_bound（应用于有序区间）

算法upper_bound是二分查找（binary search）法的一个版本。它试图在已排序的（first，last）中寻找value。更明确地说，它会返回“在不破坏顺序的情况下，可插入value的最后一个合适位置”。见图6-7。

由于STL规范“区间圈定”时的起头和结尾并不对称（是的，【first，last）包含first但不包含last），所以upper_bound与1ower_bound的返回值意义大有不同。如果你查找某值，而它的确出现在区间之内，则lower_bound返回的是一个指向该元素的迭代器。然而upper-bound不这么做，因为upper_bound所返回的是在不破坏排序状态的情况下，value可被插人的“最后一个”合适位置。如果value存在，那么它返回的迭代器将指向value的下一位置，而非指向value本身。

#### 6.7.4 binary_search(应用于有序区间)

算法binary_search是一种二分查找法,试图在已排序的(first,last)中寻找元素value.如果[first,last)内有等同于value的元素,便返回true否则返回false.

返回单纯的boo1或许不能满足你,前面所介绍的lower_bound和upper_bound能够提供额外的信息.**事实上binary_search便是利用lower-bound先找出"假设value存在的话,应该出现的位置",然后再对比该位置上的值是否为我们所要查找的目标,并返回对比结果.**

binary_search的第一版本采用operator<进行比较,第二版本采用仿函数comp进行比较.

#### 6.7.5 next_permutation

STL提供了两个用来计算排列组合关系的算法,分别是next-permucation和prevpermutation.首先我们必须了解什么是"下一个"排列组合,什么是"前一个"排列组合.考虑三个字符所组成的序列(a,b,c).这个序列有六个可能的排列组合:abc,acb,bac,bca,cab,cba.这些排列组合根据less-than操作符做字典顺序(lexicographical)的排序.也就是说,abc名列第一,因为每一个元素都小于其后的元素.acb是次一个排列组合,因为它是固定了a(序列内最小元素)之后所做的新组合,同样道理,那些固定b(序列内次小元素)而做的排列组合,在次序上将先于那些固定c而做的排列组合.以bac和bca为例,bac在bca之前,因为序列ac小于序列ca.面对bca,我们可以说其前一个排列组合是bac,而其后一个排列组合是cab.序列abc没有"前一个"排列组合,cba没有"后一个"排列组合.

next_permutation()会取得[first,last)所标示之序列的下一个排列组合,如果没有下一个排列组合,便返回false:否则返回true.

这个算法有两个版本.版本一使用元素型别所提供的less-than操作符来决定下一个排列组合,版本二则是以仿函数comp来决定.

稍后即将出现的实现，简述如下，符号表示如图6-8所示。首先，从最尾端开始往前寻找两个相邻元素，令第一元素为`*i`，第二元素为`*ii`，且满足`*i<*ii`。找到这样一组相邻元素后，再从最尾端开始往前检验，找出第一个大于`*i`的元素，令为`*j`，将i，j元素对调，再将ii之后的所有元素颠倒排列。此即所求之“下一个”排列组合。

举个实例，假设我手上有序列（0，1，2，3，4），图6-9便是套用上述演算法则，一步一步获得的“下一个”排列组合。图中只框出那符合“第一元素为`*i`，第二元素为`*ii`，且满足`*i<*ii`的相邻两元素，至于寻找适当的j、对调、逆转等操作并未显示出。

![《STL源码剖析》的笔记-nextpermutation.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-nextpermutation.png)

#### 6.7.6 prev_permutation

所谓"前一个"排列组合,其意义已在上一节阐述,实际做法简述如下,其中所用的符号如图6-8所示.首先,从最尾端开始往前寻找两个相邻元素,令第一元素为`*i`,第二元素为`*ii`,且满足`*i>*ii`.找到这样一组相邻元素后,再从最尾端开始往前检验,找出第一个小于`*i`的元素,令为`*j`,将i,j元素对调,再将ii之后的所有元素颠倒排列.此即所求之"前一个"排列组合.

举个实例,假设我手上有序列(4,3,2,1,0),图6-10便是套用上述演算法则,一步一步获得的"前一个"排列组合.图中只框出那符合"第一元素为`*i`,第二元素为`*ii`,且满足`*i>*ii`"的相邻两元素,至于寻找适当的j、对调、逆转等操作并未显示出.

![《STL源码剖析》的笔记-prevpermutation.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-prevpermutation.png)

#### 6.7.7 random_shuffle

这个算法将[first,last)的元素次序随机重排.也就是说,在N!种可能的元素排列顺序中随机选出一种,此处N为last-first.

N个元素的序列,其排列方式有N!种,random_shuffle会产生一个均匀分布,因此任何一个排列被选中的机率为1/N!.这很重要,因为有不少算法在其第一阶段过程中必须获得序列的随机重排,但如果其结果未能形成"**在N!个可能排列上均匀分布(uniform distribution)**",便很容易造成算法的错误.

random_shuffle有两个版本，差别在于随机数的取得。版本一使用内部随机数产生器，版本二使用一个会产生随机随机数的仿函数。特别请你注意，该仿函数的传递方式是by reference而非一般的by value，这是因为随机随机数产生器有一个重要特质：它拥有**局部状态（local state）**，每次被调用时都会有所改变，并因此保障产生出来的随机数能够随机。

#### 6.7.8 partial_sort/partial_sort_copy（用到了堆！）

本算法接受一个middle迭代器(位于序列[first,last)之内),然后重新安排[first,last),使序列中的middle-first个最小元素以递增顺序排序,置于(first,middle)内.其余last-middle个元素安置于[middle,last)中,不保证有任何特定顺序

使用sort算法,同样能够保证较小的N个元素以递增顺序置于(first,first+N)之内.选择partial_sort而非sort的唯一理由是效率.是的,如果只是挑出前N个最小元素来排序,当然比对整个序列排序快上许多.

partial_sort有两个版本,其差别在于如何定义某个元素小于另一元素.第一版本使用less-than操作符,第二版本使用仿函数comp.算法内部采用heapsort(4.7.2节)来完成任务,简述于下.

partial_sort的任务是找出middle-first个最小元素,因此,首先界定出区间[first,middle),并利用4.7.2节的make-heap()将它组织成一个max-heap,然后就可以将[middle,last)中的每一个元素拿来与max-heap的最大值比较(max-heap的最大值就在第一个元素身上,轻松可以获得):如果小于该最大值,就互换位置并重新保持max-heap的状态.如此一来,当我们走遍整个[middle,last)时,较大的元素都已经被抽离出(first,middle),这时候再以sort-heap()将[first,midale)做一次排序,即功德圆满.见图6-11的步骤详解.

![《STL源码剖析》的笔记-partialsort.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-partialsort.png)

#### 6.7.9 sort（重难点！！！）

STL所提供的各式各样算法中,sort()是最复杂最庞大的一个.这个算法接受两个Random Access Iterators(随机存取迭代器),然后将区间内的所有元素以渐增方式由小到大重新排列.第二个版本则允许用户指定一个仿函数(functor),作为排序标准,STL的所有关系型容器(associative containers)都拥有自动排序功能(底层结构采用RB-tree,见第5章),所以不需要用到这个sort算法.至于序列式容器(sequence containers)中的stack、queue和priority-queue都有特别的出入口,不允许用户对元素排序.剩下vector,deque和list,前两者的迭代器属于Random Access Iterators,适合使用sort算法,list的迭代器则属于Bidirectioinal Iterators,不在STL标准之列的slist,其迭代器更属于Forwarditerators,都不适合使用sort算法.如果要对list或slist排序,应该使用它们自己提供的member functions sort().稍后我们便可看到为什么泛型算法sort()一定要求RandomAccessterators.

STL的sort算法，数据量大时采用Quick Sort，分段递归排序。一旦分段后的数据量小于某个门槛，为避免Quick Sort的递归调用带来过大的额外负荷（overhead），就改用Insertion Sort，如果递归层次过深，还会改用Heap Sort（已于4.7.2节介绍）。以下分别介绍Quick Sort和Insertion Sort，然后再整合起来介绍STL sort算法。

##### Insertion Sort

Insertion Sort以双层循环的形式进行.外循环遍历整个序列,每次迭代决定出一个子区间:内循环遍历子区间,将子区间内的每一个"逆转对(inversion)"倒转过来。

所谓"逆转对"是指任何两个迭代器i,j, `i<j`而`*i>*j`.一旦不存在"逆转对",序列即排序完毕,这个算法的复杂度为O(N^2),说起来并不理想,但是当数据量很少时,却有不错的效果,原因是实现上有一些技巧(稍后源代码可见),而且不像其它较为复杂的排序算法有着诸如递归调用等操作带来的额外负荷.图6-12是Insertion Sort的详细步骤示意.

![《STL源码剖析》的笔记-insertionsort-modified.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-insertionsort-modified.png)

##### Quick Sort

如果我们拿Insertion Sort来处理大量数据,其O(N^2)的复杂度就令人摇头了.

大数据量的情况下有许多更好的排序算法可供选择.正如其名称所昭示,Quick Sort是目前已知最快的排序法,平均复杂度为O(NlogN),最坏情况下将达O(N^2),不过IntroSort(极类似median-of-three QuickSort的一种排序算法)可将最坏情况推进到O(NlogN).早期的STL sort算法都采用Quick Sort,SGI STL已改用IntroSort.

Quick Sort算法可以叙述如下.假设s代表将被处理的序列:

1. 如果S的元素个数为0或1,结束
2. 取S中的任何一个元素,当作枢轴(pivot) v.
3. 将S分割为L,R两段,使L内的每一个元素都小于或等于v,R内的每一个元素都大于或等于v.
4. 对L,R递归执行Quick Sort.

Quick Sort的精神在于将大区间分割为小区间，分段排序。每一个小区间排序完成后，串接起来的大区间也就完成了排序。最坏的情况发生在分割（partitioning）时产生出一个空的子区间-那完全没有达到分割的预期效果。图6-13说明了Quick Sort的分段排序。

![《STL源码剖析》的笔记-quicksortpartition.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-quicksortpartition.png)
[C++ 中的 typename 及 class 关键字的区别](https://liam.page/2018/03/16/keywords-typename-and-class-in-Cxx/)

##### Median-of-Three（三点中值）

注意，任何一个元素都可以被选来当作枢轴（pivot），但是其合适与否却会影响Quick Sort的效率。为了避免“元素当初输人时不够随机”所带来的恶化效应，最理想最稳当的方式就是取整个序列的头、尾、中央三个位置的元素，以其中值（median）作为枢轴。这种做法称为median-of-three partitioning，或称为mediun-of-three-QuickSort。为了能够快速取出中央位置的元素，显然迭代器必须能够随机定位，亦即必须是个RandomAccessterators。

##### Partitioining（分割）

分割方法不只一种，以下叙述既简单又有良好成效的做法。令头端选代器first向尾部移动，尾端迭代器last向头部移动。当`*first`大于或等于枢轴时就停下来，当`*last`小于或等于枢轴时也停下来，然后检验两个迭代器是否交错。

如果first仍然在左而last仍然在右，就将两者元素互换，然后各自调整一个位置（向中央逼近），再继续进行相同的行为。如果发现两个迭代器交错了（亦即`!(first < last)`），表示整个序列已经调整完毕，以此时的first为轴，将序列分为左右两半，左半部所有元素值都小于或等于枢轴，右半部所有元素值都大于或等于枢轴。

![《STL源码剖析》的笔记-quicksortpartitioneg.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-quicksortpartitioneg.png)

##### threshold（阈值）

面对一个只有十来个元素的小型序列，使用像Quick Sort这样复杂而（可能）需要大量运算的排序法，是否划算？不，不划算，在小数据量的情况下，甚至简单如Insertion Sort者也可能快过Quick Sort-因为Quick Sort会为了极小的子序列而产生许多的函数递归调用。

鉴于这种情况，适度评估序列的大小，然后决定采用Quick Sort或Insertion Sort，是值得采纳的一种优化措施。然而究竟多小的序列才应该断然改用Insertion Sort呢？唔，并无定论，5-20都可能导致差不多的结果，实际的最佳值因设备而异。

##### introsort

不当的枢轴选择，导致不当的分割，导致Quick Sort恶化为O（N^2）。

Introspective Sorting（内省式排序），简称IntroSort，其行为在大部分情况下几乎与median-of-3 Quick Sort完全相同（当然也就一样快）。但是当分割行为（partitioning）有恶化为二次行为的倾向时，能够自我侦测，转而改用Heap Sort，使效率维持在Heap Sort的O（N lgN），又比一开始就使用Heap Sort来得好。

#### 6.7.10 equal_range(应用于有序区间)

算法equal_range是二分查找法的一个版本,试图在已排序的(first,last)中寻找value.它返回一对迭代器i和j,其中i是在不破坏次序的前提下,value可插入的第一个位置(亦即lower-bound),j则是在不破坏次序的前提下,value可插入的最后一个位置(亦即upper-bouna).因此,(i,j)内的每个元素都等同于value,而且[i,j)是[first,last)之中符合此一性质的最大子区间.

如果以稍许不同的角度来思考equal-range,我们可把它想成是[first,last)内"与value等同"之所有元素所形成的区间A.由于[first,last)有序(sorted),所以我们知道"与value等同"之所有元素一定都相邻,于是,算法lower_bound返回区间A的第一个迭代器,算法upper_bound返回区间A的最后元素的下一位置,算法equal_range则是以pair的形式将两者都返回.

即使(first,last)并未含有"与value等同"之任何元素,以上叙述仍然合理,这种情况下"与value等同"之所有元素所形成的,其实是个空区间.

在不破坏次序的前提下,只有一个位置可以插入value,而equal_range所返回的pair,其第一和第二元素(都是迭代器)皆指向该位置.

本算法有两个版本,第一版本采用operator<进行比较,第二版本采用仿函数comp进行比较,稍后只列出版本一的源代码.图6-15展示equal-range的意义.

![《STL源码剖析》的笔记-equalrange.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-equalrange.png)

#### 6.7.11 inplace_merge(应用于有序区间)

如果两个连接在一起的序列(first,middle)和[middle,last)都已排序,那么inplace_merge可将它们结合成单一一个序列,并仍保有序性(sorted).

如果原先两个序列是递增排序,执行结果也会是递增排序,如果原先两个序列是递减排序,执行结果也会是递减排序.

和merge一样,inplace-merge也是一种稳定(stable)操作.每个作为数据来源的子序列中的元素相对次序都不会变动:如果两个子序列有等同的元素,第一序列的元素会被排在第二序列元素之前.

inplace_merge有两个版本,其差别在于如何定义某元素小于另一个元素第一版本使用operators进行比较,第二版本使用仿函数(tunctor)comp进行比较.

这个算法如果有额外的内存（缓冲区）辅助，效率会好许多。但是在没有缓冲区或缓冲区不足的情况下，也可以运作。

#### 6.7.12 nth element

这个算法会重新排列[first,last),使迭代器nth所指的元素,与"整个[first,last)完整排序后,同一位置的元素"同值.此外并保证[nth,last)内没有任何一个元素小于(更精确地说是不大于)[first,nth)内的元素,但对于(first,nth)和[nth,last)两个子区间内的元素次序则无任何保证-这一点也是它与partial-sort很大的不同处.以此观之,nth-element比较近似partition而非sort或partial_sort.

例如,假设有序列(22,30,30,17,33,40,17,23,22,12,201,以下操作:

```c++
nth_element(iv.begin(),iv.begin()+5,iv.end());
```

便是将小于*(iv.begin()+5)(本例为40)的元素置于该元素之左,其余置于该元素之右,并且不保证维持原有的相对位置.获得的结果为(20,12,22,17,17,22,23,30,30,33,40),执行完毕后的5th个位置上的元素值22,与整个序列完整排序后{12,17,17,20,22,22,23,30,30,33,40)的5th个位置上的元素值相同.

如果以上述结果(20,12,22,17,17,22,23,30,30,33,40)为根据,再执行以下操作:

```c++
nth_element(iv.begin(),iv.begin()+5,iv.end(),greatercint>());
```

那便是将大于*(iv.begin()+5)(本例为22)的元素置于该元素之左,其余置于该元素之右,并且不保证维持原有的相对位置.获得的结果为(40,33,30,30,23,22,17,17,22,12,20}.

由于nth_element比partial-sort的保证更少(是的,它不保证两个子序列内的任何次序),所以它当然应该比partial_sort较快.

nth-element有两个版本,其差异在于如何定义某个元素小于另一个元素.第一版本使用operator<进行比较,第二个版本使用仿函数comp进行比较.注意，这个算法只接受RandomAccessiterator.

nth_element的做法是，不断地以median-of-3 partitioning（以首、尾、中央三点中值为枢轴之分割法，见6.7.9节）将整个序列分割为更小的左（L）、右（R）子序列。如果nth迭代器落于左子序列，就再对左子序列进行分割，否则就再对右子序列进行分割，依此类推，直到分割后的子序列长度不大于3（够小了），便对最后这个待分割的子序列做Insertion Sort，大功告成。

![《STL源码剖析》的笔记-nthelement.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-nthelement.png)

#### 6.7.13 merge sort

虽然SGI STL所采用的排序法是IntroSort（一种比Quick Sort考虑更周详的算法，见6.7.9节），不过，另一个很有名的排序算法Merge Sort，很轻易就可以利用STL算法inplace-merge（6.7.11节）实现出来。

Merge Sort的概念是这样的：既然我们知道，将两个有序（sorted）区间归并成一个有序区间，效果不错，那么我们可以利用“分而治之”（devide and conquer）的概念，以各个击破的方式来对一个区间进行排序，首先，将区间对半分割，左右两段各自排序，再利用inplace-merge重新组合为一个完整的有序序列。对半分割的操作可以递归进行，直到每一小段的长度为0或1（那么该小段也就自动完成了排序）。

Merge Sort的复杂度为O（NlogN）。虽然这和Quick Sort是一样的，但因为Merge Sort需借用额外的内存，而且在内存之间移动（复制）数据也会耗费不少时间，所以Merge Sort的效率比不上Quick Sort。实现简单、概念简单，是Merge Sort的两大优点。

## Chapter7 仿函数functors，另名函数对象function objects

### 7.1 仿函数（functors）概观

这一章所探索的东西，在STL历史上有两个不同的名称。仿函数（functors）是早期的命名，C++标准规格定案后所采用的新名称是函数对象（function objects）。

就实现意义而言，“函数对象”比较贴切：一种具有函数特质的对象。不过，就其行为而言，以及就中文用词的清晰漂亮与独特性而言，“仿函数”一词比较突出。因此，本书绝大部分时候采用“仿函数”一词。这种东西在调用者可以像函数一样地被调用（调用），在被调用者则以对象所定义的function call operator扮演函数的实质角色。

仿函数的作用主要在哪里？从第6章可以看出，STL所提供的各种算法，往往有两个版本，其中一个版本表现出最常用（或最直观）的某种运算，第二个版本则表现出最泛化的演算流程，允许用户“以template参数来指定所要采行的策略”。拿accumulate（）来说，其一般行为（第一版本）是将指定范围内的所有元素相加，第二版本则允许你指定某种“操作”，取代第一版本中的“相加”行为。再举sort（）为例，其第一版本是以operator<为排序时的元素位置调整依据，第二版本则允许用户指定任何“操作”，务求排序后的两两相邻元素都能令该操作结果为true。

根据以上陈述，既然函数指针可以达到“将整组操作当做算法的参数”，那又何必有所谓的仿函数呢？原因在于函数指针毕竟不能满足STL对抽象性的要求，也不能满足软件积木的要求-函数指针无法和STL其它组件（如配接器adapter，第8章）搭配，产生更灵活的变化。

就实现观点而言，仿函数其实上就是一个“行为类似函数”的对象。为了能够“行为类似函数”，其类别定义中必须自定义（或说改写、重载）function call运算子（operator（），语法和语意请参考1.9.6节）。拥有这样的运算子后，我们就可以在仿函数的对象后面加上一对小括号，以此调用仿函数所定义的operator（），像这样：

```c++
#include <functional>
#include <iostream>
using namespace std;
int main (){
    greater<int> ig;
    cout << boolalpha << ig (4, 6); // (A) false1程序中的boolalpha是一种所谓的iostream manipulators（操控器），用来控制输出人设备的状态.boolalpha意思是从此以后对boo1值的输出，都改为以字符串“true”或“false”表现。
    cout << greater<int>()(6, 4);   // (B) true
```

![《STL源码剖析》的笔记-functorswithalgorithm.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-functorswithalgorithm.png)

STL仿函数的分类,若以操作数(operand)的个数划分,可分为一元和二元仿函数,若以功能划分,可分为算术运算(Arithmetic)、关系运算(Rational)逻辑运算(Logical)三大类.任何应用程序欲使用STL内建的仿函数,都必须含入头文件,SGI则将它们实际定义于文件中.以下分别描述.

### 7.2 可配接（Adaptable）的关键

在STL六大组件中，仿函数可说是体积最小、观念最简单、实现最容易的一个。但是小兵也能立大功--它扮演一种“策略”2角色，可以让STL算法有更灵活的演出。而更加灵活的关键，在于STL仿函数的可配接性（adaptability）

是的，STL仿函数应该有能力被函数配接器（function adapter，第8章）修饰，彼此像积木一样地串接。为了拥有配接能力，每一个仿函数必须定义自己的相应型别（associative types），就像迭代器如果要融入整个STL大家庭，也必须依照规定定义自己的5个相应型别一样。这些相应型别是为了让配接器能够取出，获得仿函数的某些信息，相应型别都只是一些typedef，所有必要操作在编译期就全部完成了，对程序的执行效率没有任何影响，不带来任何额外负担。

仿函数的相应型别主要用来表现函数参数型别和传回值型别。为了方便起见，`<stl_function.h>`定义了两个classes，分别代表一元仿函数和二元仿函数（STL不支持三元仿函数），其中没有任何data members或member functions，唯有一些型别定义。任何仿函数，只要依个人需求选择继承其中一个class，便自动拥有了那些相应型别，也就自动拥有了配接能力。

#### 7.2.1 unary_function

unary_function用来呈现一元函数的参数型别和回返值型别，其定义非常简单：

```c++
//STL规定,每一个Adaptable Unary Function都应该继承此类别
template <class Arg, class Result>
struct unary_function{
    typedef Arg argument_type;
    typedef Result result_type;
};

```

#### 7.2.2 binary_function

binary_function用来呈现二元函数的第一参数型别、第二参数型别，以及回返值型别。其定义非常简单：

```c++
//STL规定，每一个Adaptable Binary Function都应该继承此类别
template <class Arg1, class Arg2，class Result>
struct binary_function{
    typedef Arg1 first_argument_type;
    typedef Arg2 second_argument_type;
    typedef Result result_type;
};
```

### 7.3 算术类（Arithmetic）

仿函数STL内建的“算术类仿函数”，支持加法、减法、乘法、除法、模数（余数，modulus）和否定（negation）运算。除了“否定”运算为一元运算，其它都是二元运算。

- 加法：`plus<T>`
- 减法：`minus<T>`
- 乘法：`multiplies<T>`
- 除法：`divides<T>`
- 模取（modulus）：`modulus<T>`
- 否定（negation）：`negate<T>`

用法有以下两种，其中`functor<T>()`是一个临时对象（这里的小括号指默认构造函数），后面再接一对小括号即调用function call operator：

```c++
plus<int> plusobj;
cout << plusobj(3, 5);  // 8

cout << plus<int>()(3, 5); // 8
```

稍早我已提过,不会有人在这么单纯的情况下运用这些功能极其简单的仿函数.仿函数的主要用途是为了搭配STL算法.例如,以下式子表示要以1为基本元素,对vector iv中的每一个元素进行乘法(multiplies)运算:

```c++
accumulate(iv.begin(),iv.end(),1,multiplies<int>());
```

#### 证同元素（identity element）

所谓“运算op的证同元素（identity element）”，意思是数值A若与该元素做op运算，会得到A自己。加法的证同元素为0，因为任何元素加上0仍为自己。

乘法的证同元素为1，因为任何元素乘以1仍为自己。

请注意，这些函数并非STL标准规格中的一员，但许多STL实现都有它们。

### 7.4 关系运算类(Relational)

仿函数STL内建的"关系运算类仿函数"支持了等于、不等于、大于、大于等于、小于、小于等于六种运算.每一个都是二元运算.

- 等于(equality):`equal_to<T>`
- 不等于(inequality):`not_equal_to<T>`
- 大于(greater than):`greater<T>`
- 大于或等于(greater than or equal):`greater_equal<T>`
- 小于(less than):`less<T>`
- 小于或等于(less than or equal):`less_equal<T>`

这些仿函数所产生的对象，用法和一般函数完全相同，当然，我们也可以产生一个无名的临时对象来履行函数功能。下面是一个实例，显示两种用法，其中`functor<T>()`是一个临时对象（这里的小括号指默认构造函数），后面再接一对小括号即调用function call operator：

```c++
equal_to<int> equal_to_obj;
cout << equal_to_obj(3, 5); // 0

cout << equal_to<int>()(3, 5); // 0
```

一般而言不会有人在这么单纯的情况下运用这些功能极其简单的仿函数.仿函数的主要用途是为了搭配STL算法.例如以下式子表示要以递增次序对vector iv进行排序:

```c++
sort(iv.begin(),iv.end(),greater<int>());
```

### 7.5 逻辑运算类（Logical）仿函数

STL内建的“逻辑运算类仿函数”支持了逻辑运算中的And。Or。Not三种运算，其中And和or为二元运算，Not为一元运算。

- 逻辑运算And：`logical_and<T>`
- 逻辑运算Or：`logical_or<T>`
- 逻辑运算Not：`logical_not<T>`

这些仿函数所产生的对象，用法和一般函数完全相同。当然，我们也可以产生一个无名的临时对象来履行函数功能。下面是一个实例，显示两种用法：

```c++
logical_and<int> and_obj;
cout << and_obj(true, true); // 1

cout << logical_and<int>()(true, true); // 1
```

一般而言，不会有人在这么单纯的情况下运用这些功能极其简单的仿函数。仿函数的主要用途是为了搭配STL算法。

### 7.6 证同（identity）、选择（select）、投射（project）

这一节介绍的仿函数，都只是将其参数原封不动地传回。其中某些仿函数对传回的参数有刻意的选择，或是刻意的忽略。之所以不在STL或其它泛型程序设计过程中直接使用原本极其简单的identity，project，select等操作，而要再划分一层出来，全是为了间接性，间接性是抽象化的重要工具。

## Chapter8 配接器 adapters

配接器（adapters）在STL组件的灵活组合运用功能上，扮演着轴承、转换器的角色。Adapter这个概念，事实上是一种设计模式（design pattern）。《Design Patterns》一书提到23个最普及的设计模式，其中对adapter样式的定义如下：**将一个class的接口转换为另一个class的接口，使原本因接口不兼容而不能合作的classes，可以一起运作**。

### 8.1 配接器之概观与分类

STL所提供的各种配接器中，改变仿函数（functors）接口者，我们称为function adapter，改变容器（containers）接口者，我们称为container adapter，改变迭代器（iterators）接口者，我们称为iterator adapter。

#### 8.1.1 应用于容器，container adapters

STL提供的两个容器queue和stack，其实都只不过是一种配接器。它们修饰deque的接口而成就出另一种容器风貌。这两个container adapters已于第4章介绍过。

#### 8.1.2 应用于迭代器，iterator adapters

STL提供了许多应用于迭代器身上的配接器，包括insert iterators，reverse iterators，iostream iterators。C++ Standard规定它们的接口可以藉由`<iterator>`获得，SGI STL则将它们实际定义于`<stl_iterator.h>`。

迭代器适配器在C++ Primer中有讲解

##### insert Iterators

所谓insert iterators，可以将一般迭代器的赋值（assign）操作转变为插入（insert）操作。这样的迭代器包括专司尾端插人操作的back_insert_iterator，专司头端插入操作的front_insert_iterator，以及可从任意位置执行插入操作的insert_iterator。由于这三个iterator adapters的使用接口不是十分直观，给一般用户带来困扰，因此，STL更提供三个相应函数：back_inserter()、front inserter()、inserter()，如图8-1所示，提升使用时的便利性。

![《STL源码剖析》的笔记-threeinserterhelper.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-threeinserterhelper.png)

##### Reverse Iterators

所谓reverse iterators，可以将一般迭代器的行进方向逆转，使原本应该前进的operator++变成了后退操作，使原本应该后退的operator--变成了前进操作。

这种错乱的行为不是为了掩人耳目或为了欺敌效果，而是因为这种倒转筋脉的性质运用在“从尾端开始进行”的算法上，有很大的方便性。稍后我有一些范例展示。

##### IOStream Iterators

所谓iostream terators,可以将迭代器绑定到某个iostream对象身上.绑定到istream对象(例如std::cin)身上的,称为istream_iterator,拥有输人功能;绑定到ostream对象(例如std::cout)身上的,称为ostream_iterator,拥有输出功能.这种迭代器运用于屏幕输出,非常方便.以它为蓝图，稍加修改，便可适用于任何输出或输入装置上。例如，你可以在透彻了解iostream_lterators的技术后，完成一个绑定到Internet Explorer cache身上的迭代器，或是完成一个系结到磁盘目录上的一个迭代器

请注意，不像稍后即将出场的仿函数配接器（functor adapters）总以仿函数作为参数，予人以“拿某个配接器来修饰某个仿函数”的直观感受，这里所介绍的迭代器配接器（iterator adapters）很少以迭代器为直接参数3，所谓对迭代器的修饰，只是一种观念上的改变（赋值操作变成插入操作啦、前进变成后退啦、绑定到特殊装置上啦…）。你可以千变万化地写出适合自己所用的任何迭代器。就这一点而言，为了将STL灵活运用于你的日常生活之中，iterator adapters的技术是非常重要的。

#### 8.1.3 应用于仿函数，functor adapters

functor adapters（亦称为function adapters）是所有配接器中数量最庞大的一个族群，其配接灵活度也是前二者所不能及，可以配接、配接、再配接。这些配接操作包括系结（bind）、否定（negate），组合（compose）、以及对一般函数或成员函数的修饰(使其成为一个仿函数).C++ Standard规定这些配接器的接口可由`<functional>`获得,SGI STL则将它们实际定义于`<stl function.h>`.

function adapters的价值在于,通过它们之间的绑定、组合、修饰能力,几乎可以无限制地创造出各种可能的表达式(expression),搭配STL算法一起演出.例如,我们可能希望找出某个序列中所有不小于12的元素个数.虽然,"不小于"就是"大于或等于",我们因此可以选择STL内建的仿函数greater_equal,但如果希望完全遵循题目语意(在某些更复杂的情况下,这可能是必要的),坚持找出"不小于"12的元素个数,可以这么做:

```c++
not1(bind2nd(less(),12))
```

这个式子将less()的第二参数系结(绑定)为12,再加上否定操作,便形成了"不小于12"的语意,整个凑和成为一个表达式(expression),可与任何"可接受表达式为参数"之算法搭配-是的,几乎每个STL算法都有这样的版本.

再举一个例子,假设我们希望对序列中的每一个元素都做某个特殊运算,这个运算的数学表达式为:

```c++
f(g(elem))
```

其中f和g都是数学函数,那么可以这么写:

```c++
composel(f(x),g(y));
```

例如我们希望将容器内的每一个元素v进行`(v+2)*3`的操作,我们可以令`f(x)=x*3,g(y)=y+2`,并写下这样的式子:

```c++
composel(bind2nd(multiplies(),3),bind2nd(plus(),2))
//第一个参数被拿来当做f(),第二个参数被拿来当做g().
```

这一长串形成一个表达式,可以拿来和任何接受表达式的算法搭配.不过,务请注意,这个算式会改变参数的值,所以不能和non-mutating算法搭配.例如不能和for_each搭配,但可以和transform搭配,将结果输往另一地点.

由于仿函数就是"将function call操作符重载"的一种class,而任何算法接受一个仿函数时,总是在其演算过程中调用该仿函数的operator1),这使得不具备仿函数之形、却有真函数之实的"一般函数"和"成员函数(member functions)感到为难.如果这些既存的心血不能纳入复用的体系中,完美的规划就崩落了一角.

为此,STL又提供了为数众多的配接器,使"一般函数"和"成员函数"得以无缝隙地与其它配接器或算法结合起来.当然,STL所提供的这些配接器不可能在变化纷歧的各种应用场合完全满足你的所有需求,例如它没有能够提供我们写出"大于5且小于10"或是"大于8或小于6"这样的算式.不过,对STL源代码有了一番彻底研究后,要打造专用的配接器,不是难事.

请注意,所有期望获得配接能力的组件,本身都必须是可配接的(adaptable).换句话说,一元仿函数必须继承自unary_function(7.11节),二元仿函数必须继承自binary-function(7.1.2节),成员函数必须以mem_fun处理过,一般函数必须以ptr_fun处理过.一个未经ptr_fun处理过的一般函数,虽然也可以函数指针(pointer to function)的形式传给STL算法使用,却无法拥有任何配接能力.

![《STL源码剖析》的笔记-functionadapters.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-functionadapters.png)

### 8.2 container adapters

#### 8.2.1 stack

stack的底层由deque构成.从以下接口可清楚看出stack与deque的关系:

```c++
template <class T, class Sequence = deque<T> >
class stack{
protected:
    Sequence c; //底层容器
    ...
};
```

C++ Standard规定客户端必须能够从中获得stack的接口,SGI STL则把所有的实现细节定义于的内,请参考4.5节.class stack封住了所有的deque对外接口,只开放符合stack原则的几个函数,所以我们说stack是一个配接器,一个作用于容器之上的配接器.

#### 8.2.2 queue

queue的底层由deque构成。从以下接口可清楚看出queue与deque的关系：

```c++
template <class T，class Sequence = deque<T> >
class queue{
protected:
    Sequence c; //底层容器
    ...
};
```

C++ Standard规定客户端必须能够从`<queue>`中获得queue的接口，SGI STL则把所有的实现细节定义于`<stl_queue.h>`内，请参考4.6节.class queue封住了所有的deque对外接口，只开放符合queue原则的几个函数，所以我们说queue是一个配接器，一个作用于容器之上的配接器。

### 8.3 iterator adapters

本章稍早已说过iterator adapters的意义和用法,以下研究其实现细节

#### 8.3.1 insert iterators

下面是三种insert iterators的完整实现列表.其中的主要观念是,每一个insert iterators内部都维护有一个容器(必须由用户指定):容器当然有自己的迭代器,于是,**当客户端对insert iterators做赋值(assign)操作时,就在insert iterators中被转为对该容器的迭代器做插入(insert)操作**,也就是说,在insert iterators的operator=操作符中调用底层容器的push_front()或push_back()或insert()操作函数.

至于其它的迭代器惯常行为如operator++,operator++(int),operator*都被关闭功能,更没有提供operator--(int)或operator--或operator->等功能(因此被类型被定义为output_iterator_tag).换句话说insert iterators的前进、后退、取值、成员取用等操作都是没有意义的,甚至是不允许的.

#### 8.3.2 reverse iterators

所谓reverse terctor，就是将迭代器的移动行为倒转。如果STL算法接受的不是一般正常的迭代器，而是这种逆向迭代器，它就会以从尾到头的方向来处理序列中的元素。例如：

```c++
// 将所有元素逆向拷贝到ite所指位置上
// rbegin()和rend()与reverse_iterator有关
copy(id.rbegin(), id.rend(), ite);
```

为什么"正向迭代器"和"与其相应的逆向迭代器"取出不同的元素呢?这并不是一个潜伏的错误,而是一个刻意为之的特征,主要是为了配合迭代器区间的"前闭后开"习惯(1.9.5节).从图8-3的rbegin()和end()关系可以看出,当迭代器被逆转方向时,虽然其实体位置(真正的地址)不变,但其逻辑位置(送代器所代表的元素)改变了(必须如此改变):

![《STL源码剖析》的笔记-reverseiterator.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-reverseiterator.png)

唯有这样,才能保持正向迭代器的一切惯常行为.换句话说,唯有这样,当找们将一个正向迭代器区间转换为一个逆向迭代器区间后,不必再有任何额外处理,就可以让接受这个逆向迭代器区间的算法,以相反的元素次序来处理区间中的每一个元素,例如(以下出现于8.1.2节实例之中):

```c++
copy(id.begin().id.end(),outite);   // 1 0 1 2 3 4 0 1 2 5 3
copy(id.rbegin(),id.rend(),outite); // 3 5 2 1 0 4 3 2 1 0 1
```

#### 8.3.3 stream iterators

所谓stream iterators,可以将迭代器绑定到一个stream(数据流)对象身上.

绑定到istream对象(例如std::cin)者,称为istream-iterator,拥有输入能力:绑定到ostream对象(例如std::cout)者,称为ostreamiterator拥有输出能力,两者的用法在8.1.2节的例子中都有示范.

乍听之下真神奇.所谓绑定一个istreamobject,其实就是在istream iterator内部维护一个istream member,**客户端对于这个迭代器所做的operator++操作,会被导引调用迭代器内部所含的那个istream member的输入操作(operator>>)**.这个迭代器是个Input terator,不具备operator-.

请注意，源代码清楚告诉我们，只要客户端定义一个istream iterator并绑定到某个istream object，程序便立刻停在`istream iterator<T>::read()`函数，等待输人。这不是我们所预期的行为，因此，请在绝对必要的时刻才定义你所需要的istream iterator，这其实是C++程序员在任何时候都应该遵循的守则，但是很多人忽略了。

![《STL源码剖析》的笔记-copywithistreamiterator.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-copywithistreamiterator.png)

以上便是istream iterator的讨论。至于osteam iterator，所谓绑定一个ostream object，就是在其内部维护一个ostream member，**客户端对于这个迭代器所做的operator=操作，会被导引调用对应的（迭代器内部所含的）那个ostream member的输出操作（operator<<）**。这个迭代器是个Outputterator。下面的源代码和注释说明了一切。

![《STL源码剖析》的笔记-copywithostreamiterator.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-copywithostreamiterator.png)

### 8.4 function adapters

8.1.3节已经对function adapters做了介绍,并举了一个颇为丰富的运用实例.从这里开始,我们就直接探索SGI STL function adapters的实现细节吧.

一般而言,对于C++ template语法有了某种程度的了解之后,我们很能够理解或想象,容器是以class templates完成,算法以function templates完成,仿函数是一种将operator()重载的class template,迭代器则是一种将operator++和operator*等指针习惯常行为重载的class template.然而配接器呢?应用于容器身上和迭代器身上的配接器,已于本章稍早介绍过,都是一种class template.可应用于仿函数身上的配接器呢?如何能够"事先"对一个函数完成参数的绑定、执行结果的否定、甚至多方函数的组合?请注意我用"事先"一词.我的意思是,最后修饰结果(视为一个表达式,expression)将被传给STL算法使用,STL算法才是真正使用这表达式的主格.而我们都知道,只有在真正使用(调用)某个函数(或仿函数)时,才有可能对参数和执行结果做任何干涉

这是怎么回事?

关键在于,就像本章先前所揭示,container adapters内藏了一个container member一样,或是像reverse iterator(adapters)内藏了一个iterator member一样,或是像stream iterator(adapters)内藏了一个pointer to stream一样,或是像insert iterator(adapters)内藏了一个pointer to container(并因而得以取其iterator)一样,每一个function adapters也内藏了一个member object,其型别等同于它所要配接的对象(那个对象当然是一个"可配接的仿函数",adaptable functor),图8-6是一份整理.当function adapter有了完全属于自己的一份修饰对象(的副本)在手,它就成了该修饰对象(的副本)的主人,也就有资格调用该修饰对象(一个仿函数)并在参数和返回值上面动手脚了.

图8-7鸟瞰了count_if()搭配`bind2nd(less<int>(),12))`的例子,其中清楚显示count_if()的控制权是怎么落到我们的手上.控制权一旦在我们手上,我们当然可以予取予求了.

![《STL源码剖析》的笔记-countifwithbind2nd.png](https://raw.githubusercontent.com/edisonleolhl/PicBed/master/%E3%80%8ASTL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E3%80%8B%E7%9A%84%E7%AC%94%E8%AE%B0-countifwithbind2nd.png)

#### 8.4.1 对返回值进行逻辑否定：not1，not2

#### 8.4.2 对参数进行绑定：bind1st，bind2nd

#### 8.4.3 用于函数合成：compose1，compose2

#### 8.4.4 用于函数指针：ptr_fun

这种配接器使我们能够将一般函数当做仿函数使用。一般函数当做仿函数传给STL算法，就语言层面本来就是可以的，就好像原生指针可被当做迭代器传给STL算法样。但如果你不使用这里所说的两个配接器先做一番包装，你所使用的那个-般函数将无配接能力，也就无法和前数小节介绍过的其它配接器接轨。

#### 8.4.5 用于成员函数指针：mem_fun，mem_fun_ref

这种配接器使我们能够将成员函数（member functions）当做仿函数来使用，于是成员函数可以搭配各种泛型算法。当容器的元素型式是x&或x*，而我们又以虚拟（virtual）成员函数作为仿函数，便可以藉由泛型算法完成所谓的多态调用（polymorphic function call）。这是泛型（genericity）与多态（polymorphism）之间的一个重要接轨。

## 附录

台湾常用术语与大陆常用术语转换表

机能 - 功能（函数）
走访/巡访/循序 - 遍历
资料 - 数据
全域 - 全局
弹性 - 灵活性
实作 - 实现
映像 - 映射
杂凑表 - 哈希表
串行 - 链表
指标 - 指针
配接器 - 适配器
提领 - 解引用
决议 - 解析
具现化 - 实例化
物件 - 对象
自变量 - 参数
函式 - 函数
型别 - 类型
巢状 - 内嵌
安插 - 插入

本笔记很多文字是直接通过OCR识别扫描版PDF得到的，所以可能会有一些错误，不过不影响阅读，很多英文括号识别成了中文括号，请读者自行注意，重难点部分的笔记已经经过了本人的勘误，但难免有漏网之鱼，因时间有限，敬请谅解。
