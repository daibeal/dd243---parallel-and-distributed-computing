# Introduction

The purpose of the lab is to have you explore a fairly advanced lock-free concurrent data structure in terms of performance and scalability, and to devise methods to instrument and test your implementation for correctness.

You will use many of the skills learned in the first 2/3 of the course to implement a highly concurrent data structure, a lock-free skiplist. The basic algorithms are given in HSLS, ch 14.4. Start by reading about lazy and lock-free skiplists in HSLS ch 14. In the lab, you will modify and complement the code samples given in the textbook. Other resources are to be found in the textbook companion website here Links to an external site., including more java code samples. It is ok to make use of these, but you should of course understand what any code you import is doing, and why, in detail. We may well have to discuss some of this during the lab presentations. And, as always, carefully document the origin of any code snippets that are not yours in your hand-in.

## Tasks

1. Generating Population

Skeleton code and bug-free LockFreeSkipList: https://github.com/HAKarlsson/pardis-lab-3 Links to an external site.. Book code is buggyFinish the UniformPopulation and NormalPopulation classes, the first class should generate integers chosen uniformly at random and the second should use a normal distribution with a suitably chosen mean and variance, to ensure that the mean is distinct but all points have a "reasonable" minimum probability of getting sampled. Use the supplied test to verify that the two populations have roughly the expected means and variances.

Devise a little program that will populate your skiplist using the uniform or normal populations with 10^7 integers of range 0-10^7 (in the first instance, later we will vary this range).

Note. Since our skiplist implements a set interface (no duplicates), not all elements of the generated population will be successfully added, and the populated skiplist will have less than 10^7 elements. In other words, generate 10^7 integers of range 0-10^7, and attempt to add them to the skiplist, if an add fails, do not generate a new integer as a replacement.

2. Measuring Execution Time

Devise a test class that, for a given number of threads up to 48, exposes the populated skiplists from Task 1 to an adjustable mix of 10^6 add/remove/contains operations. Make sure the test class does not impose any sequentialisation constraints. Using Dardel, run the test with 10 runs with four different thread counts (e.g., 4, 16, 32, and 64 threads) on both populations using the following distributions of operations:

- 10% add, 10% remove, 80% contains
- 50% adds and 50% removes

Remember that the Java runtime needs to be warmed up before making any measurements, for instance by running each experiment twice, once for warm-up and once for measurement.

Plot the average execution time for each case. Explain your observations. 

Before using Dardel, verify that the program works locally by plotting the execution time with different ratios of operations and number of threads. If the results make sense, continue with Dardel.

Example Bash Scripts for running your code on Dardel.

3. Identifying linearization points

We believe the skiplist implementation is linearisable, and we wish to test this hypothesis. We want to use System.nanoTime() to record the time at which the linearisation point was encountered. To do this, identify the linearisation points and sample the times they are crossed by inserting calls to System.nanoTime() at appropriate places in the code. This needs to be done with care.

Hint: If you are uncertain where the methods are linearized, why, and under which conditions, you may consult the textbook. (But remember you must be able to explain your choice)

Recording the time of the linearization using System.nanoTime() will introduce measurement errors, since the traversal of the linearisation point and the sampling of the clock is not an atomic operation. It is possible for a thread to intervene between the point of sampling the time and the time when the linearisation point is traversed. This may cause a trace constructed by ordering the sampled events according to the time samples to violate the sequential specification. For example, contains(x) may erroneously return true if if the last prior event on x was to remove x. The purpose of the remaining parts of the lab is to devise ways of instrumenting the skiplist as non-intrusively as possible, in order to validate that our skiplist implementation is indeed linearisable, and to estimate the error resulting from any imprecision in our sampling of the linearisation point times. Below, we suggest you explore three different ways to accomplish such sampling. If you get inspired to devise a method of your own, feel free to substitute oneof the methods suggested below with your approach.

3.1. Locked time sampling

First, we use a global lock to protect the interval between the linearisation point and the time sampling event (in whatever order you implement this). This will prevent measurement errors since these intervals are now executed atomically, but it is also highly intrusive, due to the use of a global lock, which will introduce sequential execution constraints into the system that are not present in the original system. To get an idea of the global lock's effect on the execution time, repeat some of the measurements from Task 2 and compare. Do you observe a noticeable effect as the number of threads increases? If so, how do you explain this?

To answer if you've identified the correct linearisation points, record the function, its arguments, return value, and linearisation point to a global log. If the linearisation points are correctly identified and the log is computed correctly, you should be able to verify that the computed log satisfies its sequential specification, for instance that contains(x) only succeeds when the most recent add/remove of x was an add, etc. Write a small routine checking that the sequential specification is met and verify that the logs obtained from the experiments pass your check.

# 3.2. Lockfree time sampling
For a less intrusive approach we try instead to log the time samples without using a global lock. This will in general produce sampling errors since the interval between crossing the linearisation points and sampling the time is no longer protected. The purpose of the remaining part of the lab is to get a feel for the magnitude of this error. 

Perform the same experiment as in Task 3.1 but remove the global lock and build the log locally, i.e., per thread at runtime, by writing the samples into a suitable structure like a large enough thread local array or linked list. Keep in mind that performing the sampling and writing the log are two quite separate activities. At the end of execution the local logs can then be merged into a single global log ordered by sampled time instances. We expect that `System.nanoTime()` samples the time at a fine enough granularity that duplicate time samples are eliminated.

To build a thread local log, use the `ThreadLocal` class or return the linearization point using the following pattern:

```
boolean add(T x, long linTime[]) {
    ...
    linTime[0] = System.nanoTime();
    ...
}
```

# 3.3. Lockfree Multiple-producer-single-consumer queue
A special thread is assigned the task of aggregating the samples at runtime. This will obviously reduce the number of "payload threads" that perform operations on the actual skiplist by one. This aggregation can be performed by implementing a lockfree multiple producer-single consumer queue that the skiplist threads can use to pass samples on to the aggregator thread. Implement both these methods carefully, remembering to introduce as little extra synchronisation as possible. Use your knowledge of the Java Memory Model and the happens-before relation to determine what new execution constraints, if any, are introduced on your skiplist by your two (hopefully) non-blocking logging methods.

# 3.4. Testing Lockless Methods
If both of the lockless methods are correct and maximally non-intrusive they should produce similar results, so as the final part of the lab we try to check if this is the case. Note that the probability of a race occurring that can result in a log violating the sequential specification must increase as the integer value range decreases. For both logging methods repeat the experiment of Task 2, 10 times for 50% remove, 50% add, and 64 cores, first for the 107 then 106 etc value range until either a violation of the sequential specification is found or the value range cannot be reduced further. What do you find, and do your two non-blocking logging methods produce consistent results? If not, why?

# Submission
Submit Java code, script used for Dardel, plots from experiments, and more in an archive. Include a README file describing who worked on the lab, the description of other files, and how to run your code. The archive filename should be "Lab3LastnameFirstnameLastnameFirstname.tar" or just "Lab3LastnameFirstname.tar" if you do the lab alone.

We allow the following archival types: tar, tgz, tar.gz, and zip.

If you can not hand in your submission with one of the above archival types, email [henrik10@kth.se](mailto:henrik10@kth.se)