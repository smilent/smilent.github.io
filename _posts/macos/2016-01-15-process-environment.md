---
layout: post
title: Process Environment
tags:
- UNIX
comments: true
description: "an introduction to UNIX process environment"
keywords: "UNIX, process"
---

* toc
{:toc}
----

This post is a note of APUE.
<br><br>

##main Function

When a C program is executed by the kernel -- by one of the `exec` functions, a special start-up routine is called before the `main` function is called. The executable program file specifies this routine as the starting address for the program; this is set up by the link editor when it is invoked by the C compiler. This routine takes values from the kernel -- the command-line arguments and the environment -- and sets things up.

##Process Termination

There are eight ways for a process to terminate. Normal termination occurs in five ways:

1. Return from `main`
2. Calling `exit`
3. Calling `_exit` or `_Exit`
4. Return of the last thread from its start routine
5. Calling `pthread_exit` from the last thread

Abnormal termination occurs in three ways:

1. Calling `abort`
2. Receipt of a signal
3. Response of the last thread to a cancellation request

##Process Termination

###<code>exit</code>, <code>Exit</code> and <code>_exit</code> Functions

<pre>
#include &lt;stdlib.h&gt;
void exit(int status);
void _Exit(int status);

#include &lt;unistd.h&gt;
void _exit(int status);
</pre> 

The `exit` function has always performed a clean shutdown of the standard I/O library: the `fclose` function is called for all open streams. All three exit functions expect a single integer argument *status*. If (a) any of these functions is called without an exit status, (b) `main` does a `return` without a return value, or, (c) the `main` function is not declared to return an integer, the exit status of the process is undefined. However, if the return type of `main` is an integer and `main` "falls off the end" (an implicit return), the exit status of the process is 0;

Returning an integer value from the `main` function is equivalent to calling `exit` with the same value.

###<code>atexit</code> Function

<pre>
#include &lt;stdlib.h&gt;
int atexit(void (*func)(void));
<div align="right">Returns 0 if OK, nonzero on error</div>
</pre>

We pass the address of a function as the argument to `atexit`. When this function is called, it is not passed any arguments and is not expected to return a value. The `exit` function calls theses functions in reverse order of their registration. Each function is called as many times as it was registered.

![how a c program is started and how it terminates]({{site.baseurl}}/images/how_a_c_program_starts_and_terminates.png)

##Environment List

Each program is also passed an *environment list*. Like the argument list, the environment list is an array of character pointers, with each pointer containing the address of a null-terminated C string. The address of the array of pointers is contained in the global variable *environ*:
<pre>extern char **environ</pre>

##Memory Layout of a C Program

A C program has been composed of the following pieces:

- Text segment, consisting of the machine instructions that the CPU executes. Usually, the text segment is sharable so that only a single copy needs to be in memory for frequently executed programs, such as text editors, the C compiler, the shells, and so on. Also, the text segment is often read-only, to prevent a program from accidentally modifying its instructions.
- Initialized data segment, usually called simply the data segment, containing variables that are specifically initialized in the program. For example, the C declaration 
<pre>int maxcount = 99;</pre>
appearing outside any function causes this variable to be stored in the initialized data segment with its initial value.
- Uninitialized data segment, often called the "bss" segment, named after an ancient assembler operator that stood for "block started by symbol". Data in this segment is initialized by the kernel to arithmetic 0 or null pointers before the program starts executing. The C declaration
<pre>long sum[1000];</pre>
appearing outside any function causes this variable to be stored in the uninitialized data segment.
-Stack, where automatic variables are stored, along with information that is saved each time a function is called. Each time a function is called, the address of where to return to and certain information about the caller's environment, such as some of the machine registers, are saved on the stack. The newly called function them allocates room on the stack for its automatic and temporary variables. This is how recursive functions in C can work.Each time a recursive function calls itself, a new stack frame is used, so one set of variables doesn't interfere with the variables from another instance of the function.
- Heap, where dynamic memory allocation usually takes place. Historically, the heap has been located between the uninitialized data and the stack.

<a id="memoryarrangement"></id>
![typical memory arrangement]({{site.baseurl}}/images/typical_memory_arrangement.png){: width="60%"}

##Memory Allocation

###<code>malloc</code>, <code>calloc</code> and <code>realloc</code> Functions

<pre>
#include &lt;stdlib.h&gt;
void *malloc(size_t size);
void *calloc(size_t nobj, size_t size);
void *realloc(void *ptr, size_t newsize);
<div align="right">All three return: non-null pointer if OK, NULL on error</div>
void free(void *ptr)
</pre>

- `malloc`, which allocates a specified number of bytes of memory. The initial value of the memory is *indeterminate*.
- 'calloc', which allocates space for a specified number of objects of a specified size. The space is initialized to all 0 bits.
- `realloc`, which increases or decreases the size of a previously allocates area. When the size increases, it may involve moving the previously allocates area somewhere else, to provide the additional room at the end. Also, when the size increases, the initial value of the space between the old contents and the end of the new area is indeterminate. As a special case, if *ptr* is as null pointer, `realloc` behaves like `malloc` and allocates a region of the specified `newsize`.

The pointer returned by the three allocation functions is guaranteed to be suitably aligned so that it can be used for any data object. For example, if the most restrictive alignment requirement on a particular system requires that `double`s must start at memory locations that are multiples of 8, them all pointers returned by these three functions would be so aligned.

The function `free` causes the space pointed to by `ptr` to be deallocated. This freed space is usually put into a pool of available memory and can be allocated in a later call to one of the three `alloc` functions.

##Environment Variables

Then environment strings are usually of the form 
<pre>name=value</pre>

The Unix kernel never looks at these strings; their interpretation is up to the various applications. The shells, for example, use numerous environment variables. Some, such as `HOME` and `USER`, are set automaticaly at login; others are left for us to set. We normally set environment variables in a shell start-up file to control the sehll's actions. If we set the environment variable `MAILPATH`, for example, it tells the Bourne shell, GNU Bourne-again shell, and Korn shell where to look for mail. 

ISO C defines a function that we can use to fetch values from the environment, but this standard says that the contents of the environment are implementation defined.

<pre>
#include &lt;stdlib.h&gt;
char *getenv(const char *name);
<div align="right">Returns: pointer to value associated with name, NULL if not found.</div>
</pre>

Note that this function returns a pointer to the *value* of a `name=value` stringa. We should always use `getenv` to fetch a specific value from the environment, instead of accessing *environ* directly.

Here are functions that set an environment variable. But please note that not all systems support this capability.

<pre>
#include &lt;stdlib.h&gt;
int putenv(char *str);
<div align="right">Returns: 0 if OK, nonzero on error.</div>
int setenv(const char *name, const char *value, int rewrite);
int unsetenv(const char *name);
<div align="right">Both return: 0 if OK, -1 on error. </div>
</pre>

The operations of these three functions is as follows:
- The `putenv` function takes a string of the form *name=value* and places it in the environment list. If *name* already exists, its old definition is first removed.

- The `setenv` function sets *name* to *value*. If *name* already exists in the environment, then (a) if *rewrite* is nonzero, the existing definition for *name* is first removed; or (b) if *rewrite* is 0, an existing definition for *name* is not removed, *name* is not set to the new *value*, and no error occurs.

- The `unsetenv` function removes any definition of *name*. It is not an error if such a definition does not exist.


In figure <a href="#memoryarrangement">memory arrangement</a>, we show that the environment strings are typically stored at the top of a process's memory space, above the stack. Thus adding a string or modifying an existing string is difficult. The space at the top cannot be expanded, because it is oftern at the top of the address space of the process and so can't expand upward; it can't be expanded downward, because all the stack frames below it can't be moved.

- If we're modifying an existing *name*:
    - If the size of the new *value* is less than or equal to the size of the existing *value*, we can just copy the new string over the old string.

    - If the size of the new *value* is larger than the old one, however, we must `malloc` to obtain room for the new string, copy the new string to this area, and then replace the old pointer in the environment list for *name* with the pointer to this allocated area.

- If we're adding a new *name*, it's more complicated. First, we have to call `malloc` to allocate room for the *name=value* string and copy the string to this area.
    - Then, if it's the first time we've added a new *name*, we have to call `malloc` to obtain room for a new list of pointers. We copy the old environment list to this new area and store a pointer to the *name=value* string at the end of this list of pointers. We also store a null pointer at the end of this list, of course. Finally, we set *environ* to point to this new list of pointers. Note from figure <a href="#memoryarrangement">memory arrangement</a> that if the original environment list was contained above the top of the stack, as is common, then we have moved this list of pointers to the heap. But most of the pointers in this list stil point to *name=value* strings above the top of the stack.

    - If this isn't the first time we've added new strings to the environment list, then we know that we've already allocated room for the list on the heap, so we just call realloc to allocate room for one more pointer. The pointer to the new *name=value* string is stored at the end of the list (on top of the previous null pointer), followd by a null pointer.


