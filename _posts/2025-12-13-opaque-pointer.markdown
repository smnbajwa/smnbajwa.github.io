---
layout: post
title: Discovering opaque pointers
---

### Motivation

Early in my career, I would expose the struct definition to users in the header file:

```C
// foo.h

typedef struct Foo {
    uint8_t x;
    uint8_t y;
} Foo;

void Foo_do_something(Foo *f);
```

Users could use it like so:

```C
// app.c
#include "foo.h"

int main() {
    Foo myFoo = {
        .x = 1,
        .y = 2
    };

    Foo_do_something(&myFoo);
}
```

This approach exposes the internals of `Foo` to the user. If `Foo` had ten members, we wouldn't want users to know about all of them, they might try to access or modify them, potentially breaking the module's invariants. Also, if the struct definition changes, then it could break the user's application.

It's good practice to hide the internal details of a module as much as possible and only expose what the user needs. This results in reusable modules and allows us to make changes to a single module in isolation.

### A Better Way
To accomplish this, we need to hide the struct definition from the user. Let's move it to the source file and try compiling. The compiler will complain about an unknown type used in `foo.h`. A forward declaration is sufficient to fix this:

```C
// foo.h

typedef struct Foo Foo;

void Foo_init();
void Foo_do_something(Foo *f);
```

### Who allocates the memory?
`Foo.c` can allocate memory for a predefined number of `Foo`s but that puts a limit on maximum number of instances allowed at library compile time. Also, unused instances waste memory if they are not used.

The application can also allocate memory for `myFoo`. Because it does not have access to the definition, it does not know how much memory to allocate.

How can we expose the size of our struct without exposing the definition? We could provide an API that returns the size of `Foo`, which the application could call at runtime to allocate memory. In embedded systems, compile-time memory allocation is preferred, so providing a `#define` for the size shall suffice.

```C
// foo.h
#define FOO_SIZE_IN_UINT8   (2)

typedef struct Foo Foo;

Foo *Foo_init(uint8_t *mem);
void Foo_do_something(Foo *f);
```

The application can allocate the required memory as a `uint8_t` array and call the initialization function, which returns a pointer to a `Foo` object. This pointer is the so-called opaque pointer because the application does not know its internals.

#### A Note About Memory Alignment
If `Foo` has members larger than `uint8_t`, the struct will be allocated aligned to the largest member type. But the memory is being allocated as an array of `uint8_t`, which will be aligned to a byte address. This might cause our struct to span more memory boundaries than required.

Memory alignment and structure packing is a fascinating topic, read more about it here:
http://www.catb.org/esr/structure-packing/
```C
// app.c
#include "foo.h"

int main() {
    uint8_t mem[FOO_SIZE_IN_UINT8];

    Foo *myFoo = Foo_init(mem);
    Foo_do_something(myFoo);
}
```

We could also add a check to ensure that the `#define` and the actual size always match:

```C
// Foo.c
#include "foo.h"

typedef struct Foo {
    uint8_t x;
    uint8_t y;
} Foo;

static_assert(sizeof(Foo) == FOO_SIZE_IN_UINT8);
```

To make it even easier for the user, we could provide a function macro:

```C
// foo.h
#define FOO_SIZE_IN_UINT8   (2)
#define FOO_DECLARE(var)    \
    uint8_t var[FOO_SIZE_IN_UINT8]

typedef struct Foo Foo;

Foo *Foo_init(uint8_t *mem);
void Foo_do_something(Foo *f);
```

The application uses it like this:
```C
// app.c
#include "foo.h"

int main() {
    FOO_DECLARE(mem);

    Foo *myFoo = Foo_init(mem);
    Foo_do_something(myFoo);
}
```

### Conclusion
The Opaque Pointer design pattern allows us to hide information and reduces the coupling between the library and the application code. This results in a more modular code at the cost of an increase in complexity.
