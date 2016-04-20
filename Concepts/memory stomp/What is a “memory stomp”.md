
Memory is "stomped" when a piece of code manipulates memory without realizing that another piece of code is using that memory in a way that conflicts. There are several common ways memory can be stomped.

One is allocating, say, 100 bytes of memory but then storing something past the 100th address. This memory might be used to hold something completely different. This is particularly hard to debug because the problem will appear when something tries to access the victim that was stomped on, and the code that stomped on it may be totally unrelated.

Another is accessing memory after it was freed. The memory may be allocated for another object. Again, the code that shows the problem may be related to the newly-allocated object that got the same address and unrelated to the code that caused the problem.

####Example

1.

Very often it is a buffer overrun; as an example, this code:
```
char buffer[8];
buffer[8] = 'a';
```
will "stomp" on whatever happens to be in the next thing in memory after buffer. Generally speaking, 'stomping' is when memory is written to unintentionally.


2.

Other answers basically are correct, but I would like to give an example.
```
int array[10], i;
for (i = 0; i < 11 ; i++)
    array[i] = 0;
```
This code may lead into infinite loop (or may not lead), because it is undefined behavior.

Very likely variable i in memory is stored just after array. So accessing array[10] actually accesses i in other words it resets loop counter.

I think it is good example that demonstrates memory "stomping".

REF:
- [What is a “memory stomp”?](http://stackoverflow.com/questions/13669329/what-is-a-memory-stomp)
