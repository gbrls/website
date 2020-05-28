---
title: "myLisp Interpreter 2: Functions and Eval"
date: 2020-04-20T19:22:06-03:00
draft: true
---


This is the third part of a series of articles that I'm writing to document by
progress on building a Interpreter for myLisp, a lisp-like language. For the previous post [click
here](/mk-lisp-1).  


### myLisp symbols

From the previous articles myLisp's symbols are just strings, but if we write
`(CONS (QUOTE A) nil)` how does de compiler know which piece of code runs when
`CONS` is called? It doesn't know. So we are going to change the symbol
representation from a string to a struct:

```c
typedef struct {
	char* name; // Symbol's name
	Lisp_Object*function; // Builtin function pointer
	Lisp_Object obj; // The object which is stored in the symbol
}Lisp_Symbol;
```
