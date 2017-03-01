
#### 第一章：What Is Garbage Collection?

[原文的地址](https://plumbr.eu/handbook/what-is-garbage-collection)    

顾名思义，垃圾回收，就是找到“垃圾”，并且处理了。事实上垃圾回收做的是跟踪所有在使用的对象，把那些不在使用的对象设置为“垃圾”。下面，我们将逐步的了解JVM中的垃圾回收器是怎么做到内存自动回收的。    

追本溯源，我们首先看一下垃圾回收的起源，概念和方法。    

##### 人工内存管理

以前没有垃圾回收器的时候，我们使用数据的时候，是通过手动的分配和释放的。如果，忘记了释放申请的内存，这块内存就被占用着，不能被再次的利用。这种情况就被称之为：内存泄漏。   

下面就是一个C语言的内存管理的例子：     
```
int send_request() {
    size_t n = read_size();
    int *elements = malloc(n * sizeof(int));

    if(read_elements(n, elements) < n) {
        // elements not freed!
        return -1;
    }

    // …

    free(elements)
    return 0;
}
```    
如果忘记了free(elements),就会造成内存的泄漏。内存泄漏一旦发生排查和修改还是比较的困难的。所以一种比较好的办法就是内存管理自动化，自动的释放已经不再使用的内存，减少人工编程过程中可能出现的错误。这种机制就称之为垃圾回收期，简称GC。    

##### Smart Pointers    

垃圾回收期的方式的其中一种，是建立在引用计数（reference counting）。对于一个对应，如果它被引用的次数是0，那么这个对应就可以被回收了。一个比较出名的例子就是C++的共享指针。   
~~~
int send_request() {
    size_t n = read_size();
    shared_ptr<vector<int>> elements
              = make_shared<vector<int>>();
    if(read_elements(n, elements) < n) {
        return -1;
}
return 0; }
~~~    

shared_ptr,如果我们在别处使用的时候，对应的引用次数就会增加，当执行返回退出作用范围的时候，对应的引用次数就会减少。当然，在实际实现的过程中，不可能这么简单，此例只是为了说明问题。

##### 内存管理自动化     

第一个实现垃圾收集器是1959年的Lisp语言，自从那以后，垃圾回收期才开始发展起来。      

###### 引用计数

引用计数：就类似上面所说的，计算C++中每一个指向对象的指针的引用次数一样的原理。像Perl，Python，PHP使用的这种方法，这种理念可以使用下图来说明：    
![引用计数](http://7xtrwx.com1.z0.glb.clouddn.com/d9283efe658addf96c07e2de2a1c8476.png)

当然这种思想，也有一些和上面图中不太一样的情形，例如：   
![循环引用的情况](http://7xtrwx.com1.z0.glb.clouddn.com/ae8fe2589a89ef478f75f6ec202e4eba.png)

从图中，可以看到，红色的部分已经变成了不可达的了。即使没有程序能够通过引用访问到这些对象，但是因为存在循环引用的问题，红色部分中的每一个对应的引用计数值都大于0.当然是用这种思想实现垃圾回收器的Perl，Python，PHP，都有自己的方案来处理这种情况，但是具体是怎么处理的已经不再我们的讨论的范围之内了。
