---
layout: post
category : 读书笔记
title : 我的第一篇markdown文章
tagline: "Supporting tagline"
tags : [读书笔记]
---
{% include JB/setup %}

# 伪共享（False Sharing）

所谓伪共享，指的是在SMP(对称多处理）CPU体系结构中，内存在CPU的缓存是以Cache Line的形式存在的，一般Cache line是64个字节，也就意味着两个线程看似没有操作同一个变量，但实际上因为操作了同一个Cache line导致了线程并发时的性能下降。
这就叫做伪共享，下面是一个伪共享的例子，


{% highlight java %}

public final class FalseSharing
    implements Runnable
{
    public final static int NUM_THREADS = 4; // change
    public final static long ITERATIONS = 500L * 1000L * 1000L;
    private final int arrayIndex;
 
    private static VolatileLong[] longs = new VolatileLong[NUM_THREADS];
    static
    {
        for (int i = 0; i < longs.length; i++)
        {
            longs[i] = new VolatileLong();
        }
    }
 
    public FalseSharing(final int arrayIndex)
    {
        this.arrayIndex = arrayIndex;
    }
 
    public static void main(final String[] args) throws Exception
    {
        final long start = System.nanoTime();
        runTest();
        System.out.println("duration = " + (System.nanoTime() - start));
    }
 
    private static void runTest() throws InterruptedException
    {
        Thread[] threads = new Thread[NUM_THREADS];
 
        for (int i = 0; i < threads.length; i++)
        {
            threads[i] = new Thread(new FalseSharing(i));
        }
 
        for (Thread t : threads)
        {
            t.start();
        }
 
        for (Thread t : threads)
        {
            t.join();
        }
    }
 
    public void run()
    {
        long i = ITERATIONS + 1;
        while (0 != --i)
        {
            longs[arrayIndex].value = i;
        }
    }
 
    public final static class VolatileLong
    {
        public volatile long value = 0L;
        public long p1, p2, p3, p4, p5, p6,p7; // comment out
    }
}

{% endhighlight %}

在如下环境中测试。
硬件： cpu : 2.4 GHz Intel Core i5
	  内存  8 GB 1333 MHz DDR3
	  硬盘  ssd 镁光 128G 
	 
操作系统：  OS X 10.8 (12A269)	
 
带了cache line padding的运行时间是
20834608000ns
不带cache line padding的运行时间是
28633701000ns
## 速度提升了27.2%
