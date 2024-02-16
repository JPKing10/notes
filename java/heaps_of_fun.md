# hello heap
- work in progress
- some simple notes on jvm heap memory and garbage collector (hotspot)

- all objects live in a memory region called the Java heap
- a garbage collector automates memory management on the heap
    - it allocates and deallocates memory
    - maybe a little misleading to call it garabage collector... maybe memory manager would be a better choice
- the heap size can grow during execution to a specified maximum size
- heap is organised into different regions to make garbage collection more efficient
- full heap regions trigger garbage collection (gc)
- gc clears out unused objects from the heap
    - first, unused object are marked
        - objects without live references are counted as unused
            - simple approach is to follow all references, starting from a root object, and mark all reached objects as live
    - then, marked objects are deleted
        - really, the memory allocator stores a reference to the free spaces
    - this can fragment memory, so free space is dispersed between live objects
        - hence, the memory can be compacted, where live objects are shifted in memory to create a contiguous free region
    - free memory could be returned to os or used for objects
- not all gcs gc the same
    - stop-the-world
        - application suspended, then gc runs
        - simpler because application wont be changing objects
        - better for throughput, like a server processing trades
    - concurrent/pause free
        - run gc alongside application
        - more complex because application can change objects, hence more overhead
        - sometimes stops still required
        - better for predictable latency
    - hotspot use a mix of approaches
- generational garbage collection
    - improves efficiency of gc based on observations of typical application object lifetimes
        - lots of objects live for a short time
        - some objects live for a medium time
        - and some objects live for the duration of the application
    - insight: partition the heap by lifetime so that objects considered for gc can be limited
        - heap is partitioned into young and old generations
        - when the young generation is full a minor gc is triggered
            - this gc only targets the young generation, which is likely to have a high proportion of dead objects
        - when the old generation is full a major gc is triggered
            - gc targets the entire heap but can happen less frequently
        - young objects which live through gc can be promoted to the old generation
    - young generation is partitioned into eden and two survivor spaces
        - at the end of gc eden and one of the survivor spaces is empty
        - during gc live objects in young heap are moved into the empty survivor space
        - objects can age into the old generation if they survived enough gc cycles or if the survivor space gets full



- hotspot gc tuning
    - key parameters are maximum pause-time goal, throughput goal, and heap size
    - heap size and gc frequency can change at runtime to try and achieve these goals
    - logging
        - `-Xlog:gc*`
            - log all gc tagged logs
    - maximum pause-time goal
        - `-XX:GCTimeRatio=nnn`
            - maximum pause-time goal is `nnn` millis
    - throughput goal
        - `-XX:GCTimeRatio=nnn`
            - the ratio of application time to garbage collection time
            - max target of `1/(1 + nnn)` of the runtime for gc pauses
    - heap size
        - `-Xms=nnn`
            - minimum heap size
        - `-Xmx=nnn`
            - maximum heap size
            - higher heap usage can increase gc pause time
        - generation sizes
            - `XX:NewRatio=nnn`
                - ratio of old to new generation size
                - default `2`

## Further Reading

- [Java Memory Management White Paper](https://www.oracle.com/technetwork/java/javase/memorymanagement-whitepaper-150215.pdf): A comprehensive white paper from Oracle providing in-depth insights into Java memory management.

- [Garbage Collection Tutorial](https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html#:~:text=The%20Young%20Generation%20is%20where,objects%20is%20collected%20very%20quickly.): An Oracle tutorial offering hands-on guidance on Java garbage collection, with a focus on the Young Generation.

- [Memory Segments and Arenas in Java](https://docs.oracle.com/en/java/javase/21/core/memory-segments-and-arenas.html): Official documentation explaining Java memory segments and arenas, providing a detailed understanding of memory management concepts.

- [Java Garbage Collection Tuning](https://docs.oracle.com/en/java/javase/17/gctuning): Oracle's documentation on garbage collection tuning, offering guidelines and parameters for optimizing garbage collection performance.

- [Java Virtual Machine Options](https://www.oracle.com/java/technologies/javase/vmoptions-jsp.html): A comprehensive list of Java Virtual Machine (JVM) options, including those related to garbage collection and memory management. Helpful for configuring JVM behavior.
