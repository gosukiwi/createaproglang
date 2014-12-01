# Parsing
Parsing or semantic analysis is what gives meaning to a bunch of tokens. What 
does an identifier followed by an equal, followed by a number means? 

    a = 1

Most likely it means that I'm assigning the value `1` to the variable `a`, we'll
represent this semantic using an Abstract Syntax Tree (AST).

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

Remember that in Computer Science, trees are upside down for some reason, but
it's still a tree!

In that example tree, you can see an `assign` node has two children, an
identifier and a value, which are the tree leaves (because they have no
children), this is a syntax rule, which might be defined as follows using the
[Backus-Naur Form](http://en.wikipedia.org/wiki/Backus%E2%80%93Naur_Form):

    assign ::= identifier equal expression
    expression ::= string | number

BNF is just a way to represent a grammar. I know I said I won't get deep into
theory, and I promise I wont! But let's just take a quick glance at grammars
before we continue writing our parser.

## Gammars
A grammar is a collection or rules which can accept or reject an input string,
pretty much like Regular Expressions did! But they are more powerful. The formal
definition can be found in Wikipedia:

> In formal language theory, a grammar [...] is a set of production rules for
> strings in a formal language. The rules describe how to form strings from the
> language's alphabet that are valid according to the language's syntax.

Grammars are useful because they allow us to easily build a set or rules to
identify a language. An example grammar could be defined as follows:

    start = letters
    letter = "a"
    letters = letter letters | letter

Grammars are pretty recursive, in the example above we match "a", "aa" or
"aaaa". To see it in action you can play around at [PegJS's online
parser](http://pegjs.majda.cz/online).

The example above is not very impressive right? You could do that with a regular
expression after all. Well, try to match `0^n1^n` with a regular expression,
just to be clear, it should:

    match     01
    match     0011
    match     000111
    not match 001
    not match 011

With a grammar this would be

    start = numbers

    zero = "0"
    one  = "1"

    numbers = 
     zero numbers one /
     zero one
    
With a regular expression? You can't. That's because regular expression only
match [regular
languages](http://web.stanford.edu/class/archive/cs/cs103/cs103.1132/lectures/15/Small15.pdf),
we can use them for tokenizers but for parsing we need something more. All we do
in parsing is simulate a grammar, accepting and rejecting strings as a grammar
would do.

If you want to know more about grammars and the theorical aspect of parsing you
can see [Marc Moreno Maza's
document](http://www.csd.uwo.ca/~moreno//CS447/Lectures/Syntax.html/Syntax.html)
on the matter.

## Our First Parser
Before we dive right in, let's see what we'll build. There are several ways to
write parsers, each has it's own advantages and disadvantages, for our first
parser we'll use the easiest to implement, and also the most commonly used, it's
called a Recursive Descent Parser, according to Wikipedia:

 > In computer science, a recursive descent parser is a kind of top-down
 > parser built from a set of mutually recursive procedures (or a 
 > non-recursive equivalent) where each such procedure usually implements one
 > of the production rules of the grammar. Thus the structure of the resulting
 > program closely mirrors that of the grammar it recognizes.

Summing it up, a Recursive Descent Parser is collection of functions which might
call themselves (thus they are _recursive_), and each function is in charge of
recognizing a rule in a that of grammar. 

Let's write some tests first! Right into `test/parser.js`.

    /* global describe, it */
    'use strict';


    describe('Parser', function () {
      var assert    = require('assert');
      var Tokenizer = require('../src/tokenizer.js');
      var tokenizer = new Tokenizer();
      var Parser    = require('../src/parser.js');

      function parse(str) {
        var tokens = tokenizer.tokenize(str);
        var parser = new Parser(tokens);
        return parser.parse();
      }
    });

Before writing any test, let's analyze the semantics of our language. We'll have
the following parts.

 * Atom or Literal: It's a value on it's own, for example, an string or a number
 * Expression: Can be passed as parameter or at the right hand side of an
 Assignment, it's dynamic.
 * Statement: An instruction to be executed, can be a function definition, an
   if, a while, a function call, an assignment, etc.
 * Program: A collection of statements, executed one after another.

Okay, now we know what we are looking for!
