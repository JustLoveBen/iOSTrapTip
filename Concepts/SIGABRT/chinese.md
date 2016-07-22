平常我们写程序的时候经常会遇到这样的问题。program received signal:SIGABRT 以及EXC_BAD_ACCESS
SIGABRT 一般是过度release 或者 发送 unrecogized selector导致。
EXC_BAD_ACCESS 是访问已被释放的内存导致。
查了下StackOverflow。找到下面的答案，说道linux内核下面了！
SIGABRT is raised by the abort(3) function. It's impossible to tell exactly what's going on in your program without more information, but the most common reasons that abort() gets called are:

Your sending a message to an Objective-C object that doesn't support/implement that message. This results in the dreaded "unrecognized selector sent to instance" error.（你发送给对象一个它并不支持或者是没实现的消息，这导致可怕的 "unrecognized selector sent to instance" 错误）
You have a failed assertion somewhere. In non-debug builds that define the macro NDEBUG, the standard library macro assert(3) calls abort() when the assertion fails.（某个地方你做了个错误的判断）
You have some memory stomping/allocation error. When malloc/free detect a corrupted heap, the may call abort() (see, e.g. this question)
You're throwing an uncaught exception (either a C++ exception or an Objective-C exception)（内存分配出错，内存泄露，会调用abort()）
In almost all cases, the debug console will give you a little more information about what's causingabort() to be called, so always take a look there.

（多数情况下，调试控制台会给你更多的信息关于导致了abort的原因。所以仔细看）。
