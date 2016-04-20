`SIGABRT` is raised by the `abort(3)` function. It's impossible to tell exactly what's going on in your program without more information, but the most common reasons that abort() gets called are:

- Your sending a message to an Objective-C object that doesn't support/implement that message. This results in the dreaded `unrecognized selector sent to instance` error.

- You have a failed assertion somewhere. In non-debug builds that define the macro `NDEBUG`, the standard library macro `assert(3)` calls `abort()` when the assertion fails.

- You have some memory **stomping**/**allocation** error. When malloc/free detect a corrupted heap, the may call `abort()` (see, e.g. [this question](http://stackoverflow.com/questions/3171194/getting-program-received-signal-sigabrt-in-iphone-sdk)).

- You're throwing an uncaught exception (either a C++ exception or an Objective-C exception).

In almost all cases, the debug console will give you a little more information about what's causing `abort()` to be called, so always take a look there.


REF:
- [Program received signal SIGABRT](http://stackoverflow.com/questions/3887609/program-received-signal-sigabrt?answertab=votes#tab-top)
