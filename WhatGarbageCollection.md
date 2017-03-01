
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
如果忘记了free(elements),就会造成内存的泄漏。内存泄漏一旦发生排查和修改还是比较的困难的。
