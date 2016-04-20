#### Name

assert - abort the program if assertion is false

#### Synopsis

```
#include <assert.h>
void assert(scalar expression);
```

#### Description

If the macro `NDEBUG` was defined at the moment `<assert.h>` was last included, the macro `assert()` generates no code, and hence does nothing at all. Otherwise, the macro `assert()` prints an error message to standard error and terminates the program by calling `abort(3)` if expression is false (i.e., compares equal to zero).
The purpose of this macro is to help the programmer find bugs in his program. The message **"assertion failed in file foo.c, function do_bar(), line 1287"** is of no help at all to a user.

#### Return Value

No value is returned.

#### Conforming to

POSIX.1-2001, C89, C99. In C89, expression is required to be of type int and undefined behavior results if it is not, but in C99 it may have any scalar type.

#### Bugs

`assert()` is implemented as a macro; if the expression tested has side-effects, program behavior will be different depending on whether NDEBUG is defined. This may create Heisenbugs which go away when debugging is turned on.
