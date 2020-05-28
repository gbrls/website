---
title: "myLisp Interpreter 1: Lexer and Parser"
date: 2020-03-30T16:24:43-03:00
---


This is the second part of a series of articles that I'm writing to document by
progress on building a Interpreter for myLisp, a lisp-like language. For the previous post [click
here](/mk-lisp-0).  

###


### What is an interpreter?

An Interpreter is a program that takes souce code as input and executes it. It
is composed of several parts, today we are going to write a lexer and a parser.  

The lexer will read a string of characters and output a vector of tokens, with
some information attached to them like its type. The lexer is important to the
parser because it returns the soure code in a better representation, holding
only information that matters to it, _(this is a very important pattern in
programming, thowing away information that is not important for what we want, by
keeping the data in a more convenient representation)_.  

The parser will take the vector of tokens created by the lexer and create a data
structure _(in our case S-Expressions)_ which can then be executed by another part of the interpreter.

### The interpreter's implementation

I will be writing this interpreter in C. The most important reason for it is that
this is a learning project more than anything else, and in C provides us with
the bare minimum to create what we want.  
It is important to note that there are better ways to implement it, but I don't
like to read other people's code before I try my own way. Maybe in the future when I'm done,
I'll change it.

### Lexer

First, lets define a little macro to help us debug.

```c
#ifdef DEBUG_F

#define DEBUG(fmt, args...)						\
    printf("(%s:%d) " fmt, __FILE__,__LINE__, ##args)

#else

#define DEBUG(f, fmt, args...) /* Do nothing */

#endif

```

Next, we need to create a few data types to store the tokens.
_As you can see our Lisp won't support strings and floats for now_.


```c
enum Token_Type {
    TOKEN_OPEN=0,
    TOKEN_CLOSE=1,
    TOKEN_NUMBER=2,
    TOKEN_SYMBOL=3,
};

/*
** Used to debug the lexer
*/
char* Token_Type_Str[4] = {"(", ")", "NUM", "SYM"};

typedef struct {
	enum Token_Type type;

	union {
		int number;
		char* name;
	}data;

}Token;
```

And then the tokenize fuction, there are a few details about it that need to be discussed.
Usually in Lisp, symbol names can be in some written in some formats, like +1, +$,
10@b.com. But for simplicity's sake we have a different set of rules: They must start
with an alphabetical character and not contain any whitespace or parenthesis.



```c
Token* tokenize(char* in, int* ret_sz) {
	int cursor = 0;
	int sz = 0, alloc=1;
	Token* vec = (Token*) malloc(sizeof(Token));

	while(in[cursor] != '\0') {

		Token cur;

		if(in[cursor]=='(') {
			cur.type = TOKEN_OPEN;
			cursor++;

		} else if(in[cursor]==')') {
			cur.type = TOKEN_CLOSE;
			cursor++;

		} else if(isdigit(in[cursor])) {
			int number;
			sscanf(in+cursor,"%d",&number);


			while(in[cursor] != '\0'
				  && isdigit(in[cursor])) {
				cursor++;
			}

			cur.type = TOKEN_NUMBER;
			cur.data.number = number;
			DEBUG("TOKEN_NUMBER: %d\n", number);

		} else if(isalpha(in[cursor])) {

			char* name = (char*) malloc(MAX_SYM_SZ);
			assert(name!=NULL);
		
			int ptr = 0;

			while(in[cursor] != '('
				  && in[cursor] != ')'
		 		  && in[cursor] != ' '
				  && in[cursor] != '\0')
			  {

				if(isalpha(in[cursor])) in[cursor] = toupper(in[cursor]);
				name[ptr] = in[cursor];
				ptr++, cursor++;
			}

			name[ptr] = '\0';

			cur.type = TOKEN_SYMBOL;
			cur.data.name = name;
			DEBUG("TOKEN_NAME: %s\n", name);

		} else {
			cursor++;
			continue;
		}

		if(alloc <= sz+1) {

			alloc *= 2;
			vec = (Token*) realloc(vec, sizeof(Token)*alloc);
		}

		DEBUG("TOKEN: %s\n", Token_Type_Str[cur.type]);

		vec[sz++] = cur;
	}

	*ret_sz = sz;

	return vec;
}
```

And that's about it for the lexer.

### Representing S-Expressions

Before we start to implement the parser we need to write the data types for it.
But this time it is a little bit different because the parser will generate a
data structure which can be executed. We are going to represent those structures
as lisp S-Expressions.  

First, let's define the type `Lisp_Object`, _(it has nothing to do with OOP's
objects)_, which will represent everything that a cons cell can hold as its car
or cdr.

### Tagged Pointers

There are a few ways that we could choose to represent `Lisp_Object`, we are
going to use tagged pointers, which are common in many Lisp implementations.  

Every variable in C has at least 8 bits, so every valid pointer points to an
address which is a multiple of 8. This means that the least significant bits of
a valid address will aways end with three zeros. So we can use these bits to
store additional information about what the pointer is pointing to. When we need
to deference it we just mask out those three bits.

They are useful because a pointer usually occupies eight bytes and an `int` or
`float` just 4, so we can embed ints and floats into them, all we have to do is to
shift them up three bits. That's a huge improvement over deferencing a pointer
to an integer which is allocated on the heap. Using tagged pointers we can
utilize ints and floats on the stack.


Now, we will define a variable that will represent everything that a cons cell
can hold as its car and cdr: `Lisp_Object`.

```c
typedef uintptr_t Lisp_Object;
#define NIL (Lisp_Object)0;
```

`Lisp_Object` is a tagged pointer, that's why we used `uintptr_t` _(it is
defined be a variable that can store a pointer)_. And now a few functions to
operate on it.


```c
enum Tag {
	TAG_SYMBOL = 0,
	TAG_NUMBER = 1,
	TAG_CONS = 2
	/* ... */
};

Lisp_Object ptr_tag(Lisp_Object obj, enum Tag tag) {
	return obj | (int)tag;
}

Lisp_Object ptr_untag(Lisp_Object obj) {
	return obj & ~((Lisp_Object) 7);
}

int ptr_getTag(Lisp_Object obj) {
	return (int)(obj&7);
}

```
Those are the basic operations that we are going to be doing with them. But now we
need a few helper functions and macros to help us create new objects.

```c
#define OBJ(val, type)							\
	Obj_New_ ## type (val)

Lisp_Object Obj_New_symbol(char* str) {
	char* nstr = (char*) malloc(sizeof(str));
	strcpy(nstr, str);
	Lisp_Object ret = ptr_tag((Lisp_Object)nstr, TAG_SYMBOL);

	return ret;
}

Lisp_Object Obj_New_number(int val) {
	Lisp_Object nval = val;
	nval = val<<3;

	return ptr_tag(nval, TAG_NUMBER);
}

typedef struct {

	Lisp_Object car;
	Lisp_Object cdr;

}Lisp_Cons_Cell;

Lisp_Object fcons(Lisp_Object a, Lisp_Object b) {
	Lisp_Cons_Cell* cell = (Lisp_Cons_Cell*) malloc(sizeof(Lisp_Cons_Cell));
	cell->car = a, cell->cdr = b;

	return ptr_tag((Lisp_Object)&cell, TAG_CONS);
}

```

`OBJ` is a little helper macro do help us create new objects. We use it like
this:   

	Lisp_Object a = OBJ("hello-there", symbol);
	Lisp_Object b = OBJ(42, number);

I didn't write a `Obj_New_cons` because we will be using the fcons function to
create new conses.

### Printing S-Expressions
Before we start the parser itself we need to create a function to print
S-Expressions, in order to debug it. There's an algorithm for it but I won't get
into detail. Here it is:  

```c
void _Lisp_Print(Lisp_Object obj, int head);

void Lisp_Print_cons(Lisp_Object obj, int head) {
	if(head) putchar('(');

	_Lisp_Print(fcar(obj), 1);

	if(ptr_untag(fcdr(obj))==NIL) {
		putchar(')');
	} else if(ptr_getTag(fcdr(obj))!=TAG_CONS) {
		printf(" . ");
		_Lisp_Print(fcdr(obj), 0);
		putchar(')');
	} else {
		putchar(' ');
		_Lisp_Print(fcdr(obj), 0);
	}
}


void _Lisp_Print(Lisp_Object obj, int head) {
	enum Tag tag = ptr_getTag(obj);

	switch(tag) {
		case TAG_NUMBER:
			printf("%d", GET_VAL(obj, number));
			break;
		case TAG_SYMBOL:
			printf("%s", GET_VAL(obj, symbol));
			break;
		case TAG_CONS:
			Lisp_Print_cons(obj, head);
			break;
		default:
			break;
	}
}


void Lisp_Print(Lisp_Object obj) {
	_Lisp_Print(obj, 1);
	putchar('\n');
}

```

### The parser

I really liked implementing a Lisp parser a for another interpreter a few days
ago, the algorithm is so simple!  

If a S-Expression is a list, whe can think of it as a list of atoms and for each
atom we parse it recursively. So, how do we parse a simple list? This is how it
would like in ~~py~~pseudocode:  

	def parse (n):
		obj = get_obj(n)

		if obj == nil:
			return obj

		else:
			return cons(obj, parse(n+1))
			

It doesn't work for every S-Expression *yet*, but the main takeaway for it is that
you cons the current object with the rest of the list recursively. The only
difference between this and a complete parser is that the latter handles
parenthesis by parsing what is inside the current block and consing it with the
rest of the list.

```c
Lisp_Object parse(Token* tokens, int pos, int sz) {
	if(pos==sz) return NIL; /* the stop condition */

	if(tokens[pos].type == TOKEN_OPEN) {

		Lisp_Object car = parse(tokens,pos+1,sz);

		int aux = 1, balance = -1;

		while(balance != 0) { /*  Looking for the matching ) */

			assert(aux+pos < sz);

			if(tokens[pos+aux].type == TOKEN_OPEN)
				balance--;
			if(tokens[pos+aux].type == TOKEN_CLOSE)
				balance ++;

			aux++;
		}

		Lisp_Object cdr = parse(tokens, pos+aux, sz);

		return fcons(car, cdr);

	} else if(tokens[pos].type == TOKEN_CLOSE) {

		return NIL;

	} else if(tokens[pos].type == TOKEN_NUMBER) {

		int n = tokens[pos].data.number;
		Lisp_Object car = OBJ(n, number);

		return fcons(car, parse(tokens, pos+1, sz));

	} else if(tokens[pos].type == TOKEN_SYMBOL) {

		char* str = tokens[pos].data.name;

		Lisp_Object car = OBJ(str, symbol);
		return fcons(car, parse(tokens, pos+1, sz));

	}

	return NIL;
}

```

This code isn't that much different from the pseudocode that I've written
before, the main difference is the parenthesis handling. The code for this
article can be seen
[here](https://github.com/gbrls/lisp-interpreter/blob/a15eb40743e64e9cc60c9e01474050ebf25b59ec/main.c).  

Every cons and symbol in which we allocate are not fred, so our interpreter is
leaking memory. The solution for this is called garbage collection, we are going
to implement it in a latter article.
