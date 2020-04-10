---
title: OpenMP基础
tags: ["multiprocessing"]
categories: ["programming-language"]
reward: true
copyright: true
date: 2020-04-08 10:04:28
thumbnail: openmp-basic/openmp.png
---



本文介绍 OpenMP C++版本的基本使用方法。

<!--more-->

# 安装

无需安装，gcc自带。只需在编译时启动 `-fopenmp` 选项。

# 第一个OpenMP程序

```cpp
#include <iostream>

int main()
{
    #pragma omp parallel
    {
        std::cout << "Hello world!\n";
    }
    return 0;
}
```

**在使用g++编译时应使用 `-fopenmp` 选项。**

```shell
$ g++ test1.cpp -fopenmp
$ ./a.out
Hello world!
Hello world!
Hello world!
Hello world!
```

程序输出了4个 `Hello world!`， 这是因为 `#pragma omp parallel` 会根据硬件和操作系统配置在运行时生成代码，创建尽可能多的线程，每个线程的起始例程为代码块中位于指令之后的代码。

# 控制线程数量

## 使用编译命令的 `num_threads` 参数控制线程数量

```cpp
#include <iostream>

int main()
{
    #pragma omp parallel num_threads(6)
    {
        std::cout << "Hello World!\n";
    }
    return 0;
}
```

上述程序控制用6个线程执行代码块，因此输出了6个`Hello World!`。

## 使用 `omp_set_num_threads` 控制线程数量

使用 OpenMP 的 API 需要包含头文件 `omp.h` 。

```cpp
#include <omp.h>
#include <iostream>

int main()
{
    omp_set_num_threads(6);
    #pragma omp parallel
    {
        std::cout << "Hello World!\n";
    }
    return 0;
}
```

## 通过环境变量控制线程数量

```cpp
#include <iostream>

int main()
{
    #pragma omp parallel
    {
        std::cout << "Hello world!\n";
    }
    return 0;
}
```

在外部环境中通过环境变量 `OMP_NUM_THREADS` 控制线程数量。仅当没有在程序中显式设置线程数量时，该环境变量才会生效。

```shell
$ export OMP_NUM_THREADS=2
$ ./a.out
Hello world!
Hello world!
```

# 并行化 `for` 循环

```cpp
#include <omp.h>
#include <math.h>
#include <time.h>
#include <iostream>

int main() {
    clock_t clock_timer;
    double wall_timer;
    double c[100000];
    for (int nthreads = 1; nthreads <= 8; nthreads++) {
        clock_timer = clock();
        wall_timer = omp_get_wtime();

        int i;
        #pragma omp parallel for private(i) num_threads(nthreads)
        for (i = 0; i < 100000; i++)
            c[i] = sqrt(i * 4 + i * 2 + i);
        
        std::cout << "threads: " << nthreads << " time on clock(): " << (double)(clock() - clock_timer) / CLOCKS_PER_SEC << " time on wall: " << omp_get_wtime() - wall_timer << std::endl;
    }
    return 0;
}
```

+ `omp_get_wtime` 任选一个点返回已用去的时间，单位为秒。
+ `parallel for private(i)` 意味着循环变量 `i` 将作为一个线程进行处理，每个线程有一个该变量的副本，且未进行初始化。

```shell
threads: 1 time on clock(): 0.002979 time on wall: 0.00297725
threads: 2 time on clock(): 0.000751 time on wall: 0.000606505
threads: 3 time on clock(): 0.000426 time on wall: 0.000428914
threads: 4 time on clock(): 0.006254 time on wall: 0.00260468
threads: 5 time on clock(): 0.002958 time on wall: 0.00133061
threads: 6 time on clock(): 0.001262 time on wall: 0.000695385
threads: 7 time on clock(): 0.001098 time on wall: 0.000518768
threads: 8 time on clock(): 0.001211 time on wall: 0.000538669
```

# 临界区(critical section)

## 编译指示

OpenMP中处理一些互斥的区域时，需要显式指定临界区。

**语法:**

```cpp
#pragma omp critical (optional section name)
{
    // no 2 threads can execute this code blcok concurrently
}
```

代码块中的代码同一时间只会有一个线程会执行。

`optional section name` 是一个全局标识符，即临界区的名字，同一时刻不能同时运行两个相同名字的临界区。

```cpp
#pragma omp critical (section1)
{
    myhashtable.insert("key1", "value1");
}

#pragma omp critical (section2)
{
    myhashtable.insert("key2", "value2");
}
```

## 互斥锁

除了编译指示，OpenMP还提供了互斥锁 `omp_lock_t`。主要为以下5个API：

+ `omp_init_lock`：必须时第一个访问`omp_lock_t`的API，使用它来初始化锁。初始化后锁处于未设置状态。
+ `omp_destroy_lock`：销毁锁，调用时锁必须处于未设置状态。
+ `omp_set_lock`：设置锁，获得互斥。如果一个线程无法设置锁，将会阻塞直到成功设置锁。 
+ `omp_test_lock`：尝试设置锁，不会阻塞，成功返回1，失败返回0。
+ `omp_unset_lock`：释放锁。

例: 构建一个线程安全的队列。

```cpp
#include <omp.h>

template<typename T>
class ThreadSafeQueue : public ThreadUnsafeQueue<T> {
public:
    ThreadSafeQueue(maxsize = 10) : ThreadUnsafeQueue(maxsize) {
        omp_init_lock(&lock);
    }

    ~ThreadSafeQueue() {
        omp_destroy_lock(&lock);
    }

    bool push(const T& data) {
        omp_set_lock(&lock);
        bool result = ThreadUnsafeQueue<T>::push(data);
        omp_unset_lock(&lock);
        return result;
    }

    bool trypush(const T& data) {
        bool result = omp_test_lock(&lock);
        if (result) {
            result = ThreadUnsafeQueue<T>::push(data);
            omp_unset_lock(&lock);
        }
        return result;
    }

    // likewise for pop
    
private:
    omp_lock_t lock;
};
```

## 嵌套锁

OpenMP提供了 `omp_nest_lock_t` 锁。它与 `omp_lock_t` 类似，但有一个优点: 当已经持有锁的线程通过 `omp_set_nest_lock` 重新获得锁时，内部计数器会加一，当多个 `omp_unset_nest_lock` 使计数器置为0时，才会释放锁。

API与`omp_lock_t` 类似:

+ `omp_init_nest_lock`：必须时第一个访问`omp_lock_t`的API，使用它来初始化锁。初始化后锁处于未设置状态，计数器为0。
+ `omp_destroy_nest_lock`：销毁锁，销毁计数器不为0的锁将产生未定义的行为。
+ `omp_set_nest_lock`：设置锁，获得互斥。若线程已持有该锁，则计数器+1，仍然能获得锁，若不持有该锁，将会阻塞直到成功设置锁。 
+ `omp_test_nest_lock`：尝试设置锁，不会阻塞，成功返回1，失败返回0。
+ `omp_unset_nest_lock`：计数器-1，当计数器为0时释放锁。

# 控制任务执行的细粒度

通过 `sections` 可以将没有相关性的程序用 `#pragma omp section` 分割区块，各区块分得一个线程并行运行。

```cpp
#include <omp.h>
#include <iostream>

int main()
{
    #pragma omp parallel
    {
        std::cout << "All threads run this\n";
        
        #pragma omp sections
        {
            #pragma omp section
            {
                std::cout << omp_get_thread_num() << "[A] This executes in parallel\n";
            }

            #pragma omp section
            {
                std::cout << omp_get_thread_num() << "[B] This executes in parallel\n";
                std::cout << omp_get_thread_num() << "[C] This executes after B\n";
            }

            #pragma omp section
            {
                std::cout << omp_get_thread_num() << "[D] This also executes in parallel\n";
            }
        }
    }
    return 0;
}
```

+ 每个section独立运行，互不干扰。
+ OpenMP会使用当前空闲的线程运行 section，如果不加 `#pragma omp parallel`，空闲的线程就一直只有主线程一个，所有的section还是会在单线程上串行运行。

```
All threads run this
0[A] This executes in parallel
0[B] This executes in parallel
All threads run this
1[D] This also executes in parallel
0[C] This executes after B
All threads run this
All threads run this
```

也可以将 `parallel` 和 `sections` 合并:

```cpp
#include <omp.h>
#include <iostream>

int main()
{
    
    #pragma omp parallel sections
    {
        #pragma omp section
        {
            std::cout << omp_get_thread_num() << "[A] This executes in parallel\n";
        }

        #pragma omp section
        {
            std::cout << omp_get_thread_num() << "[B] This executes in parallel\n";
            std::cout << omp_get_thread_num() << "[C] This executes after B\n";
        }

        #pragma omp section
        {
            std::cout << omp_get_thread_num() << "[D] This also executes in parallel\n";
        }
    }
    
    return 0;
}
```

```
0[A] This executes in parallel
0[D] This also executes in parallel
1[B] This executes in parallel
1[C] This executes after B
```






# 参考资料

+ [通过GCC学习OpenMP框架 | IBM](https://www.ibm.com/developerworks/cn/aix/library/au-aix-openmp-framework/index.html)
+ [OpenMP 文档](https://www.openmp.org/resources/refguides/)
+ [OpenMP Reference Sheet for C/C++ | W3Cschool](https://www.w3cschool.cn/openmp/dict)