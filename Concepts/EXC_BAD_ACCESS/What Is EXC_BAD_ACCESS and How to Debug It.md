At one point or another, you will run into a crash caused by EXC_BAD_ACCESS. In this quick tip, you will learn what EXC_BAD_ACCESS is and what it is caused by. I will also give you a few tips to fix bugs that are caused by EXC_BAD_ACCESS.

1. What Is EXC_BAD_ACCESS?

Once you comprehend the underlying cause of EXC_BAD_ACCESS, you'll better understand its cryptic name. There's a simple explanation and a more technical explanation. Let's start with the simple explanation first.

Keeping It Simple
Whenever you encounter EXC_BAD_ACCESS, it means that you are sending a message to an object that has already been released. This is the most common scenario, but there are exceptions as we'll discuss in a moment.

What It Really Means
The technical explanation is a bit more complex. In C and Objective-C, you constantly deal with pointers. A pointer is nothing more than a variable that stores the memory address of another variable. When you send a message to an object, the pointer that points to the object you're sending the message to needs to be dereferenced. This means that you take the memory address the pointer is pointing to and access the value of that block of memory.

When that block of memory is no longer mapped for your application or, put differently, that block of memory isn't used for what you think it's used, it's no longer possible to access that chunk of memory. When this happens, the kernel sends an exception (EXC), indicating that your application cannot access that block of memory (BAD ACCESS).

In summary, when you run into EXC_BAD_ACCESS, it means that you try to send a message to a block of memory that can't execute that message.

In some cases, however, EXC_BAD_ACCESS is caused by a corrupt pointer. Whenever your application attempts to dereference a corrupt pointer, an exception is thrown by the kernel.

2. Debugging EXC_BAD_ACCESS

Debugging EXC_BAD_ACCESS can be tricky and frustrating. However, now that EXC_BAD_ACCESS is no longer an enigma for you, it should be less daunting.

The first thing you need to understand is that your application doesn't necessarily crash the moment the block of memory is no longer accessible by your application. That's what often makes debugging EXC_BAD_ACCESS so difficult.

The same is true for corrupt pointers. Your application won't crash because a pointer went corrupt. It also won't crash if you pass a corrupt pointer around in your application. When your application attempts to dereference the corrupt pointer, however, thing go wrong.

Zombies
While zombies have gained in popularity over the past few years, they have been around in Xcode for more than a decade. The name zombie may sound a bit dramatic, but it's actually a great name for the feature that's going to help us debug EXC_BAD_ACCESS. Let me explain how it works.

In Xcode, you can enable zombie objects, which means deallocated objects are kept around as zombies. Put differently, deallocated objects are kept alive for debugging purposes. There's no magic involved. If you send a message to a zombie object, your application will still crash as a result of EXC_BAD_ACCESS.

Why is this useful? What makes EXC_BAD_ACCESS difficult to debug is that you don't know what object your application was trying to access. Zombie objects solve this problem in many cases. By keeping deallocated objects alive, Xcode can tell you what object you were trying to access, making the search for the problem that much easier.

Enabling zombies in Xcode is very easy. Note that this may differ depending on the version of Xcode that you're using. The following approach applies to Xcode 6 and 7. Click the active scheme in the top left and choose Edit Scheme.

Select Run on the left and open the Diagnostics tab at the top. To enable zombie objects, tick the checkbox labeled Enable Zombie Objects.

Enable zombie objects in the current scheme
If you now run into EXC_BAD_ACCESS, the output in Xcode's Console will give you a much better idea of where to start your search. Take a look at the following example output.

1
2015-08-12 06:31:55.501 Debug[2371:1379247] -[ChildViewController respondsToSelector:] message sent to deallocated instance 0x17579780
In the above example, Xcode is telling us that a message of respondsToSelector: was sent to a zombie object. However, the zombie object is no longer an instance of the ChildViewController class. The block of memory that was previously allocated to the ChildViewController instance is no longer mapped for your application. This should give you a pretty good idea of what the root cause of the problem is.

Unfortunately, zombie objects won't be able to save your day for every crash caused by EXC_BAD_ACCESS. If zombie objects don't do the trick, then it's time for some proper analysis.

Analyze
If zombie objects don't solve your problem, then the root cause may be less trivial. In that case, you need to take a closer look at the code that's being executed when your application crashes. This can be cumbersome and time-consuming.

To help you find problems in your code base, you could ask Xcode to analyze your code to help you find problematic areas. Note that Xcode analyzes your project, which means that it will point out every potential problem it encounters.

To tell Xcode to analyze your project, choose Analyze from Xcode's Product menu or press Shift-Command-B. It will take Xcode a few moments, but when it's finished you should see a list of issues in the Issue Navigator on the left. Issues found by the analysis are highlighted in blue.

Issues are highlighted in blue
When you click an issue, Xcode takes you to the block of code that needs your attention. Note that Xcode is only making a suggestion. In some cases, it is possible that the issue is not relevant and doesn't need fixing.

Xcode will tell you exactly what is wrong with your code
If you can't find the bug that's causing EXC_BAD_ACCESS, then it's important to carefully scrutinize every issue Xcode found during the analysis of your project.

Conclusion

EXC_BAD_ACCESS is a common frustration among developers and it is something that is inherent to manual memory management. Issues related to memory management have become less frequent since the introduction of ARC (Automatic Reference Counting), but they have by no means disappeared.
