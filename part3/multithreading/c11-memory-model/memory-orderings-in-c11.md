17.12.3 Memory Orderings in C11  
The memory ordering is described by one of enumeration constants \(in order of increasing strictness\).

* memory\_order\_relaxedimplies the weakest model: any memory reordering is possible as long as the single thread program’s observable behavior is left untouched.

* memory\_order\_consumeis a weaker version ofmemory\_order\_acquire.

* memory\_order\_acquiremeans that the load operation has an acquire semantics.

* memory\_order\_releasemeans that the store operation has a release semantics.

* memory\_order\_acq\_relcombines acquire and release semantics.

* memory\_order\_seq\_cstimplies that no memory reordering is performedfor all operations that are marked with it, no matter which atomic variable is being referenced.



By providing an explicit memory ordering constant, we can control how we want to allow the operations to beobservablyreordered. It includes both compiler reorderings and hardware reorderings, so when  
 the compiler sees that compiler reorderings do not provide all the guarantees we need, it will also issue platform-specific instructions, such assfence.

Thememory\_order\_consumeoption is rarely used. It relies on the notion of “consume operation.” This operation is an event that occurs when a value is read from memory and then used afterward in several operations, creating a data dependency.

In weaker architectures such as PowerPC or ARM its usage can bring a better performance due to exploitation of the data dependencies to impose a certain ordering on memory accesses. This way, the costly hardware memory barrier instruction is spared, because these architectures guarantee the data dependency ordering without explicit barriers. However, due to the fact that this ordering is so hard to implement efficiently and correctly in compilers, it is usually mapped directly tomemory\_order\_acquire, which is a slightly stronger version. We do not recommend using it. Refer to \[30\] for additional information.

Theacquireandrelease semanticsof these memory ordering options correspond directly to the notions we studied in section 17.7.

The memory\_order\_seq\_cst corresponds to the notion of sequential consistency, which we elaborated in section 17.4. As all non-explicit operations with atomics accept it as a default memory ordering value, C11 atomics are sequentially consistent by default. It is the safest route and also usually faster than mutexes. Weaker orderings are harder to get right, but they allow for a better performance as well.

The atomic\_thread\_fence\(memory\_order order\)allows us to insert a memory barrier \(compiler  
 and hardware ones\) with a strength corresponding to the specified memory ordering. For example, this operation has no effect formemory\_order\_relaxed, but for a sequentially consistent ordering in Intel 64 themfenceinstruction will be emitted \(together with compiler barrier\).



