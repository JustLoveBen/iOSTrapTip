abort()函数首先解除进程对SIGABRT信号的阻止，然后向调用进程发送该信号。abort()函数会导致进程的异常终止除非SIGABRT信号被捕捉并且信号处理句柄没有返回。

The abort() first unblocks the SIGABRT signal, and then raises that signal for the calling process (as though raise(3) was called).  This results in the abnormal termination of the process unless the SIGABRT signal is caught and the signal handler does not return (see longjmp(3)).

If the abort() function causes process termination, all open streams are closed and flushed.

If the SIGABRT signal is ignored, or caught by a handler that returns, the abort() function will still terminate the process.  It does this by restoring the default disposition for SIGABRT and then raising the signal for a second time.
