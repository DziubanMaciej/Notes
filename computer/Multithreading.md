parentTopic:: [[Computer]]

# Basic definitions
- **CAS** - compare and swap instruction. Basic building block for most of synchronization mechanisms, including atomics.
- **Atomic** - the most basic multithreading primitive requiring hardware support. It is often used as a building block for more complex mechanisms.
- **Spinlock**, busy loop - spinning in a loop waiting for a condition to be satisfied. The most efficient if the wait period is very short - it doesn't involve yielding the thread, context switching, etc.
- **Futex** - **f**ast **u**serspace mu**tex**. A synchronization mechanism available in Linux. Programs can call `futex(FUTEX_WAIT)` to wait until it's free. This adds current thread to the waitlist in the kernel. It will be woken up when another thread calls `futex(FUTEX_WAKE)`. It's a very low level primitive used to build more complex mechanisms and should generally not be directly used. Futex can be used inter-process if allocated in shared memory, see this [nice example](http://faculty.washington.edu/wlloyd/courses/tcss422/examples/Chapter28/futex_demo.c).
- **Mutex** - **mut**ual **ex**clusion. Used to guard access to a resource. Requires to be locked and unlocked by the same thread. This is a good thing, it provides protection. Mutex is used to establish critical sections.
- **Semaphore** - used for counting things atomically. The count starts at 0 and cannot become negative. Binary semaphore is just a semaphore counting up to 1. Semaphore is not tied to any thread of execution - acquiring and releasing don't have to be done in the same thread. A semaphore has two methods:
    - `release()` (or `signal()`) is used to increment the counter and potentially unblock waiting threads.
    - `acquire()` or (`wait()`) is used to decrement the counter and potentially block execution if a counter is already at 0. 
- **Hybrid lock**, hybrid mutex, adaptive lock - a lock that acts as a spinlock for a short amount of time in attempt to avoid yielding the thread and acts as a real sleeping mutex in case of failure.
- **Monitor** - an object/entity/module/instance (depends on the context) offering methods that can be safely called from any number of threads. The simplest form is just locking a mutex at the beginning of each function. It may, however, be more sophisticated, e.g. some condition may have to be met in order for a method to be called, which would require introducing a condition variable.
- **Condition variable** - synchronization mechanism used for waiting for certain condition to be true. Condition variable concept is strictly tied to mutexes. The fundamental rule: never change, **nor even check**, the predicate data state unless the predicate mutex is locked. Missing notifications come from changing the state without the lock.
    - The `wait()` method takes a locked mutex as an argument. It unlocks the mutex and waits for the condition variable to be woken up. When it's woken up, the mutex is locked again and the program has to check the condition, which may still not be true. If it's false, it should call `wait()` again.
    - The `notify()` method notifies a waiting thread to wake up. A notification doesn't necessarily mean the condition required by this thread is met, but rather that something has just changed and it *might* be met. Note that `notify()` does not have to be called while holding the mutex.
    - The `notify_all()` method is needed in case there can be multiple waiting threads. All of them will be notified. Note that they will still have to lock the mutex before exiting `wait()`, so they will not run in parallel.





# Programming techniques
## Lock-free programming
## Yielding (sleep 0?)
## Thread-local memory
## False sharing avoidance
## Proper memory ordering
## Double lock pattern (check, lock, check)
## Static init in C++

## Busy looping
Busy loops should generally not be implemented in normal user-space code on multithreaded systems. Sometimes it may make sense, e.g. when the waiting period is expected to be very short. It could help avoid going into more expensive synchronization mechanisms when not needed. Longer waits must be done by proper mechanisms because of power efficiency and wasted CPU time.

Instead of naively looping, it's good to use `pause` instruction (spin loop hint). Its opcode is `F3 90`, which is actually `rep nop`. This makes it compatible with older processors - `pause` was introduced in Pentium 4. Using `pause` should improve power consumption of spin loops, as well as performance after exiting them. See [reference](https://www.felixcloutier.com/x86/pause.html) for details. Use the `_mm_pause()` intrinsic from `<immintrin.h>`.





# Implementation of STL synchronization in x86
## std::atomic
For some atomic operations a simple `mov` or `xchg` can be used as these operations are already atomic on x86. Instructions that perform read-modify-write operation (e.g. `cmpxchg`, `add`) are usually not atomic by default. However, some of them can be made atomic with the `LOCK` prefix. The prefix also prevents memory access reordering. See [reference](https://www.felixcloutier.com/x86/lock) for details. Some datatypes cannot be implemented using atomic instructions and they may need an actual lock managed internally by `std::atomic`. This can be queried from `is_lock_free()` and `is_always_lock_free()`.

Implementations of various `std::atomic` methods under GCC:
- `store()` uses `__atomic_store_n` builtin function, which gets compiled to a single instruction
    - `mov` when `__ATOMIC_RELAXED` or `__ATOMIC_RELEASE`
    - `xchg` when `__ATOMIC_SEQ_CST` (why not `mov`?)
- `load()` uses `__atomic_load_n` builtin function, which gets compiled to a single `mov` instruction, regardless of memory order. This is because `mov` is already atomic and sequentially consistent.
- `fetch_add()` uses `__atomic_fetch_add` builtin function, which gets compiled to a single instruction, regardless of memory order:
    - `lock add`, if the result (value before addition) is not used.
    - `lock xadd`, if the result is used.
- `fetch_and()` uses `__atomic_fetch_and` builtin function, which gets compiled to:
    - `lock and` if the result (value before AND) is not used.
    - a loop if the result is used. Because there's no x86 instruction that would perform this task (like `xaddl` in case of addition), a more sophisticated approach is needed.
        ```assembly
            movl    (%rdi), %eax
        .L12:
            movl    %eax, %ecx
            movl    %eax, %edx
            andl    $7, %ecx
            lock cmpxchgl   %ecx, (%rdi)
            jne     .L12
        ```
        Explanation of above assembly code:
        - read the memory value to `eax`
        - perform the `and` operation with the operand (here hardcoded as `7`) and store result in `ecx`
        - perform `lock cmpxchg` to **try** to set the memory value to `ecx` if it's still equal to `eax`. The usage of `eax` is implicit, see [reference](https://www.felixcloutier.com/x86/cmpxchg).
            - If it's still equal to `eax`, the instruction will store the `ecx` value and set the `Z` flag.
            - If it's no longer equal to `eax`, it means some other thread changed this value. The instruction will clear the `Z` flag and read the memory value to `eax`. The following `jne` instruction will loop back to `.L12` label and try again.
- `compare_exchange_weak()`/`compare_exchange_strong()` use `__atomic_compare_exchange_n` builtin function, which gets compiled into `lock cmpxchg` followed by `sete` if the return value is used. There's no difference between `weak` and `strong` variants on x86, which already implements a strongly ordered memory model.

## std::mutex
The core functionality of a mutex is usually based on some kernel-managed synchronization, e.g. futexes on Linux. The implementation can get very complicated with a lot of optimizations that can be added, like adaptive locking with exponential backoff or unlocking futex with a CAS operation (manually setting it for 0 to 1 is valid) instead of a syscall.

Rough code trace of `std::mutex::lock()` under GCC:
- `__gthread_mutex_lock()` (libgcc)
- `pthread_mutex_lock()` (glibc, *nptl/pthread_mutex_lock.c*)
- `LLL_MUTEX_LOCK()` macro (glibc, *nptl/pthread_mutex_lock.c*)
- `lll_lock()` macro (glibc, *nptl/lowlevellock.h*)
- `__lll_lock()` macro (glibc, *nptl/lowlevellock.h*)
- `__lll_lock_wait()` function (glibc *nptl/lowlevellock.c*)
- `lll_futex_wait` macro (glibc, *nptl/lowlevellock-futex.h*)
- `lll_futex_timed_wait` macro (glibc, *nptl/lowlevellock-futex.h*)
- `lll_futex_syscall` macro (glibc, *nptl/lowlevellock-futex.h*)
- `INTERNAL_SYSCALL` macro. It goes on for a few levels and finally expands to inline assembly `int 0x80`, which, unsurprisingly, is a syscall.

## std::condition_variable
TODO: write a generic explanation
-  Waiting
    - `wait()`/`wait_until()` (gcc, *libstdc++-v3/include/bits/std_mutex.h*)
    - `__gthread_cond_wait`/`__gthread_cond_timedwait` (gcc, *libgcc/gthr-posix.h*)
    - `pthread_cond_wait`/`pthread_cond_timedwait`/`pthread_cond_clockwait` (glibc, *nptl/pthread_cond_wait.c*)
    - `__pthread_cond_wait_common` (glibc, *nptl/pthread_cond_wait.c*)
    - Complex implementation based on mutexes, atomics and futexes
- Signalling
    - `notify_one`/`notify_all` (gcc, *libstdc++-v3/include/bits/std_mutex.h*)
    - `__gthread_cond_signal`/`__gthread_cond_broadcast` (gcc, *libgcc/gthr-posix.h*)
    - `pthread_cond_signal`/`pthread_cond_broadcast` (glibc, *nptl/pthread_cond_signal.c*/*nptl/pthread_cond_broadcast.c*)
    - Complex implementation based on mutexes, atomics and futexes
