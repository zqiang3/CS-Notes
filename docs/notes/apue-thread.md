```c
#include <pthread.h>
int pthread_equal(pthread_t tid1, pthread_t tid2);
// Returns: nonzero if equal, 0 otherwise

pthread_t pthread_self(void);
// Returns: the thread ID of the calling thread

int pthread_create(pthread_t *restrict tidp, const pthread_attr_t *restrict attr, void* (*start_rtn)(void*), void *restrict arg);
// Returns: 0 if OK, error number on failure

void pthread_exit(void *rval_ptr);

int pthread_join(pthread_t tid, void **rval_ptr);
// Returns: 0 if OK, error number on failure

int pthread_cancel(pthread_t tid);
// Returns: 0 if OK, error number on failure

void pthread_cleanup_push(void (*rtn)(void *), void *arg);
void pthread_cleanup_pop(int execute);

int pthread_detach(pthread_t tid);
// Returns: 0 if OK, error number on failure

int pthread_mutex_init(pthread_mutex_t *restrict mutex, const pthread_mutexattr_t *restrict attr);
int pthread_mutex_destroy(pthread_mutex_t *mutex);
// Both return: 0 if OK, error number on failure

int pthread_mutex_lock(pthread_mutex_t *mutex);
int pthread_mutex_trylock(pthread_mutex_t *mutex);
int pthread_mutex_unlock(pthread_mutex_t *mutex);
// Both return: 0 if OK, error number on failure

int pthread_rwlock_init(pthread_rwlock_t *restrict rwlock, const pthread_rwlockattr_t *restrict attr);
int pthread_rwlock_destroy(pthread_rwlock_t *rwlock);
// Both return: 0 if OK, error number on failure

int pthread_rwlock_rdlock(pthread_rwlock_t *rwlock);
int pthread_rwlock_wrlock(pthread_rwlock_t *rwlock);
int pthread_rwlock_unlock(pthread_rwlock_t *rwlock);
// Both return: 0 if OK, error number on failure

int pthread_rwlock_tryrdlock(pthread_rwlock_t *rwlock);
int pthread_rwlock_trywrlock(pthread_rwlock_t *rwlock);
// Both return: 0 if OK, error number on failure

int pthread_attr_init(pthread_attr_t *attr);
int pthread_attr_destroy(pthread_attr_t *attr);
// Both return: 0 if OK, error number on failure

int pthread_attr_getdetachstate(const pthread_attr_t *restrict attr, int detachstate);
// Both return: 0 if OK, error number on failure

int pthread_attr_getstack(const pthread_attr_t *restrict attr, void **restrict stackaddr, size_t *restrict stacksize);
int pthread_attr_setstack(const pthread_attr_t *attr, void *stackaddr, size_t *stacksize);
// Both return: 0 if OK, error number on failure

int pthread_attr_getguardsize(const prhtread_attr_t *restrict attr, size_t *restrict guardsize);
int pthread_attr_setguardsize(pthread_attr_t *attr, size_t guardsize);
// Both return: 0 if OK, error number on failure
```

