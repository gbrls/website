---
title: "Lisp Interpreter 0: What is Lisp?"
date: 2020-03-29T02:56:40-03:00
---

This is the first part of a series of articles that I'll be writing to document
my progress on building a Lisp interpreter.  

I've been interested in learning different programming languages for a while now, I've read the
first chapters of Haskell and Lisp books recently and the one that hooked me on was [Land
of lisp](http://landoflisp.com/). Lisp's simplicity really struck me: from very
simple building blocks you can create anything in such an elegant and simple way.

### A bit of history

Lisp is a programming language that was invented in the late 50's by John McCarthy, in 1960 he published a
[paper](http://www-formal.stanford.edu/jmc/recursive.html) where he defined the
language and wrote a Lisp interpreter in Lisp! It's crazy to think that you can
write a interpreter for Lisp in Lisp in such an small and beautiful way using only simple
operations, it really shows its elegance and power. There's a more [approachable
paper](http://www.paulgraham.com/rootsoflisp.html) by Paul Graham in which he
explains step by step how Lisp was defined in McCarthy's paper, here's an
excerpt from it:
>  "_I wrote this article to help myself understand exactly what McCarthy
> discovered. You don't need to know this stuff to program in Lisp, but it
> should be helpful to anyone who wants to understand the essence of Lisp - both
> in the sense of its origins and its semantic core. The fact that it has such a
> core is one of Lisp's distinguishing features (...)_"

### What is it?

The building blocks of Lisp are cons cells. It is a structure that holds two
values, head and tail or as Lisp calls them: `CAR` and `CDR`. Each of them can
either point to another cons cell or to atoms. Atoms are anything that can't be
divided into smaller parts (as we can with cons cells which can be divided into
two parts) such as: `14`, `hello-there`,
`aux`, `"I'm a string"`.
Bellow you can see some examples on how cons cells can work together:  

> ![image](/img/cons-cell-0.png)  

> _Example 1: List_

> ![image](/img/cons-cell-1.jpg)  


> _Example 2: Nested list_


NIL is a special object which denotes the end of a list. As you can see from the first example we've just created a list, which is the
single most important data structure in Lisp. That's where it's name comes from:
**LIS**t **P**rocessor.

### S-Expressions

Now that we know the basic concepts of the language we are going to learn how to
represent them in Lisp code.  

S-Expressions are how Lisp represent its code and data. Code and data being
represented in the same way is a very important concept to Lisp, it is called
[homoiconicity](https://en.wikipedia.org/wiki/Homoiconicity). They can be in the form of atoms or other s-expressions enclosed by
parenthesis and separated by a whitespace.

Lets see how the examples 1 and 2 can be represented using S-Expressions.  

    (42 69 613)
    (c-major (c e g))

Lisp can look at these two expressions as code or as data. By default Lisp reads
them as if they were code. The first element of an expression, (it's `CAR`)
is interpreted as a function, so `(f a b)` has the same meaning as `f(a, b);`
have in C, for example.

### Basic functions

* `PRINT`: Recieves an expression as its input, prints it, and then returns it.  
* `QUOTE`: Recieves an expression as its input and returns it, without evaluating
it.  

Let's try these two functions, let's say that I want to print the list from the
first example. If I call `(PRINT (32 69 613))` it will throw an error because
Lisp will try to call the function `32` with `69` and `613` as its arguments.
But with I call `(PRINT (QUOTE (32 69 613)))` it will work as I intended,
because `QUOTE` returns `(32 69 613)` and then `PRINT` prints it, so `QUOTE` is
a way to convert code into data.  

* `CONS`: Takes `a` and `b` and returns a cons cell with `a` as it's first part and
`b` as the second.
* `CAR`: Takes a cons cell and returns it's first part.  
* `CDR`: Takes a cons cell and returns it's second part.  
* `CADR`: Same as `CAR(CDR(a))`, `CDDR`: Same as `(CDR(CDR(a)))`, `CADDR`,
  `CADAR`, ...

Examples: 

* `(CONS (QUOTE A) (CONS (QUOTE B) NIL))` returns `(A B)`.  
* Expression to create the nested list from the second image:  
`(CONS (QUOTE C-MAJOR) (CONS (CONS (QUOTE C) (CONS (QUOTE E) (CONS (QUOTE G)
NIL))) NIL))` which returns `(C-MAJOR (C E G))`. 

_Sidenote: You don't have to declare things this way, I just made this way for
illustration purposes, in Lisp you could also just do_ `(QUOTE (C-MAJOR (C E
G)))`.  

There are other important functions that we'll be discussing later, but for the
next article that's enough.

