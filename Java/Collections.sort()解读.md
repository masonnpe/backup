---
title: Collections.sort()解读
categories: Java
tags: [Collections]
---

Collections.sort方法底层就是调用的Array.sort

```java
static <T> void sort(T[] a, int lo, int hi, Comparator<? super T> c,
                         T[] work, int workBase, int workLen) {
        assert c != null && a != null && lo >= 0 && lo <= hi && hi <= a.length;

        int nRemaining  = hi - lo;
        if (nRemaining < 2)
            return;  // array的大小为0或者1就不用排了

        // 当数组大小小于MIN_MERGE(32)的时候，就用一个"mini-TimSort"的方法排序
        if (nRemaining < MIN_MERGE) {
            // 将最长的递减序列，找出来，然后倒过来
            int initRunLen = countRunAndMakeAscending(a, lo, hi, c);
            // 长度小于32的时候，是使用binarySort的
            binarySort(a, lo, hi, lo + initRunLen, c);
            return;
        }

        // 先扫描一次array，找到已经排好的序列，然后再用刚才的mini-TimSort，然后合并
        TimSort<T> ts = new TimSort<>(a, c, work, workBase, workLen);
        int minRun = minRunLength(nRemaining);
        do {
            // Identify next run
            int runLen = countRunAndMakeAscending(a, lo, hi, c);

            // If run is short, extend to min(minRun, nRemaining)
            if (runLen < minRun) {
                int force = nRemaining <= minRun ? nRemaining : minRun;
                binarySort(a, lo, lo + force, lo + runLen, c);
                runLen = force;
            }

            // Push run onto pending-run stack, and maybe merge
            ts.pushRun(lo, runLen);
            ts.mergeCollapse();

            // Advance to find next run
            lo += runLen;
            nRemaining -= runLen;
        } while (nRemaining != 0);

        // Merge all remaining runs to complete sort
        assert lo == hi;
        ts.mergeForceCollapse();
        assert ts.stackSize == 1;
    }
```

<!--more-->