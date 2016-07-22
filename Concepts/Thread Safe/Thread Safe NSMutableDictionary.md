According to Apple’s notes, the NSMutableDictionary is not safe if it is accessed from multiple threads simultaneously. To tackle this problem, one may choose to use a lock or a serial queue to synchronise access to a NSMutableDictionary. However, these approaches are pretty inefficient, especially in situations where reads happen much more often then writes. If a NSMutableDictionary is not being modified for a period of time, threads are free to access that dictionary simultaneously during that time interval. Following this logic, we can use dispatch_barrier_async and dispatch_sync operations on a parallel queue to implement a much more efficient multi-thread safe NSMutableDictionary.

The idea is to let multiple readers in different threads use dispatch_sync on a parallel queue, so they can access a NSMutableDictionary simultaneously. Then, when they would like to change the NSMutableDictionary, let them act as if there is only one writer writing to the dictionary. This is achieved by using dispatch_barrier_async, which will wait for all previously scheduled readers’ dispatch_sync and the writer’s dispatch_barrier_async operations to finish before executing itself. Also, operations scheduled after a dispatch_barrier_async operation will wait for the barrier operation to finish before executing. For more details, refer to the One Resource, Multiple Readers, and a Single Writer section of this article.

The following is my implementation. I name it GGMutableDictionary.

```
#import <Foundation/Foundation.h>

@interface GGMutableDictionary : NSMutableDictionary

@end
```

```
#import "GGMutableDictionary.h"

@implementation GGMutableDictionary {
    dispatch_queue_t isolationQueue_;
    NSMutableDictionary *storage_;
}

/**
 Private common init steps
 */
- (instancetype)initCommon
{
    self = [super init];
    if (self) {
        isolationQueue_ = dispatch_queue_create([@"GGMutableDictionary Isolation Queue" UTF8String], DISPATCH_QUEUE_CONCURRENT);
    }
    return self;
}

- (instancetype)init
{
    self = [self initCommon];
    if (self) {
        storage_ = [NSMutableDictionary dictionary];
    }
    return self;
}

- (instancetype)initWithCapacity:(NSUInteger)numItems
{
    self = [self initCommon];
    if (self) {
        storage_ = [NSMutableDictionary dictionaryWithCapacity:numItems];
    }
    return self;
}

- (NSDictionary *)initWithContentsOfFile:(NSString *)path
{
    self = [self initCommon];
    if (self) {
        storage_ = [NSMutableDictionary dictionaryWithContentsOfFile:path];
    }
    return self;
}

- (instancetype)initWithCoder:(NSCoder *)aDecoder
{
    self = [self initCommon];
    if (self) {
        storage_ = [[NSMutableDictionary alloc] initWithCoder:aDecoder];
    }
    return self;
}

- (instancetype)initWithObjects:(const id [])objects forKeys:(const id<NSCopying> [])keys count:(NSUInteger)cnt
{
    self = [self initCommon];
    if (self) {
        if (!objects || !keys) {
            [NSException raise:NSInvalidArgumentException format:@"objects and keys cannot be nil"];
        } else {
            for (NSUInteger i = 0; i < cnt; ++i) {
                storage_[keys[i]] = objects[i];
            }
        }
    }
    return self;
}

- (NSUInteger)count
{
    __block NSUInteger count;
    dispatch_sync(isolationQueue_, ^{
        count = storage_.count;
    });
    return count;
}

- (id)objectForKey:(id)aKey
{
    __block id obj;
    dispatch_sync(isolationQueue_, ^{
        obj = storage_[aKey];
    });
    return obj;
}

- (NSEnumerator *)keyEnumerator
{
    __block NSEnumerator *enu;
    dispatch_sync(isolationQueue_, ^{
        enu = [storage_ keyEnumerator];
    });
    return enu;
}

- (void)setObject:(id)anObject forKey:(id<NSCopying>)aKey
{
    aKey = [aKey copyWithZone:NULL];
    dispatch_barrier_async(isolationQueue_, ^{
        storage_[aKey] = anObject;
    });
}

- (void)removeObjectForKey:(id)aKey
{
    dispatch_barrier_async(isolationQueue_, ^{
        [storage_ removeObjectForKey:aKey];
    });
}

@end
```
