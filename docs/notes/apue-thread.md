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
```

