16.2.1 How Do We use Cache Effectively?

Caching is one of the most important mechanisms of performance boosting. We spoke about the general concepts of caching in Chapter4. This section will further investigate how to use these concepts effectively.

We want to start by elaborating that contrary to the spirit of von Neumann architecture, the common CPUs have been using separate caches for instructions and data for at least 25 years. Instructions and code inhabit virtually always different memory regions, which explains why separate caches are more effective. We are interested in data cache.

By default, all memory operations involve cache, excluding the pages marked with cache-write-through and cache-disable bits \(see Chapter4\).

Cache contains small chunks of memory of 64 bytes calledcache-lines,alignedon a 64-byte boundary.

Cache memory is different from the main memory on a circuit level. Each cache-line is identified by atag—an address of the respective memory chunk. Using special circuitry it is possible to retrieve the cache line by its address very fast \(but only for small caches, like 4MB per processor, otherwise it is too expensive\).

When trying to read a value from memory, the CPU will try to read it from the cache first. If it is missing, the relevant memory chunk will be loaded into cache. This situation is calledcache-missand often makes ahugeimpact on program performance.

There are usually several levels of cache; each of them is bigger and slower.  
 TheLL-cacheis the last level of cache closest to main memory.  
 For programs with good locality, caching works well. However, when the locality is broken for a piece

of code, bypassing cache makes sense. For example, writing values into a large chunk of memory which will not be accessed any time soon is better performed without using cache.

The CPU tries to predict what memory addresses will be accessed in the near future and preloads the relevant memory parts into cache. It favors sequential memory accesses.

This gives us two important empirical rules needed to use caches efficiently.

* Try to ensure locality.

* Favor sequential memory access \(and design data structures with this point in mind\).



