# Parsing
Parsing or semantic analysis is what gives meaning to a bunch of tokens. What 
does an identifier followed by an equal, followed by a number means (a = 1)? 
Most likely it means an assignment, the output of a parser is a tree, called 
Abstract Syntax Tree.

![How a lexer works](images/ch1-parser.png)

Using the same tokens from the example above, the parser process would look 
somethig like this

    INPUT:
    [('identifier', 'name'), ('equal'), ('string', 'mike')]
    
    OUTPUT:
    [('assign', ('variable', ('identifier', 'name')), ('value', ('string', 'mike')))]

The output is a tree represented as a nested array, it might be hard to see,
but it's just a regular tree, like a genealogical tree!

![How a lexer works](images/ch1-ast.png)

You can see an `assign` node has two children, an identifier and a value, this 
is a syntax rule, which might be defined as follows using the 
[BNF](http://en.wikipedia.org/wiki/Backus%E2%80%93Naur_Form):

    assign ::= identifier equal expression
    expression ::= string | number

BNF is just a way to represent a context-free grammar, we'll talk more about it
later, for now, just know that we can define how the tokens will be organized
by using rules. The left side of the `::=` is the name of the rule, and
the right side is the value, the `|` means _or_, just knowing that we can 
deduce an `assign` rule is composed of an identifier, an equal, and an
expression, which can be either a string or a number.

These rules are really useful when we want to define how our language must
be structured, and we can even throw significant _Syntax Errors_.

## Interpreting / Code Generation
The final part is to work with our AST, here we have two choices, the first one
is generate code in another language (most of the time a lower-level language
but not really mandatory). An example of code generation is the CoffeeScript 
compiler, which takes a CoffeeScript program as input, and outputs a valid 
Javascript program.

Another thing we can do with the AST is interpret it and return a value, this 
is what intepreted languages such as Ruby and PHP do, take for example the 
Python interpreter, it's a C program which takes a Pythong program as input, 
and outputs a value by evaluating that code. 

Creating an interpreter isn't really harder than a compiler, but it does require 
quite a lot of knowledge on low-level programming if you plan on creating a 
"real" interpreted programming language, as most languages require to techniques
such as JIT and custom VMs creation.

## Our first parser

CHECK --v

There are some limitations though, recursive descent parsers can only parse a 
subset of context free grammars, called LL(1) Grammars, they are just grammars 
which limit the format of the rules a bit in order to make it easier to parse. 
According to [Marc Moreno Maza](http://www.csd.uwo.ca/~moreno//CS447/Lectures/Syntax.html/node14.html):

    The first L stands for scanning the input from left to right
    The second L stands for producing a leftmost derivation
    And the 1 stands for using one input symbol of lookahead at each step to make parsing action decision.

    [...] It can be shown that LL(1) grammars are
     * Not ambiguous and
     * Not left-recursive.

 There are rules to convert a CF grammar to a LL grammar, 

As we are not going to get deep into theory, I won't write down a LL(1) 
grammar, I'll just specify the nodes of the AST I want and match them.

END OF CHECK --^

### Meet URY our toy language

The language we'll create looks pretty much like ruby, but simpler. Here's
how it looks like

    # comments use hashes
    # assignment
    a = 2 

    # function definition
    def add(a, b)
        return a + b
    end

    # conditional
    if a == 2
        print "a is 2"
    end

    # while loop
    while a < 5
        a = a + 1
    end

So far we'll only work on this, we'll add more features later on, feel free
to add and change anything you want! That's the point of creating your very
own programming language!

## Our First Parser
Let's get down to bussiness and start writing some code! There are several ways
to write parsers, each has it's own advantages and disadvantages, for our first
parser we'll use the easiest to implement, and also the most commonly used, 
it's called a Recursive Descent Parser, according to Wikipedia:

    In computer science, a recursive descent parser is a kind of top-down
    parser built from a set of mutually recursive procedures (or a 
    non-recursive equivalent) where each such procedure usually implements one
    of the production rules of the grammar. Thus the structure of the resulting
    program closely mirrors that of the grammar it recognizes.

A Recursive Descent Parser is just a bunch of functions which might call 
themselves, and each function is in charge of recognizing a rule in a 
special type of grammar called LL(1). Don't worry much about the grammar for
now.


