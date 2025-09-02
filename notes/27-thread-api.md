# 27. Interlude: Thread API

The crux: how to create and control threads?
- What interfaces should the OS present for thread creation and control?
- How should these interfaces be designed to enable ease of use as well as utility?

## 27.1. Thread Creation

Thread creation interface (in POSIX):
```c
#include <pthread.h>
int
pthread_create(pthread_t    *thread,
    const pthread_attr_t    *attr,
    void                    *(*start_routine)(void*),
    void                    *arg);
```

- `thread`: is a pointer to a structure of type `pthread_t`
- `attr`: is used to specify any attributes this thread might have
    - Include setting the stack size or perhaps information about the scheduling proiority of the thread.
    - In most cases, the defaults (`NULL`) will be fine.
- `(*start_routine)(void*)`: function pointer - which funtion should this thread start running in?
- `arg`: the argument to be passed to the function where the thread begins execution.

But why do we need these void pointers? Having a void pointer as an argument to the function `start_routine` allows us to pass in any type of argument; having it as a return value allows the thread to return any type of result.

## 27.2. Thread Completion

What happens if you want to wait for a thread to complete?
You need to call the routine `pthread_join()` in order to wait for completion.

```c
int pthread_join(pthread_t thread, void **value_ptr);
```

- `thread`: is used to specify which thread to wait for.
- `*value_ptr`: is a pointer to the return value you expect to get back.
    - Because the `pthread_join()` routine changes the value of the passed in argument, you need to pass in a pointer to that value, not just the value itself.

## 27.3. Locks

```c
int pthread_mutex_lock(pthread_mutex_t *mutex);
int pthread_mutex_unlock(pthread_mutex_t *mutex);
```

The intent of the code:
- If no other thread holds the lock when `pthread_mutex_lock()` is called, the thread will acquire the lock and enter the critical secion.
- If another thread does indeed hold the lock, the thread trying to grab the lock will not return from the call until it has acquired the lock.

SKIPPED

## 27.4. Condition Variables

**Condition variables** are useful when some kind of signaling must take place between threads, if one thread is waiting for another to do something before it can continue.

```c
int pthread_cond_wait(pthread_cond_t *cond, pthread_mutex_t *mutex);
int pthread_cond_signal(pthread_cond_t *cond);
```

The first routine, `pthread_cond_wait()`, puts the calling thread to sleep, and thus waits for some other thread to signal it.

WIP

## 27.5. Compiling and Running 

SKIPPED
