摘自：[Are “EXC_BREAKPOINT (SIGTRAP)” exceptions caused by debugging breakpoints?](http://stackoverflow.com/questions/2611607/are-exc-breakpoint-sigtrap-exceptions-caused-by-debugging-breakpoints)

`Are “EXC_BREAKPOINT (SIGTRAP)” exceptions caused by debugging breakpoints?`

No. Other way around, actually: A SIGTRAP (trace trap) will cause the debugger to break (interrupt) your program, the same way an actual breakpoint would. But that's because the debugger always breaks on a crash, and a SIGTRAP (like several other signals) is one type of crash.

SIGTRAPs are generally caused by NSExceptions being thrown, but not always—it's even possible to directly raise one yourself.

I've now noticed that I forgot to remove some debugging breakpoints, including _NSLockError, [NSException raise], and objc_exception_throw.
Those are not breakpoints. Two of them are functions and -[NSException raise] is a method.

Did you mean you set breakpoints on those functions and that method?

I assume that using the "Release" configuration prevents setting of any breakpoints--
No.

The configurations are build configurations. They affect how Xcode builds your applications.

Breakpoints are not part of the build; you set them in the debugger. They only exist, only get hit, and only stop your program when you run your program under the debugger.

Since they aren't part of the build, it's not possible to pass your breakpoints to a user simply by giving them the app bundle.

I am not sure exactly how breakpoints work …
When your program hits the breakpoint, the debugger breaks (interrupts) your program, whereupon you can examine the program's state and step carefully forward to see how the program goes wrong.

Since it's the debugger that stops your program, breakpoints have no effect when you're not running your program under the debugger.

… or whether the program needs to be run from within gdb for breakpoints to have any effect.
It does. Debugger breakpoints only work within the debugger.

My questions are: could my having left the breakpoints set be the cause of the crashes observed by the user?
No.

First, as noted, even if these breakpoints did somehow get carried over to the user's system, breakpoints are only effective in the debugger. The debugger can't stop on a breakpoint if your program isn't running under the debugger. The user almost certainly isn't running your app under the debugger, especially since they got a crash log out of it.

Even if they did run your app under the debugger with all of these breakpoints set, a breakpoint is only hit when your program reaches that point, so one of these breakpoints could only fire if you or Cocoa called _NSLockError, -[NSException raise], or objc_exception_throw. Getting to that point wouldn't be the cause of the problem, it'd be a symptom of the problem.

And if you did crash as a result of one of those being called, your crash log would have at least one of them named in it. It doesn't.

So, this wasn't related to your breakpoints (different machine, debugger not involved), and it wasn't a Cocoa exception—as I mentioned, Cocoa exceptions are one cause of SIGTRAPs, but they are not the only one. You encountered a different one.

If not, has anybody else had similar problems with [NSFont fontWithName:size:]?
There's no way we can tell whether any problems we've had are similar because you cut off the crash log. We know nothing about what context the crash happened in.

The only thing that's good to cut out is the “Binary images” section, since we don't have your dSYM bundles, which means we can't use that section to symbolicate the crash log.

You, on the other hand, can. I wrote an app for this purpose; feed the crash log to it, and it should detect the dSYM bundle automatically (you are keeping the dSYM bundle for every Release build you distribute, right?) and restore your function and method names into the stack trace wherever your functions and methods appear.

For further information, see the Xcode Debugging Guide.
