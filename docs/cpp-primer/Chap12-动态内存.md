- 对象都有其生命期。[p399]
    - 全局对象在程序启动时分配，在程序结束时销毁。
    - 局部自动对象，在进入其定义所在的程序块时被创建，在离开块时销毁。
    - 局部static对象在第一次使用前分配，在程序结束时销毁。
    - 动态分配的对象的生命期与其在哪里创建是无关的，只有当显示地被释放时，这些对象才会销毁。



- ==静态内存== 用来保存局部static对象、类static数据成员以及定义在任何函数之外的变量。==栈内存== 用来保存定义在函数内的非static对象。分配在静态或栈内存中的对象由编译器自动创建和销毁。除了静态内存和栈内存，每个程序还有一个内存池，这部分内存被称为 ==自由空间或堆(heap)==，程序用堆来存储动态分配的对象，即那些在程序运行时分配的对象，动态对象的生命期由程序控制。p[400]



- 动态内存的管理通过一对运算符来完成：[p400]

    **new**：在动态内存中为对象分配空间并返回一个指向该对象的指针，可以选择对对象进行初始化；

    **delete**：接受一个动态对象的指针，销毁该对象，并释放与之关联的内存。



- 为了更容易也更安全的使用动态内存，新标准库提供了两种智能指针(smart pointer)类型来管理动态对象。智能指针的行为类似常规指针，重要的区别是它负责自动释放所指向的对象。这两种智能指针的区别在于管理底层指针的方式：

    - `shared_ptr`允许多个指针指向同一个对象
    - `unique_ptr`则独占所指向的对象

    标准库还定义了一个名叫`weak_ptr`的伴随类，它是一种弱引用，指向`shared_ptr`所管理的对象。这三种类型都定义在`memory`头文件中。[p400]



- 智能指针的常用操作 [p401]

    - `shared_ptr`和`unique_ptr`都支持的操作
        ```cpp
        shared_ptr<T> sp  //空智能指针，可以指向类型为T的对象
        unique_ptr<T> up
        p      //将p用作一个条件判断，若p指向一个对象，则为true
        *p     //解引用p，获得它指向的对象
        p->mem // 等价于(*p).mem
        p.get() //返回p中保存的指针，要小心使用，若智能指针释放了其对象，返回的指针所指向的对象也就消失了
        swap(p, q) //交换p和q中的指针
        p.swap(q)
        ```

    - `shared_ptr`独有的操作

        ```cpp
        make_shared<T>(args) //返回一个shared_ptr，指向一个动态分配的类型为T的对象，使用args初始化此对象
        shared_ptr<T>p(q)  //p是shared_ptr q的拷贝；此操作会递增q中的计数器。q中的指针必须能转换为T*
        p = q //p和q都是shared_ptr，所保存的指针必须能相互赞欢呼。此操作会递减p的引用计数，递增q的引用计数；所p的引用计数变为0，则将其管理的原内存释放
        p.unique()//若p.use_count()为1，返回true，否则返回false
        p.use_count()//返回与p共享对象的智能指针数量；可能很慢，主要用于调试
        ```



- 最安全的分配和使用动态内存的方法是调用一个名为`make_shared`的标准库函数。此函数在动态内存中分配一个对象并初始化它，返回指向此对象的`shared_ptr`。[p401]



- 程序使用动态内存的三种原因：[p403]

    1. 程序不知道自己需要使用多少对象
    2. 程序不知道所需对象的准确类型
    3. 程序需要在多个对象间共享数据



- 使用`new`和`delete`管理动态内存存在的三个常见问题：[p410]

    - ==忘记delete内存==。忘记释放动态内存会导致"内存泄漏"问题，因为这种内存永远不可能被归还给自由空间了。
    - ==使用已经释放掉的对象==。通过将释放内存后将指针置为空，有时可以检测出这种错误。
    - ==同一块内存释放两次==。当有两个指针指向相同的动态分配对象时，可能发生这种错误。

     **最佳实践**：坚持只使用智能指针，就可以避免所有这些问题。


- 当delete一个指针后，指针值就变为无效了。虽然指针已经无效，但在很多机器上指针仍然保留着(已经释放了的)动态内存的地址。在delete之后，指针就变为了空悬指针(dangling pointer)，即，指向一块曾经保存数据对象但现在已经无效的内存的指针。[p411]



- 动态内存的一个基本问题是可能有多个指针指向相同的内存。在`delete`内存之后重置指针为`nullptr`只对这个指针有效，对其他任何仍指向(已释放的)内存的指针是没有作用的。例如：[p411]
    ```cpp
    int *p(new int(42)); // p指向动态内存
    auto q = p;   // p和q指向相同的内存
    delete p;     // p和q均变为无效
    p = nullptr;  //指出p不再绑定到任何对象(对q没有任何作用！！！)
    ```



- 定义和改变`shared_ptr`的其他方法 [p412]
    ```cpp
    shared_ptr<T> p(q)  //p管理内置指针q所指向的对象；q必须指向new分配的内存，且能够转换为T*类型
    shared_ptr<T> p(u)  //p从unique_ptr u那里接管了对象的所有权；将u置为空
    shared_ptr<T> p(q,d) //p接管了内置指针q所指向的对象的所有权。q必须能转换为T*类型。p将使用可调用对象d来代替delete
    shared_ptr<T> p(p2,d) //p是shared_ptr p2的拷贝，p将用可调用对象d来代替delete

    p.reset()    //若p是唯一指向其对象的shared_ptr，reset会释放此对象。
    p.reset(q)   //若传递了可选的参数内置指针q，会令p指向q，否则会将p置为空。
    p.reset(q,d) //若还传递了参数d，将会调用d而不是delete来释放q
    ```


- 不要混合使用普通指针和智能指针。[p413]


- 不要使用`get`初始化另一个智能指针或为智能指针赋值。`get`返回一个内置指针，指向智能指针管理的对象，用来将指针的访问权限传递给(不能使用智能指针的)代码，只有在确定代码不会delete指针的情况下才能使用`get`。[p414]


- 使用智能指针可以确保在程序发生异常后资源被正确地释放。[p415]
    ```cpp
    void f()
    {
      shared_ptr<int> sp(new int(42));  //分配一个新对象
      //这段代码抛出一个异常，且在f中未被捕获
    } //在函数结束时shared_ptr自动释放内存
    ```
    ```cpp
    void f()
    {
      int *ip = new int(42); //动态分配一个新对象
      //这段代码抛出一个异常，且在f中未被捕获
      delete ip;  //在退出之前释放内存
    } //内存不会被释放
    ```



- 使用智能指针的一些基本规范 [p417]

    - 不使用相同的内置指针值初始化(或reset)多个智能指针

    - 不`delete` `get()`返回的指针

    - 不使用`get()`初始化或`eset另一个智能指针

    - 如果使用`get()`返回的指针，记住当最后一个对应的智能指针销毁后，指针就无效了

    - 如果使用智能指针管理的资源不是`new`分配的内存，记住传递给它一个删除器


unique_ptr
-----

- 某个时刻只能有一个`unique_ptr`指向一个给定的对象，当`unique_ptr`被销毁，其所指向的对象也被销毁。[p417]

    其所支持的操作：

    ```cpp
    unique_ptr<T> u1  //空unique_ptr，可以指向类型为T的对象。u1会使用delete来释放  它的指针；
    unique_ptr<T,D> u2  //u2会使用一个类型为D的可调用对象来释放它的指针
    unique_ptr<T,D> u(d)  //空unique_ptr，指向类型为T的对象，用类型为D的对象d来代  替delete
    u = nullptr  //释放u指向的对象，将u置为空
    u.release()  //u放弃对指针的控制权，返回指针，并将u置为空
    u.reset()    //释放u指向的对象
    u.reset(q)   //如果提供了内置指针q，令u指向这个对象；否则将u置为空
    u.reset(nullptr) //
    ```


- 当定义一个`unique_ptr`时，需要将其绑定到一个`new`返回的指针上 [p417]
    ```cpp
    unique_ptr<double> p1; //p1是可以指向一个double的unique_ptr
    unique_ptr<int> p2(new int(42)); //p2指向一个值为42的int
    ```

- 由于`unique_ptr`拥有它所指向的对象，因此`unique_ptr`不支持普通的拷贝或赋值操作：[p417]
    ```cpp
    unique_ptr<string> p1(new string("hello"));
    unique_ptr<string> p2(p1); //错误：unique_ptr不支持拷贝
    unique_ptr<string> p3;
    p3 = p2; //错误：unique_ptr不支持赋值
    ```

- 通过调用`release`或`reset`可以将指针的所有权从一个(非`const`)`unique_ptr`传递给另一个`unique_ptr`：p[418]

    ```cpp
    //将所有权从p1(指向string hello)转移给p2
    unique_ptr<string> p2(p1.release());  //release将p1置为空
    unique_ptr<string> p3(new string("world"));
    //将所有权从p3转移给p2
    p2.reset(p3.release()); //reset释放了p2原来指向的内存
    ```



- `release`返回的指针通常被用来初始化另一个智能指针或给另一个智能指针赋值。但如果不用另一个智能指针来保存`release`返回的指针，程序就要负责资源的释放：p[418]

    ```cpp
    p2.release(); //错误：p2不会释放内存，而且丢失了指针
    auto p = p2.release(); //正确，但必须记得delete(p)
    ```



- 不能拷贝`unique_ptr`的规则有一个例外：可以拷贝或赋值一个 **将要被销毁** 的`unique_ptr`，最常见的例子是从函数返回一个`unique_ptr`：

    ```cpp
    unique_ptr<int> clone(int p) {
      //正确：从int*创建一个unique_ptr<int>
      return unique_ptr<int>(new int(p));
    }
    ```

    还可以返回一个局部对象的拷贝：

    ```cpp
    unique_ptr<int> clone(int p) {
      unique_ptr<int> ret(new int(p));
      // ...
      return ret;
    }
    ```

    对于上面两段代码，编译器知道要返回的对象将要被销毁。在此情况下，编译器执行一种特殊的"  拷贝"。p[418]



- 可以重载一个`unique_ptr`中默认的删除器，但`unique_ptr`管理删除器的方式与`shared_ptr`不同，必须在尖括号中`unique_ptr`指向类型之后提供删除器类型，在创建或reset一个这种`unique_ptr`类型的对象时，必须提供一个指定类型的可调用对象(删除器)：p[419]

    ```cpp
    //p指向一个类型为objT的对象，并使用一个类型为delT的对象释放objT对象
    //它会调用一个名为fcn的delT类型对象
    unique_ptr<objT, delT> p(new objT, fcn);
    ```



weak_ptr
-----

- `weak_ptr`是一种不控制所指向对象生存期的智能指针，它指向由一个`shared_ptr`管理的对象。将一个`weak_ptr`绑定到一个`shared_ptr` ==不会改变`shared_ptr`的引用计数==。一旦最后一个指向对象的`shared_ptr`被销毁，对象就会被释放，即使有`weak_ptr`指向对象，对象也还是会被释放。p[420]

    ```cpp
    weak_ptr<T> w  //空weak_ptr可以指向类型为T的对象
    weak_ptr<T> w(sp) //与shared_ptr sp指向相同对象的weak_ptr。T必须能转换为sp指向的类型
    w = p  //p可以是一个shared_ptr或一个weak_ptr。赋值后w与p共享对象
    w.reset()  //将w置为空
    w.use_count() //与w共享对象的shared_ptr的数量
    w.expired()   //若w.use_count()为0，返回true，否则返回false
    w.lock() //如果expired为true，返回一个空shared_ptr，否则返回一个指向w的对象的shared_ptr
    ```



- 当创建一个`weak_ptr`时，要用一个`shared_ptr`来初始化它：p[420]

    ```cpp
    auto p = make_shared<int>(42);
    weak_ptr<int> wp(p); //wp弱共享p；p的引用计数未改变
    ```


- 由于对象可能不存在，不能使用`weak_ptr`直接访问对象，而 ==必须调用lock==。此函数检查`weak_ptr`指向的对象是否仍存在。如果存在，lock返回一个指向共享对象的`shared_ptr`。例如：p[420]

    ```cpp
    if (shared_ptr<int> np = wp.lock()) { // 如果np不为空则条件成立
      //在if中，np和p共享对象
    }
    ```



动态数组
-----

- C++语言和标准库提供了两种一次分配一个对象数组的方法。
    - C++语言定义了另一种new表达式语法，可以分配并初始化一个对象数组。

    - 标准库中包含一个名为allocator的类，允许将分配和初始化分离。

      使用allocator通常会提供更好的性能和更灵活的内存管理能力。p[423]


- new分配要求数量的对象并(假定分配成功后)返回指向第一个对象的指针：p[423]

    ```cpp
    int *pia = new int[get_size()];  // pia指向第一个int
    typedef int arrT[42]; //arrT表示42个int的整型数组
    int *p = new arrT; //分配一个42个int的数组；p指向第一个int
    ```

- 默认情况下，new分配的对象，不管是单个分配的还是数组中的，都是默认初始化的。可以对数组中的元素进行值初始化。p[424]

    ```cpp
    int *pia = new int[10];         //10个未初始化的int
    int *pia2 = new int[10]();      //10个值初始化为0的int
    string psa = new string[10];    //10个空string
    string psa2 = new string[10](); //10个空string

    int *pia3 = new int[10]{0,1,2,3,4,5,6,7,8,9}; //10个int分别用列表中对应的初始化器初始化
    string *psa3 = new string[10]{"a", "an", "the", string(3,'x')}; //10个string，前4个用给定的初始化器初始化，剩余的进行值初始化
    ```



- 动态分配一个空数组是合法的。p[424]

    ```cpp
    char arr[0];  //错误：不能定义长度为0的数组
    char *cp = new char[0]; //正确：但cp不能解引用
    ```

    当用new分配一个大小为0的数组时，new返回一个合法的非空指针，此指针保证与new返回的其他任何指针都不相同。对于零长度的数组来说，此指针就像尾后指针一样。



- 动态数组通过`delete []`销毁数组中的元素，并释放对应的内存。数组中的元素按逆序销毁。p[425]

    ```cpp
    delete p;     //p必须指向一个动态分配的对象或为空
    delete [] pa; //pa必须指向一个动态分配的数组或为空

    typedef int arrT[42]; //arrT是42个int的数组的类型别名
    int *pi = new arrT;   //分配一个42个int的数组；pi指向第一个元素
    delete [] pi;         //方括号是必须的，因为当初分配的是一个数组
    ```



- 标准库提供了一个可以管理`new`分配的数组的`unique_ptr`版本。p[425]

    ```cpp
    // up指向一个包含10个未初始化int的数组
    unique_ptr<int[]> up(new int[10]);
    for (size_t i = 0; i != 10; ++i)
      up[i] = i;   //为每个元素赋予一个新值
    up.release();  //自动调用delete[]销毁其指针
    ```

    指向数组的`unique_ptr`提供的操作

    ```cpp
    //指向数组的unique_ptr不支持成员访问运算符(点和箭头运算符)。其他unique_ptr操作不变
    unique_ptr<T[]> u  //u可以指向一个动态分配的数组，数组元素类型为T
    unique_ptr<T[]> u(p)  //u指向内置指针p所指向的动态分配的数组。p必须能转换为类型T*
    u[i] //返回u拥有的数组中位置i处的对象，u必须指向一个数组
    ```



- `shared_ptr` ==不直接支持管理动态数组==。如果希望使用shared_ptr管理一个动态数组，必须提供自己定义的删除器：p[426]

    ```cpp
    //为了使用shared_ptr，必须提供一个删除器
    shared_ptr<int> sp(new int[10], [](int *p) {delete[] p; });
    sp.reset(); //使用提供的lambda释放数组，它使用delete[]
    ```

    shared_ptr未定义下标运算符，而且智能指针类型不支持指针算术运算。因此为了访问数组中的元素，必须用get获取一个内置指针，然后用它来访问数组元素。

    ```cpp
    //shared_ptr未定义下标运算符，并且不支持指针的算术运算
    for (size_t i = 0; i != 10; ++i)
      *(sp.get() + i) = i; //使用get获取一个内置指针
    ```

allocator
-----

- `new`有一些灵活性上的局限，其中一方面表现在它 **将内存分配和对象构造组合在了一起**。类似的，delete将 **对象析构和内存释放** 组合在了一起。p[427]

- 标准库`allocator`类定义在头文件`memory`中，它 ==将内存分配和对象构造分离开来== 它提供一种类型感知的内存分配方法，它分配的内存是原始的、未构造的。p[427]

    其支持的操作有：

    ```cpp
    allocator<T> a //定义了一个名为a的allocator类，它可以为类型为T的对象分配内存
    a.allocate(n) //分配一段原始的、未构造的内存，保存n个类型为T的对象
    a.deallocate(p,n) //释放从T*指针p中地址开始的内存，这块内存保存了n个类型为T的对象；p必须是一个先前由allocate返回的指针，且n必须是p创建时所要求的大小。在调用deallocate之前，用户必须对每个在这块内存中创建的对象调用destroy
    a.construct(p, args) //p必须是一个类型为T*的指针，指向一块原始内存；arg被传递给类型为T的构造函数，用来在p所指向的内存中构造一个对象
    a.destroy(p) //p为T*类型的指针，此算法对p所指向的对象执行析构函数
    ```



- 当一个`allocator`对象分配内存时，它会根据给定的对象类型来确定恰当的内存大小和对齐位置：p[427]

    ```cpp
    allocator<string> alloc; //可以分配string的allocator对象
    auto const p = alloc.allocate(n); //分配n个未初始化的string
    ```



- `allocator`分配的内存是未构造的(unconstructed)。可以按需要在此内存中构造对象。在新标准库中，`construct`成员函数接受一个指针和零个或多个额外参数，在给定位置构造一个元素。额外参数用来初始化构造的对象，必须是与构造的对象的类型相匹配的合法的初始化器：

    ```cpp
    auto q = p; //q指向最后构造的元素之后的位置
    alloc.construct(q++);           //*q为空字符串
    alloc.construct(q++, 10, 'c');  //*q为cccccccccc
    alloc.construct(q++, "hi");     //*q为hi
    ```

    注意使用未定义的内存，其行为是未定义的。

    ```cpp
    cout << *p << endl; //正确：使用string的输出运算符
    cout << *q << endl; //错误：q指向未构造的内存！
    ```

    当使用完对象后，必须对每个构造的元素调用`destroy`来销毁它们，函数`destroy`接受一个指针，对指向的对象执行析构函数：

    ```cpp
    while (q != p)
      alloc.destroy(--q);  //释放真正构造的string
    ```

    注意只能对真正构造了的元素进行`destroy`操作。

    一旦元素被销毁，就可以重新使用这部分内存，也可以将其归还给系统。释放内存通过调用`deallocate`来完成：

    ```cpp
    alloc.deallocate(p,n);
    ```

    注意传递给`dealloate`的指针`p`不能为空，它必须指向由`allocate`分配的内存。而且，大小参数`n`必须与调用`allocate`分配内存时提供的大小参数一样。[p428]



- 标准库还为allocator类定义了两个伴随算法，可以 **在未初始化内存中创建对象**。[p429]

    ```cpp
    //这些函数在给定目的的位置创建元素，而不是由系统分配内存给它们
    uninitialized_copy(b,e,b2) //从迭代器b和e指出的输入范围中拷贝元素到迭代器b2指定的未构造的原始内存中。b2指向的内存必须足够大，能容纳输入序列中元素的拷贝
    uninitialized_copy_n(b,n,b2) //从迭代器b指向的元素开始，拷贝n个元素到b2开始的内存中

    uninitialized_fill(b,e,t) //在迭代器b和e指定的原始内存范围中创建对象，对象的值均为t的拷贝
    uninitialized_fill_n(b,n,t) //从迭代器b指向的内存地址开始创建n个对象。b必须指向足够大的未构造的原始内存，能够容纳给定数量的对象
    ```

    例如：

    ```cpp
    //分配比vi中元素所占用空间大一倍的动态内存
    auto p = alloc.allocate(vi.size()*2);
    //通过拷贝vi中的元素来构造从p开始的元素
    auto q = uninitialized_copy(vi.begin(), vi.end(), p); //返回指向最后一个构造的元素之后的位置的指针
    //将剩余元素初始化为42
    uninitialized_fill_n(q, vi.size(), 42); //
    ```
