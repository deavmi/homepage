---
title: "TLang Update: Extern support works! 🎆️"
author: Tristan B. Velloza Kildaire
date: 2023-01-30
---

# What is this?

The idea of the `extern` keyword is that it allows for what is known as late-binding which means that the entity referred to by the symbol (the _name_) will not be done during the T compiler's symbol processing stage but rather left up to the backend compiler to worry about. In this case the backend compiler is the `cc` or C-compiler (currently testing with clang but anyone can be used).

This can be useful because currently there is no way to embed assembly inside of a T program and hence the only way we could interact with system calls would have to be through machine code in the form of an object file that is linked in at a later stage.

## Example

To better explain the idea let's take a look at two programs, one written in T (the first) and then one written in C (the second).

As for the T program what we have below is a simple program which has a `test()` function which for the sake of the debugging facilities of the TLang compiler **will** be called upon program execution (hence no visible `main()`). Inside this function we have a reference to two external symbols, one being a variable (or _`evar`_) called `ctr` of which we increment by 1 and the other being a function (or _`efunc`_) named `doWrite`. We then call this function (currently a standalone function call not within an expression must be embedded in a `discard <expr>` statement).

```d
module simple_extern;

extern efunc uint doWrite(uint fd, ubyte* buffer, uint count);
extern evar int ctr;

void test()
{
    ctr = ctr + 1;

    ubyte* buff;
    discard doWrite(cast(uint)0, buff, cast(uint)1001);
}
```

Then for our C program we actually declare these symbols (the code that will be attached to their symbols during link time), as we can see we define the one as a variable with the initial value of 2 and as for `doWrite(.., .., ..)` it simply calls the `write` function from the `unistd.h` header:

```c
#include<unistd.h>

int ctr = 2;

unsigned int doWrite(unsigned int fd, unsigned char* buffer, unsigned int count)
{
    write(fd, buffer, count+ctr);
}
```

# Taking it for a spin 🚀️

Now the steps performed are the following:

1. Compile the C program to an object file
    * Generate it as a library, this is what the `-c` flag is for - else it would fail trying to link in the linux loader which expects a `main` symbol (to be called by the `_start` symbol)
    * We choose to output the object file to `file_io.o`
    * **Run**: `gcc source/tlang/testing/file_io.c -c -o file_io.o`
2. Compile the T program
    * We specify the T program source file `simple_extern.t`
    * We also specify the `-ll` flag which tells the T compiler to pass the provided object file `file_io.o` to the C compiler to link it with the generated object file resulting from compiling the T-to-C transpiled C program
    * **Run**: `./tlang compile source/tlang/testing/simple_extern.t -sm HASHMAPPER -et true -pg true -ll file_io.o `

> NB*: This code is available on the `compiler_object` branch as of commit [`d548a066a63add60c2d587a8ad42ef04cd685e5c`](/git/tlang/tlang/commit/d548a066a63add60c2d587a8ad42ef04cd685e5c) if you want to test it

## Running and testing

Now we will have a generated object file of which is executable (i.e. contains a `main` function), this would be `./tlang.out`.

Running it with the command below might not show much because it doesn't write anything to file descriptor `0` or _standard output_:

```bash
/tlang.out
```

However the program is most definitely running and furthermore the functions are being called with the correct values and the system call to `write` is being made, we can confirm this by `ptrace`-ing the program as follows:

```bash
strace -e trace=write ./tlang.out
```

This will output:

```
write(0, NULL, 1004)                    = -1 EFAULT (Bad address)
+++ exited with 0 +++
```

This is correct! 

Here are some analysis points:

1. Clang apparently makes any uninitialized pointer (our `ubyte* buffer`), well, initialized to `0` (or `NULL`) so we can understand that that is why the second argument is `NULL`
    * Normally this isn't the case as C compilers don't do that.
2. The first argument is `0` as we passed it that
3. The last argument is `count+ctr` which is 1001 added with whatever `ctr` was
    * If we recall, `ctr` was set to 2 initially
    * Before calling `doWrite(.., .., ..)` we set `ctr = ctr + 1` which would mean `ctr` is 3, therefore `1001+3` is equal to `1004`!
