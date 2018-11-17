---
layout: post
title: recursion() { recursion(); }
category: blogpost
date: 2018-10-30
comments: true
excerpt: You have used recursion. Wanna know what your programming language does under the hood to support it?
---

Any programmer can't imagine life without recursion. Although every recurisive algorithm can be written in an iterative manner, there's a certain beauty to writing recurive code. Easy to use & grasp. However, programming languages haven't always supported recursion. Algol 60 was the first language to support it, mostly due to the genius of Djikstra. I read in a couple of places that Lisp was the first one to implement it. Both Lisp and Algol 60 were developed & released around the same time, so I guess no one really knows.

So before I jump into the nitty-gritty of recursion, here's some things you need to know. 

## The Stack & Heap
A process is an instance of a running program. For operating systems, the process is the 'thing' the OS allocates resources to. Every process has a couple of sections when it runs. The most important ones for today's story are the `Heap` and the `Stack`. The heap is the data structure where the running program stores the global variables, or the variable allocated memory using `malloc()` in C, or the `new` keyword in C++ & Java. The lifetime of the variable is till it gets deallocated or the program ends. The stack is where the local variables are stored. They remain valid till the function is being executed. In most of the languages, the function that is being executed(callee) cannot access the variables of the function that called it (caller).

# Stack operations ?
Think of the stack as a pile of books. You can only access what's at the top. Pretty easy, huh. The stack has to operations, push and pop. the push operation is like putting a new book on the top of the book stack. The pop operation is like taking away the book at the top. Now that you know the basic terminology, the next part should be easy. 

# The stack and heap in the process image
Every process has some memory allocated to it. Memory is space on the RAM. The memory that each process gets, is numbered from `0x0000` to whatever the limit is (which I assume as `0xffff` for my examples below). In reality, the limit depends on things like processor bit width (and possibly more things which I may not know at the moment). I'm just focusing on the text, heap and stack sections. The text section is at the lower addresses. It stores the assembly code that should be executed. The heap starts after the text section. As you allocate memory during runtime, the heap grows upwards. The stack begins at the highest memory address, and grows downwards. This is a bit counter-intuitive, so read this carefully. 

The top of the stack is denoted by the `$sp` register. Think of a register as a variable name in assembly programs. The $sp stores the location of the top of the stack, where the last element was pushed. Again, understand, the $sp stores the address of the top of the stack, not the data itself. This means that all the data stored in memory locations `0xffff` to the value in the `$sp` register is the data on the stack.

A small but necessary detour: most architectures (actually memory) now (and have been since a while) are byte addressible. This means that each memory location stores one byte. So when I say that the memory in my example goes from `0x0000` to `0xffff`, it is 16^5 * 1 byte, aka 1 MB. 

Suppose we want to push 1 byte of data on the stack. In such a case, the stack has to grow. Since the memory is byte addressible, we need one memory location to store the value. Thus, the stack pointer register is updated `$sp = $sp - 1`. Don't get confused here. The stack is growing (in terms of the book example, the pile is growing taller). Remember that the stack here grows downwards. So if you push 4 bytes on the stack, the `$sp` register is updated as `$sp = $sp - 4`. If you want to pop 2 bytes from the stack, you update it as `$sp = $sp + 2`.

## Recursion
Now that you've understood this, let's move on to recursion.

# Activation Record
Every function that you write has an associated activation record. Think of it as a place to store the temporary variables, function arguments, the return address and other stuff that the function may require. The reason some languages support recursion while others do not is because of how the languages store the stack at runtime.  