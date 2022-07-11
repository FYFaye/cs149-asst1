# Assignment 1: 在4核心CPU上分析并行代码

+ 我的实验环境
    + CPU: i5-9600KF，6核，6线程，支持超线程 
    + Dram: 16G DDR4 3200MHz
----
## Program 1: Parallel Fractal Generation Using Threads (25 points)
1. Modify the starter code to parallelize the Mandelbrot generation using two processors. Specifically, compute the top half of the image in thread 0, and the bottom half of the image in thread 1. This type of problem decomposition is referred to as spatial decomposition since different spatial regions of the image are computed by different processors.  
solution: 
```
//
// workerThreadStart --
//
// Thread entrypoint.
void workerThreadStart(WorkerArgs * const args) {

    // TODO FOR CS149 STUDENTS: Implement the body of the worker
    // thread here. Each thread should make a call to mandelbrotSerial()
    // to compute a part of the output image.  For example, in a
    // program that uses two threads, thread 0 could compute the top
    // half of the image and thread 1 could compute the bottom half.
    int startRow = args->threadId * args->height / args->numThreads;
    int totRow = (args->height / args->numThreads);
    // spatial decomposition method
    mandelbrotSerial(args->x0, args->y0, args->x1, args->y1, 
                    args->width, args->height, startRow, totRow, args->maxIterations, args->output);
    printf("Hello world from thread %d\n", args->threadId);
}


```

2. Extend your code to use 2, 3, 4, 5, 6, 7, and 8 threads, partitioning the image
  generation work accordingly (threads should get blocks of the image). Note that the processor only has four cores but each
  core supports two hyper-threads, so it can execute a total of eight threads interleaved on its execution contents.
  In your write-up, produce a graph of __speedup compared to the reference sequential implementation__ as a function of the number of threads used __FOR VIEW 1__. Is speedup linear in the number of threads used? In your writeup hypothesize why this is (or is not) the case? (you may also wish to produce a graph for VIEW 2 to help you come up with a good answer. Hint: take a careful look at the three-thread datapoint.)  
  solution:
  + 使用不同线程数量生成结果图1测试结果

  |   线程数量 | 运行时间(ms)   |
  |-----------|--------------|
  | 2 | 225.396 |
  | 3 | 267.775 |
  | 4 | 183.318 |
  | 5 | 183.485 | 
  | 6 | 146.168 | 
  | 7 | 140.846 |
  | 8 | 119.016 |

  + 使用不同线程数量生成结果图2的测试结果


  |   线程数量 | 运行时间(ms)   |
  |-----------|--------------|
  | 2 | 163.854 |
  | 3 | 130.955 |
  | 4 | 111.145 |
  | 5 | 96.751 | 
  | 6 | 86.242 | 
  | 7 | 78.977 |
  | 8 | 70.701 |
  
  在生成图片1的时候，线程数量为3的时候，反而运行时间在上升，而生成图片2的时候，运行时间随着线程数量一直递减。在这个情况下就可以考虑是生成数据的迭代次数不同的问题。其实就是图片1中间的迭代次数多了很多。可以从图片生成的代码中看出来。
```
  
static inline int mandel(float c_re, float c_im, int count)
{
    float z_re = c_re, z_im = c_im;
    int i;
    for (i = 0; i < count; ++i) {

        if (z_re * z_re + z_im * z_im > 4.f)
            break;

        float new_re = z_re*z_re - z_im*z_im;
        float new_im = 2.f * z_re * z_im;
        z_re = c_re + new_re;
        z_im = c_im + new_im;
    }

    return i;
}

```

3. To confirm (or disprove) your hypothesis, measure the amount of time
  each thread requires to complete its work by inserting timing code at
  the beginning and end of `workerThreadStart()`. How do your measurements
  explain the speedup graph you previously created?

  solution: 与main.cpp中计算时间的方式一致

```

//
// workerThreadStart --
//
// Thread entrypoint.
void workerThreadStart(WorkerArgs * const args) {

    // TODO FOR CS149 STUDENTS: Implement the body of the worker
    // thread here. Each thread should make a call to mandelbrotSerial()
    // to compute a part of the output image.  For example, in a
    // program that uses two threads, thread 0 could compute the top
    // half of the image and thread 1 could compute the bottom half.
    double minThread = 1e30;
    double startTime = CycleTimer::currentSeconds();
    int startRow = args->threadId * args->height / args->numThreads;
    int totRow = (args->height / args->numThreads);
    // spatial decomposition method
    mandelbrotSerial(args->x0, args->y0, args->x1, args->y1, 
                    args->width, args->height, startRow, totRow, args->maxIterations, args->output);
    double endTime = CycleTimer::currentSeconds();
    minThread = std::min(minThread, endTime - startTime);
    //printf("Hello world from thread %d\n", args->threadId);
    printf("Thread\t[%d]:\t\t[%.3f] ms\n", args->threadId, minThread * 1000);
}

```
and according the test time, view 2 has more average time consumption in each thread:

+ view 1 测试结果
```
[mandelbrot serial]:            [447.189] ms
Wrote image file mandelbrot-serial.ppm
Thread  [0]:            [9.229] ms
Thread  [7]:            [8.497] ms
Thread  [1]:            [41.974] ms
Thread  [6]:            [42.011] ms
Thread  [2]:            [79.779] ms
Thread  [5]:            [83.333] ms
Thread  [4]:            [121.137] ms
Thread  [3]:            [123.417] ms
Thread  [7]:            [8.023] ms
Thread  [0]:            [9.896] ms
Thread  [1]:            [40.010] ms
Thread  [6]:            [44.991] ms
Thread  [2]:            [80.231] ms
Thread  [5]:            [82.177] ms
Thread  [3]:            [121.817] ms
Thread  [4]:            [122.146] ms
Thread  [0]:            [8.893] ms
Thread  [7]:            [9.203] ms
Thread  [1]:            [40.151] ms
Thread  [6]:            [49.022] ms
Thread  [2]:            [81.162] ms
Thread  [5]:            [83.116] ms
Thread  [4]:            [122.332] ms

```

+ view 2 test result

```
[mandelbrot serial]:            [271.994] ms
Wrote image file mandelbrot-serial.ppm
Thread  [7]:            [29.623] ms
Thread  [5]:            [33.251] ms
Thread  [2]:            [36.682] ms
Thread  [6]:            [37.175] ms
Thread  [3]:            [39.546] ms
Thread  [4]:            [39.783] ms
Thread  [1]:            [50.034] ms
Thread  [0]:            [70.437] ms
Thread  [7]:            [30.079] ms
Thread  [5]:            [36.188] ms
Thread  [6]:            [38.043] ms
Thread  [3]:            [38.640] ms
Thread  [4]:            [40.626] ms
Thread  [2]:            [41.195] ms
Thread  [1]:            [50.983] ms
Thread  [0]:            [80.598] ms
Thread  [7]:            [30.022] ms
Thread  [2]:            [32.771] ms
Thread  [5]:            [35.171] ms
Thread  [6]:            [36.374] ms
```
from question 3's analysis, the key to accelrate is to alloc more average, this goals is quite ez, just let one thread sample rows by fixed intervals.


5. Now run your improved code with 16 threads. Is performance noticably greater than when running with eight threads? Why or why not?  
ans: no , because, CPU only support 8 threads at one times, more threads **do not** mean more hardware useage.
## Program 2: 使用SIMD指令集的向量化代码
1. 



2. 测试结果如下，可以看到，向量运算宽度

 |   向量运算宽度 | 向量运算最大化利用率（%）   |
  |-----------|--------------|
  | 2 |  73.7% |
  | 4 | 67.5% |
  | 8 |  64.2% |
  | 16 | 62.7% |

3. 经典规约



## Program 3: 使用ISPC并行生成图片


### part 1
1. 期望是8倍的加速，实际上由于计算任务量不均衡，导致了性能降低，view 1只有4.83倍的加速，view 2只有4.14倍的加速。同样是由于view 2的白色部分更多，计算量更大导致了性能的降低。


### part 2
1. 在使用task函数后，对于View 1有了9.42倍的加速，大约是不分开计算的两倍。
2. 就按照CPU的超线程数量分配Task就可以达到最佳计算性能，不过我最佳都没达到30x，只有27.43x，可能这就是i5吧

3. SIMD与多线程有什么区别，除了xxx



## Program 4: 迭代求平方根
1. 很明显，`(4.03x speedup from ISPC)`SIMD只提供了4.03x的加速，在多核分配的task中，` (21.68x speedup from task ISPC)`又加速到21.68x，那么多核只在SIMD中提供了5.5x左右的加速。


2. 这个特殊的输入就是全都是同一个数字；测试输出结果如下：
```

    [sqrt serial]:          [2115.311] ms
    [sqrt ispc]:            [331.048] ms
    [sqrt task ispc]:       [63.220] ms
                                    (6.39x speedup from ISPC)
                                    (33.46x speedup from task ISPC)
```
可以看到，SIMD所提供的加速确实提高了，因为同一时刻，一个核心的所有线程均在运行同一条指令，没有多余的分支造成计算单元的浪费；同时多线程并没有在SIMD的基础上更近一步的加速，因为不同thread之间本身就没有任何关系，所以多线程技术提供的加速是相对不变的。

3. 最差与最好的情况分析
    + 最好的情况  
    最好的情况应该是数据均一致，并且有足够的计算量，所以我选择的是2.999f， 因为该数值的计算量最大，最好的情况是6.41x的加速。
    + 最差的情况  
    最差的情况是数据在一个计算指令中有分支，并且只有1个数据有多余的分支，因为是8个单元的SIMD，每八个数据中，一个是最大的计算量数据2.999f，其余均为最小计算量数据1.000f,最差的结果是0.84x的“加速”。

4. tobe continue


## Program 5: BLAS库中的`saxpy`函数

1. 直接跑，多线程只有1.19x的加速，根本没有起到多线程的应该有的12x左右的加速。原因就是内存有效带宽上不去。所以是不能通过多线程技术进一步加速。

2. 在写的时候，cpu首先需要从内存更新cache line的内容，然后更改，再写入内存，所以一次写代表两次IO操作。