Memory Stomp?

Many programmers have to spent several nights debugging hard-to-find bugs related to Memory Stomp. Oh :( that is ridiculous !!

The term memory stomp is used when something changes a value in memory that was not supposed to changes.

In other words, memory is "stomped" when a piece of code manipulates memory without realizing that another piece of code is using that memory in a way that conflicts.

Or, memory stomp will "stomp" on whatever happens to be in the next thing in memory after buffer.

 Memory stomps can occur in several ways.
Buffer overrun.
Accessing memory after it was freed.

Buffer overrun

Say, 8 bytes of memory but then storing something past the 8th address. This memory might be used to hold something completely different. This is particularly hard to debug because the problem will appear when something tries to access the victim that was stomped on, and the code that stomped on it may be totally unrelated.

int carray[8];
for (int count = 0; count <= 8; count++) {
 carray[count] = count;
}

Buffer overrun is tricky to debug.

Because most of the time there might not be anything allocated right after the buffer in memory and so it will not have any noticeable effect or you may end up writing an acceptable range for the variable so there memory stomp focuses.

Accessing memory after it was freed

The memory may be allocated for another object. Again, the code that shows the problem may be related to the newly-allocated object that got the same address and unrelated to the code that caused the problem.
