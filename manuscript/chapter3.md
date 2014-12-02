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

Before writing any test, let's analyze the semantics of our language. We'll have
the following parts.

 * Atom or Literal: It's a value on it's own, for example, an string or a number
 * Expression: Can be passed as parameter or at the right hand side of an
 Assignment, it's dynamic.
 * Statement: An instruction to be executed, can be a function definition, an
   if, a while, a function call, an assignment, etc.
 * Program: A collection of statements, executed one after another.

Okay, now we know what we are looking for! Let's write some tests right into
`test/parser.js`.

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

      describe('Assignments', function () {
        it('it should work with numbers', function () {
          assert.deepEqual([
            {
              NAME: 'ASSIGNMENT',
              LHS: { NAME: 'IDENTIFIER', VALUE: 'a' },
              RHS: { NAME: 'NUMBER', VALUE: 1 },
            }
          ], parse('a = 1'));
        });
      });
    });

In the code above we make a single test, to make sure that the input `a = 1`
gets translated to `{ NAME: 'ASSIGNMENT', LHS: { NAME: 'IDENTIFIER', VALUE: 'a'
}, RHS: { NAME: 'NUMBER', VALUE: 1 } }`. The return value is a node of our AST,
and note that the parser should return an array of nodes, each node, a statement
to be executed. 

To implement our parser we'll use several recursive functions, which look
something like this:

    Parser.prototype.parseAssign = function () {
      var identifier = this.pop('IDENTIFIER');
      this.pop('EQUAL');
      var num = this.pop('NUMBER');
      return { NAME: 'ASSIGNMENT', LHS: identifier, RHS: num };
    };

The `pop` and `peek` functions are helpers we use to remove a token from the
list or just take a look at it without removing it. In the `parseAssign`
function we pop the first token and demand it to be of type `IDENTIFIER`, if
it's not of that type, the pop function will throw an exception. We save the
value returned by this function as we'll need it to return our AST node. Next we
pop an `EQUAL` token, we don't need to store this one. Then we pop a number and
finally we return a the node.

This implementation is quite limited though, note that it only work for numbers,
if we want to assign a string, this would fail.

    a = "Hello, World!"

Still, the test passes, so let's add more tests and refactor.

    it('it should work with strings', function () {
      assert.deepEqual([
        {
          NAME: 'ASSIGNMENT',
          LHS: { NAME: 'IDENTIFIER', VALUE: 'a' },
          RHS: { NAME: 'STRING', VALUE: 'some string' },
        }
      ], parse('a = "some string"'));
    });

It's obvious that we'll need something more generic, let's parse an expression
instead of just a number.

    Parser.prototype.parseAssign = function () {
      var identifier = this.pop('IDENTIFIER');
      this.pop('EQUAL');
      var exp = this.parseExpression();
      return { NAME: 'ASSIGNMENT', LHS: identifier, RHS: exp };
    };

And for the `parseExpression` definition:

    Parser.prototype.parseExpression = function () {
      var first  = this.peek();
      switch(first.NAME) {
        case 'IDENTIFIER':
          return this.parseIdentifier();
        case 'STRING':
          return this.parseString();
        case 'NUMBER':
          return this.parseNumber();
        default:
          throw 'Could not parse expression, invalid token ' + first.NAME;
      }
    };

Okay! So an expression can be an identifier, a string or a number, in that
order. We peek for the first token and then parse accordingly. In this case,
they are all _literals_ or _atoms_, so they are trivial, they only return a
token. Their implementation looks like this:

    Parser.prototype.parseString = function () {
      var result = this.pop('STRING');
      return result;
    };

Feel free to ignore this paragraph, but a nerdy point to note here is that we
are using one token to check which function we must call, this could be referred
as a LL(1) Recursive Descent Parser, if we used two tokens, it would be LL(2).
Generically, we say its a LL(k) parser, for k greater than 0 and integer.

Okay, our tests pass...sort of, we haven't really written our parser boilerplate
yet, but hang on for a while, let me finish this example and I'll show you the
boilerplate code!

Let's say we add function calls to our language, they look like this

  print("I'm an argument")

Now if we want to assign the return value of a function call to a variable, it
would look like this

  number = fibonacci(2)

Remember our parser can assign an expression to the RHS of an assignment, but
there's a problem with our expression parser, both the function call and the
identifier start with the same token, an identifier. Looks like we'll need to
peek even further!

    Parser.prototype.parseExpression = function () {
      var first  = this.peek();
      var second = this.tokens[1];
      switch(first.NAME) {
        case 'IDENTIFIER':
          if(second.NAME === 'PARENS_OPEN') {
            return this.parseFunctionCall();
          }
          return this.parseIdentifier();
        case 'STRING':
          return this.parseString();
        case 'NUMBER':
          return this.parseNumber();
        default:
          throw 'Could not parse expression, invalid token ' + first.NAME;
      }
    };

We use a second token to peek, and now we can differentiate a function call from
an identifier. The more complex the language construct, the more peek tokens
you'll need.

I hope you get the idea, here's the working parser code.

    'use strict';

    /**
     * Main parser, transforms an array of tokens into an AST.
     */
    function Parser(tokens) {
      this.tokens = tokens;
    }

    Parser.prototype.parse = function () {
      var res = [];
      while(this.tokens.length > 0) {
        res.push(this.parseStatement());
      }
      return res;
    };

    // Helper methods
    // ---------------------------------------------------------------------------

    /**
     * Removes the first token from the array and returns it.
     */
    Parser.prototype.pop = function (name) {
      if(name && this.tokens[0].NAME !== name) {
        throw 'Expected ' + name + ', got ' + this.tokens[0].NAME;
      }

      return this.tokens.shift();
    };

    /**
     * Takes a peek at the tokens array and returns the first one without removing
     * it.
     */
    Parser.prototype.peek = function (name) {
      if(name && this.tokens[0].name !== name) {
        throw 'Expected ' + name + ', got ' + this.tokens[0].NAME;
      }

      return this.tokens[0];
    };

    /**
     * Consumes new lines if there
     */
    Parser.prototype.consumeNewlines = function () {
      var token = this.peek();
      while(token && token.NAME === 'NEWLINE') {
        this.pop();
        token = this.peek();
      }
    };

    // Atom parsing
    // ---------------------------------------------------------------------------

    Parser.prototype.parseString = function () {
      var result = this.pop('STRING');
      return result;
    };

    Parser.prototype.parseNumber = function () {
      var result = this.pop('NUMBER');
      return result;
    };

    Parser.prototype.parseIdentifier = function () {
      var result = this.pop('IDENTIFIER');
      return result;
    };

    // Expression parsing
    // ---------------------------------------------------------------------------

    /**
     * Parse a list of arguments a function call can have
     * <expression> <comma> <expression> [...]
     */
    Parser.prototype.parseCallArgumentList = function () {
      var result = [];
      while(this.peek().NAME !== 'PARENS_CLOSE') {
        result.push(this.parseExpression());
        // Are we done yet? no? Consume a comma and wait for more arguments
        if(this.peek().NAME !== 'PARENS_CLOSE') {
          this.pop('COMMA');
        }
      }
      return result;
    };

    /**
     * Parse a function call.
     * <t:identifier>(<argument_list>)<t:newline>
     */
    Parser.prototype.parseFunctionCall = function () {
      var name = this.pop('IDENTIFIER');
      this.pop('PARENS_OPEN');
      var args = this.parseCallArgumentList();
      this.pop('PARENS_CLOSE');
      this.consumeNewlines();
      return { NAME: 'FUNCTION_CALL', FUNCTION_NAME: name, ARGUMENTS: args };
    };

    /**
     * Parses an expression.
     * <function_call>
     * <array_value>
     * <string>
     * <number>
     */
    Parser.prototype.parseExpression = function () {
      var first  = this.peek();
      var second = this.tokens[1];
      switch(first.NAME) {
        case 'IDENTIFIER':
          if(second.NAME === 'PARENS_OPEN') {
            return this.parseFunctionCall();
          }
          return this.parseIdentifier();
        case 'STRING':
          return this.parseString();
        case 'NUMBER':
          return this.parseNumber();
        default:
          throw 'Could not parse expression, invalid token ' + first.NAME;
      }
    };

    // Statements
    // ---------------------------------------------------------------------------

    /**
     * Parse assignment.
     * <t:identifier> <t:equal> <expression>
     */
    Parser.prototype.parseAssign = function () {
      var identifier = this.pop('IDENTIFIER');
      this.pop('EQUAL');
      var exp = this.parseExpression();
      this.consumeNewlines();
      return { NAME: 'ASSIGNMENT', LHS: identifier, RHS: exp };
    };

    /**
     * Parses a statement
     */
    Parser.prototype.parseStatement = function () {
      var first  = this.peek();
      var second = this.tokens[1];
      switch(first.NAME) {
        case 'IDENTIFIER':
          if(second.NAME === 'PARENS_OPEN') {
            return this.parseFunctionCall();
          }
          return this.parseAssign();
        default:
          throw 'Invalid token';
      }
    };

It's a lot, I know, but the idea is still the same, simply use functions which
match grammar rules.
