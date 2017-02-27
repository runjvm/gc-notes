## Tricolor Marking Abstraction and Invariants

Tricolor Marking abstracts away how the graph is traversed, breadth-first or depth first. According to this abstraction, the following invariant perverses:

- No black object points to white object, because black objects are assumed to be finished.

The mutators might break this invariant during the execution and if the following two conditions hold at the same time, the collector could miss some garbage permenantly.
- A black object's field is modified to point to a white object.
- All other pointers to this white object are destroyed before the collector sees it.

## Incremental update
- Prevent the first condition.
- Add the white object referenced by the written pointer to gray set, or add the black object to gray set.
- The new allocated objects considered white by Dijkstra based on the assumption that most objects are short-lived, so a second traversal of the root set is needed to find those new live objects.

## Snapshot At the Beginning
- Prevent the second condition.
- Copy-on-write, when there's a write to a black object's field, the old reference is recorded, including reference on the stack.
- It's named so because it only scans the live objects when the tracing starts.
- The new allocated objects are thus considered black objects, as if they've already been scanned, and left to the next GC.
