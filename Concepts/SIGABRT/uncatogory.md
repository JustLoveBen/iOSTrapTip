Okay.  Here's a primer on how all this works:

When a chunk of code gets a bad input or something else is seriously wrong, an "exception" is raised, which causes the currently executing code to screech to a halt.  If there's a piece of code set up to "catch" the exception and recover from the error (and there usually is for small problems), the app will be able to recover itself.  If not (or if the error is considered significant enough), then the app will crash itself.  That's what SIGABRT means: it's "signal abort."

The only reason the error appears to be in your main function is that it's really being triggered somewhere in the system frameworks, probably because you have an interface or configuration file set up incorrectly.  And since the UIApplicationMain function is the fundamental entry point into the system frameworks and you don't have access to the frameworks' code, the error appears to be in main().  Have a look in the Debug navigator and you'll see where the problem is really being triggered—it usually provides a clue about the nature of the problem.

One last thing: exceptions almost always include a written reason for the fatal error.  Have a look in the console output (if you need help getting that to appear, let me know) for a description of the problem and, in some cases, a tip about how to correct it.
