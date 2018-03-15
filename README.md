# PacketAllocator
C++ Memory allocator for packet queues that free() in roughly the same order that they alloc().

It turned out that malloc() and calloc() amount to a great deal of
the overhead of managing queued packet data.

Tuned for packet queues:

The allocator is tuned for allocations around 1000 bytes that are freed in
roughly the same order that they are allocated.

Quick start:

    {
        pktalloc::Allocator TheAllocator;

        // Allocate 1400 bytes
        uint8_t* data = TheAllocator.Allocate(1400);

        // Reallocate data to be 2000 bytes, keeping the contents of the original 1400 bytes
        data = TheAllocator.Reallocate(data, 2000, pktalloc::Realloc::CopyExisting);

        // Example of freeing memory
        // TheAllocator.Free(data);
    } // data is automatically released when TheAllocator goes out of scope

Note that you do not need to provide memory pools to the allocator - It will take care of requesting system resources as needed.


Features:
* Simple to configure/tune for new projects.
* Allocator also has a unit test and is actively used by author.
* Construct()/Destruct() to invoke object ctor/dtor after alloc.
* GetMemoryUsedBytes()/GetMemoryAllocatedBytes() accounting functions.
* Self-test integrity check functionality.
* Provides an optimized custom BitSet template that is also unit tested.

Advantages over malloc():
* Eliminates codec performance bottleneck.
* All allocation requests will be aligned for SIMD operations.
* No thread safety or debug check overhead penalties.
* No contention with allocations from users of the library.
* Simpler cleanup: All memory automatically freed in destructor.
* Self-contained library that is easy to reuse.

Disadvantages:
* Uses more memory than strictly necessary.
* Requires some tuning for new applications.
* May be slower for random access patterns or smaller sizes.

Further Discussion:

Custom allocators are worthwhile in very large projects to allow for
accounting, or when access patterns are known a priori and better
(or more predictable) performance is desired.

This allocator does not have a constant execution-time like TLSF-based
approaches, but it does run about twice as fast on average for queue-like
access patterns and medium sized allocations between 32 and 2000 bytes.

Rather than throwing a bunch of benchmarks at you, please try evaluating it
in your own project and check if it is worthwhile.  I found it was the best
option for a few of my own projects like Siamese.
