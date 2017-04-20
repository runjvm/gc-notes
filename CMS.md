

The concurrent collection cycle typically includes the following steps:

- Stop all application threads, identify the set of objects reachable from roots, and then resume all application threads.
	- The goal of this phase is to mark all the objects in the Old Generation that are either direct GC roots or are referenced from some live object in the Young Generation. The latter is important since the Old Generation is collected separately.

- Concurrently trace the reachable object graph, using one or more processors, while the application threads are executing.


- Concurrently retrace sections of the object graph that were modified since the tracing in the previous step, using one processor.
	- While the previous phase was running concurrently with the application, some references were changed. Whenever that happens, the JVM marks the area of the heap (called “Card”) that contains the mutated object as “dirty” (this is known as Card Marking). In the pre-cleaning phase, these dirty objects are accounted for, and the objects reachable from them are also marked. The cards are cleaned when this is done.

- Concurrent Abortable Preclean. 
	- Again, a concurrent phase that is not stopping the application’s threads. This one attempts to **take as much work off the shoulders of the stop-the-world Final Remark as possible**. The exact duration of this phase depends on a number of factors, since it iterates doing the same thing until one of the abortion conditions (such as the number of iterations, amount of useful work done, elapsed wall clock time, etc) is met.

- Stop all application threads and retrace sections of the roots and object graph that may have been modified since they were last examined to finalize marking all live objects in the Old Generation, and then resume all application threads.
	- New allocated objects are also dealt with in this phase.

- Concurrently sweep up the unreachable objects to the free lists used for allocation, using one processor.

- Concurrently resize the heap and prepare the support data structures for the next collection cycle, using one processor.

[Oracle CMS Overview](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/cms.html)
[More details](https://plumbr.eu/handbook/garbage-collection-algorithms-implementations/concurrent-mark-and-sweep)
