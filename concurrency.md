As I read through the book, I considered more situations than the book does. It's probably because I missed some invariants so that some situations are actually redundant. Nonetheless, I got the idea of it.

## Snapshot-at-the-Beginning (SATB)

The initial mark has to stop the world. After that, the root objects are scanned concurrently with the mutator execution. The objects referenced by these objects are added to a mark stack, waiting to be scanned. SATB eventually scans a **super set** of the live objects as it conservatively store the old references that are modified during the tracing. 

The mutators change the object graph in two ways as the collectors traverse it.
- write to a reference field, which makes a live object dead (set L)
  - there is no way to make a dead object live though, because there is no pointer to it to copy from
- create new objects, which could be dead or live along with the execution (set N)

How to catch them?
- set L
  - use write-barrier that catches every heap write into a reference field
  - if the old(to be changed) reference has already been scanned, then no need to add it to task stack again

When a reference write happens, the host object could be in one of the three sets (tri-color context)
- already scanned (black)
- in the queue(or stack), wait to be scanned (gray)
- not visited (white)

gray set/mark queue/mark stack would refer to the same thing in this discussion.

For slot-based SATB, my pseudocode would be as following
```c++
write_barrier_slot(Object* src, Object** slot, Object* new_ref) {
  old_ref = *slot;  
  if (src in white or in gray) { // its fields haven't been scanned
    remember(old_ref) if old_ref not already in gray set;
  }
  else if (src in grey) {
  *slot = new_ref;
}
```
Possible dead objects that are scanned by SATB:
We discuss the liveness of an object with regard to the time when concurrent tracing starts.  

- set N
  - monitor allocation routine
  
