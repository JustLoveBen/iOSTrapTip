This appendix describes the high-level thread safety of some key frameworks in OS X and iOS. The information in this appendix is subject to change.

Cocoa

Guidelines for using Cocoa from multiple threads include the following:

Immutable objects are generally thread-safe. Once you create them, you can safely pass these objects to and from threads. On the other hand, mutable objects are generally not thread-safe. To use mutable objects in a threaded application, the application must synchronize appropriately. For more information, see Mutable Versus Immutable.
Many objects deemed “thread-unsafe” are only unsafe to use from multiple threads. Many of these objects can be used from any thread as long as it is only one thread at a time. Objects that are specifically restricted to the main thread of an application are called out as such.
The main thread of the application is responsible for handling events. Although the Application Kit continues to work if other threads are involved in the event path, operations can occur out of sequence.
If you want to use a thread to draw to a view, bracket all drawing code between the lockFocusIfCanDraw and unlockFocus methods of NSView.
To use POSIX threads with Cocoa, you must first put Cocoa into multithreaded mode. For more information, see Using POSIX Threads in a Cocoa Application.
Foundation Framework Thread Safety
There is a misconception that the Foundation framework is thread-safe and the Application Kit framework is not. Unfortunately, this is a gross generalization and somewhat misleading. Each framework has areas that are thread-safe and areas that are not thread-safe. The following sections describe the general thread safety of the Foundation framework.

Thread-Safe Classes and Functions

The following classes and functions are generally considered to be thread-safe. You can use the same instance from multiple threads without first acquiring a lock.

NSArray
NSAssertionHandler
NSAttributedString
NSCalendarDate
NSCharacterSet
NSConditionLock
NSConnection
NSData
NSDate
NSDecimal functions
NSDecimalNumber
NSDecimalNumberHandler
NSDeserializer
NSDictionary
NSDistantObject
NSDistributedLock
NSDistributedNotificationCenter
NSException
NSFileManager (in OS X v10.5 and later)
NSHost
NSLock
NSLog/NSLogv
NSMethodSignature
NSNotification
NSNotificationCenter
NSNumber
NSObject
NSPortCoder
NSPortMessage
NSPortNameServer
NSProtocolChecker
NSProxy
NSRecursiveLock
NSSet
NSString
NSThread
NSTimer
NSTimeZone
NSUserDefaults
NSValue
NSXMLParser
Object allocation and retain count functions
Zone and memory functions
Thread-Unsafe Classes

The following classes and functions are generally not thread-safe. In most cases, you can use these classes from any thread as long as you use them from only one thread at a time. Check the class documentation for additional details.

NSArchiver
NSAutoreleasePool
NSBundle
NSCalendar
NSCoder
NSCountedSet
NSDateFormatter
NSEnumerator
NSFileHandle
NSFormatter
NSHashTable functions
NSInvocation
NSJavaSetup functions
NSMapTable functions
NSMutableArray
NSMutableAttributedString
NSMutableCharacterSet
NSMutableData
NSMutableDictionary
NSMutableSet
NSMutableString
NSNotificationQueue
NSNumberFormatter
NSPipe
NSPort
NSProcessInfo
NSRunLoop
NSScanner
NSSerializer
NSTask
NSUnarchiver
NSUndoManager
User name and home directory functions
Note that although NSSerializer, NSArchiver, NSCoder, and NSEnumerator objects are themselves thread-safe, they are listed here because it is not safe to change the data objects wrapped by them while they are in use. For example, in the case of an archiver, it is not safe to change the object graph being archived. For an enumerator, it is not safe for any thread to change the enumerated collection.

Main Thread Only Classes

The following class must be used only from the main thread of an application.

NSAppleScript
Mutable Versus Immutable

Immutable objects are generally thread-safe; once you create them, you can safely pass these objects to and from threads. Of course, when using immutable objects, you still need to remember to use reference counts correctly. If you inappropriately release an object you did not retain, you could cause an exception later.

Mutable objects are generally not thread-safe. To use mutable objects in a threaded application, the application must synchronize access to them using locks. (For more information, see Atomic Operations). In general, the collection classes (for example, NSMutableArray, NSMutableDictionary) are not thread-safe when mutations are concerned. That is, if one or more threads are changing the same array, problems can occur. You must lock around spots where reads and writes occur to assure thread safety.

Even if a method claims to return an immutable object, you should never simply assume the returned object is immutable. Depending on the method implementation, the returned object might be mutable or immutable. For example, a method with the return type of NSString might actually return an NSMutableString due to its implementation. If you want to guarantee that the object you have is immutable, you should make an immutable copy.

Reentrancy

Reentrancy is only possible where operations “call out” to other operations in the same object or on different objects. Retaining and releasing objects is one such “call out” that is sometimes overlooked.

The following table lists the portions of the Foundation framework that are explicitly reentrant. All other classes may or may not be reentrant, or they may be made reentrant in the future. A complete analysis for reentrancy has never been done and this list may not be exhaustive.

Distributed Objects
NSConditionLock
NSDistributedLock
NSLock
NSLog/NSLogv
NSNotificationCenter
NSRecursiveLock
NSRunLoop
NSUserDefaults
Class Initialization

The Objective-C runtime system sends an initialize message to every class object before the class receives any other messages. This gives the class a chance to set up its runtime environment before it’s used. In a multithreaded application, the runtime guarantees that only one thread—the thread that happens to send the first message to the class—executes the initialize method. If a second thread tries to send messages to the class while the first thread is still in the initialize method, the second thread blocks until the initialize method finishes executing. Meanwhile, the first thread can continue to call other methods on the class. The initialize method should not rely on a second thread calling methods of the class; if it does, the two threads become deadlocked.

Due to a bug in OS X version 10.1.x and earlier, a thread could send messages to a class before another thread finished executing that class’s initialize method. The thread could then access values that have not been fully initialized, perhaps crashing the application. If you encounter this problem, you need to either introduce locks to prevent access to the values until after they are initialized or force the class to initialize itself before becoming multithreaded.

Autorelease Pools

Each thread maintains its own stack of NSAutoreleasePool objects. Cocoa expects there to be an autorelease pool always available on the current thread’s stack. If a pool is not available, objects do not get released and you leak memory. An NSAutoreleasePool object is automatically created and destroyed in the main thread of applications based on the Application Kit, but secondary threads (and Foundation-only applications) must create their own before using Cocoa. If your thread is long-lived and potentially generates a lot of autoreleased objects, you should periodically destroy and create autorelease pools (like the Application Kit does on the main thread); otherwise, autoreleased objects accumulate and your memory footprint grows. If your detached thread does not use Cocoa, you do not need to create an autorelease pool.

Run Loops

Every thread has one and only one run loop. Each run loop, and hence each thread, however, has its own set of input modes that determine which input sources are listened to when the run loop is run. The input modes defined in one run loop do not affect the input modes defined in another run loop, even though they may have the same name.

The run loop for the main thread is automatically run if your application is based on the Application Kit, but secondary threads (and Foundation-only applications) must run the run loop themselves. If a detached thread does not enter the run loop, the thread exits as soon as the detached method finishes executing.

Despite some outward appearances, the NSRunLoop class is not thread safe. You should call the instance methods of this class only from the thread that owns it.

Application Kit Framework Thread Safety
The following sections describe the general thread safety of the Application Kit framework.

Thread-Unsafe Classes

The following classes and functions are generally not thread-safe. In most cases, you can use these classes from any thread as long as you use them from only one thread at a time. Check the class documentation for additional details.

NSGraphicsContext. For more information, see NSGraphicsContext Restrictions.
NSImage. For more information, see NSImage Restrictions.
NSResponder
NSWindow and all of its descendants. For more information, see Window Restrictions.
Main Thread Only Classes

The following classes must be used only from the main thread of an application.

NSCell and all of its descendants
NSView and all of its descendants. For more information, see NSView Restrictions.
Window Restrictions

You can create a window on a secondary thread. The Application Kit ensures that the data structures associated with a window are deallocated on the main thread to avoid race conditions. There is some possibility that window objects may leak in an application that deals with a lot of windows concurrently.

You can create a modal window on a secondary thread. The Application Kit blocks the calling secondary thread while the main thread runs the modal loop.

Event Handling Restrictions

The main thread of the application is responsible for handling events. The main thread is the one blocked in the run method of NSApplication, usually invoked in an application’s main function. While the Application Kit continues to work if other threads are involved in the event path, operations can occur out of sequence. For example, if two different threads are responding to key events, the keys could be received out of order. By letting the main thread process events, you achieve a more consistent user experience. Once received, events can be dispatched to secondary threads for further processing if desired.

You can call the postEvent:atStart: method of NSApplication from a secondary thread to post an event to the main thread’s event queue. Order is not guaranteed with respect to user input events, however. The main thread of the application is still responsible for handling events in the event queue.

Drawing Restrictions

The Application Kit is generally thread-safe when drawing with its graphics functions and classes, including the NSBezierPath and NSString classes. Details for using particular classes are described in the following sections. Additional information about drawing and threads is available in Cocoa Drawing Guide.

NSView Restrictions
The NSView class is generally not thread-safe. You should create, destroy, resize, move, and perform other operations on NSView objects only from the main thread of an application. Drawing from secondary threads is thread-safe as long as you bracket drawing calls with calls to lockFocusIfCanDraw and unlockFocus.

If a secondary thread of an application wants to cause portions of the view to be redrawn on the main thread, it must not do so using methods like display, setNeedsDisplay:, setNeedsDisplayInRect:, or setViewsNeedDisplay:. Instead, it should send a message to the main thread or call those methods using the performSelectorOnMainThread:withObject:waitUntilDone: method instead.

The view system’s graphics states (gstates) are per-thread. Using graphics states used to be a way to achieve better drawing performance over a single-threaded application but that is no longer true. Incorrect use of graphics states can actually lead to drawing code that is less efficient than drawing in the main thread.

NSGraphicsContext Restrictions
The NSGraphicsContext class represents the drawing context provided by the underlying graphics system. Each NSGraphicsContext instance holds its own independent graphics state: coordinate system, clipping, current font, and so on. An instance of the class is automatically created on the main thread for each NSWindow instance. If you do any drawing from a secondary thread, a new instance of NSGraphicsContext is created specifically for that thread.

If you do any drawing from a secondary thread, you must flush your drawing calls manually. Cocoa does not automatically update views with content drawn from secondary threads, so you need to call the flushGraphics method of NSGraphicsContext when you finish your drawing. If your application draws content from the main thread only, you do not need to flush your drawing calls.

NSImage Restrictions
One thread can create an NSImage object, draw to the image buffer, and pass it off to the main thread for drawing. The underlying image cache is shared among all threads. For more information about images and how caching works, see Cocoa Drawing Guide.

Core Data Framework
The Core Data framework generally supports threading, although there are some usage caveats that apply. For information on these caveats, see Concurrency with Core Data in Core Data Programming Guide.

Core Foundation

Core Foundation is sufficiently thread-safe that, if you program with care, you should not run into any problems related to competing threads. It is thread-safe in the common cases, such as when you query, retain, release, and pass around immutable objects. Even central shared objects that might be queried from more than one thread are reliably thread-safe.

Like Cocoa, Core Foundation is not thread-safe when it comes to mutations to objects or their contents. For example, modifying a mutable data or mutable array object is not thread-safe, as you might expect, but neither is modifying an object inside of an immutable array. One reason for this is performance, which is critical in these situations. Moreover, it is usually not possible to achieve absolute thread safety at this level. You cannot rule out, for example, indeterminate behavior resulting from retaining an object obtained from a collection. The collection itself might be freed before the call to retain the contained object is made.

In those cases where Core Foundation objects are to be accessed from multiple threads and mutated, your code should protect against simultaneous access by using locks at the access points. For instance, the code that enumerates the objects of a Core Foundation array should use the appropriate locking calls around the enumerating block to protect against someone else mutating the array.
