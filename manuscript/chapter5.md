# Making sense out of our AST
So far so good, we have our pretty and shiny AST, but what do we do with it?
Well we have two choices, the first one is generating code in another language,
targeting a lower-level language most of the time, but it can be anything you
want. An example of code generation is the
[CoffeeScript](http://coffeescript.org/) compiler, which takes a CoffeeScript
program as input, and outputs a valid Javascript program. Another example is
[GCC](https://gcc.gnu.org/), a compiler which takes C code as input and outputs
highly optimized assembly code, this is why C is known as one of the fastests
languages out there, if not the fastest.

The second thing we can do with out AST is interpret it, this is what languages
such as Ruby and PHP do. For example the original Ruby interpreter, it's a C
program which takes a Ruby program as input, and outputs a value by evaluating
that code. The speed at which interpreted language performs computations is
highly dependant on the language it's implemented, that's why most interpreters
out there are in C.

A compiler and an interpreter are two very different beasts, and depending on
the language mission you should choose one or the other. In this book we'll
write a compiler, but it's possible to reuse the topics discussed here as a good
base to write an interpreter, the only thing that changes is what you do with th
AST.

There's something very important to note though, there's an intermediate point
between compiled languages and interpreted languages, which is where Virtual
Machines come along. Java is a good example, the Java compiler takes the Java
source code, and _compiles_ it to Java bytecode, which is another language much
simpler than Java, it's this compiled code the one that gets executed by an
interpreter, in this case a Virtual Machine or VM. Why do this? Well by
interpreting a reduced set of instructions the performance of the language can
increase significantly, also it allows for the generated bytecode to be
platform-agnostic, meaning it can run on any platform as long as that platform
has a working VM.

It's possible to make a language which targets an specific VM, such as the Java
VM or .NET's CLR. That way you take advantage of the huge effort put into those
interpreters to make your code fly! A good example of this is
[JRuby](http://jruby.org/) which enables the Ruby programming language to run in
the JVM, and it's known to be faster than the regular interpreted version.

## Code Generation
I think code generation is a good place to start, let's translate our AST into
Javascript! That way, once we finished, we would have written our very own
compiler!

To generate code we'll use the [Visitor
Pattern](http://www.oodesign.com/visitor-pattern.html), this is a really good
use case for this pattern, although as we are not using a statically typed
language we can't really implement interfaces or follow the OO design as
strictly as Java would, for example, but the overall idea isn't hard at all.

Now there are quite some ways to implement this,
