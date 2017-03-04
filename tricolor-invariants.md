## Tricolor Marking Abstraction and Invariants

Tricolor Marking abstracts away how the graph is traversed, breadth-first or depth first. According to this abstraction, the following invariant perverses:

- No black object points to white object, because black objects are assumed to be finished

The mutators might break this invariant during the execution and if the following two conditions hold at the same time, the collector could miss some garbage permenantly.
- A black object's field is modified to point to a white object
- All other pointers to this white object are destroyed before the collector sees it

Note that only write to black objects need to be considered. Because gray and white objects are not yet scanned anyway. 

Also mutators and collectors can not operate on a same object at the same time, it has to be atomic, since object is now a shared resource.

There are two write barrier strategy to prevent these two conditions from both holding. Note that write barrier is less expensive as read barrier, as it happens less frequently.

## Incremental update
- Prevent the first condition
- Add the white object referenced by the written pointer to gray set, or add the black object to gray set
- The new allocated objects considered white by Dijkstra based on the assumption that most objects are short-lived, so a second traversal of the root set is needed to find those new live objects

## Snapshot At the Beginning
- Prevent the second condition
- Conceptually, at the beginning of tracing, a copy-on-write virtual copy of the graph of reachable objects are made
- When there's a write to graph, the old reference is recorded, including reference from the stack
- It's named so because it only scans the live objects when the tracing starts
- The new allocated objects are thus considered black objects, as if they've already been scanned, and left to the next GC
- This implies that no object can be freed during the collection (not count old garbage)
- The view of the reachibility graph is the old graph + new allocated objects

## Old References Becomes Garbage
When a write happens to a black object, if this makes an old black or gray object, IU doesn't do anything, still keeps them. Since there is no information recorded that this old object is only referenced by the black object, and thus recognizing them as garbage would require a retrace since they've already been scanned or in the queue.

When a write happens to a white or gray object, if this makes another old object becomes garbage, this old garbage will not be traced by IU. In contrast, these "new" garbages will still be considered as live by SAB since the old reference is copied.
