# Lab 2 More Java Concurrency
![IMG2](https://images.unsplash.com/photo-1484417894907-623942c8ee29?ixlib=rb-4.0.3&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=1932&q=80)
# This is a programming exercise in which you will implement one of two algorithms in four different ways.

Choose one of the following two algorithms:

(a) Mergesort (b) Quicksort

- Implement a sequential version of the algorithm

- For the following tasks make sure you are actually using more than one worker thread. How can you check this?

- Implement a parallel version using ExecutorService. See chapter 16 in HSLS for details on how to use ExecutorService.

- Implement a parallel version using a ForkJoinPool and RecursiveAction.

Requires a Java version 7 or later.

- Implement a parallel version using the parallel streams and lambda functions that were introduced in Java 8.

Compare the four different implementations using a suitably large dataset, say 1.000.000 items. Run your solutions on Dardel with at least three different physical thread counts from 4 to max, plot your results and give a brief explanation. 

**Background reading:**

HSLS section 16.1 has an introduction to scheduling and work distribution
Read section on executors from the Java Tutorial Links to an external site.
Read section on aggregators from the Java Tutorial Links to an external site.
Note on tailoring the number of threads:

For ExecutorService and ForkJoinPool, you can use Executors.newFixedThreadPool(threads) and new ForkJoinPool(threads) respectively. For parallel streams, this is more difficult. You can check here Links to an external site. for possible solutions. In the worst case, just compare a sequential version with a parallel streams version using the default worker pool. 

PDC Bash Scripts for running your code.