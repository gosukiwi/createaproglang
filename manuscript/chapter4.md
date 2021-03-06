# Parser Generator: Jison
Jison, as described by the official website:

> Jison takes a context-free grammar as input and outputs a JavaScript file
> capable of parsing the language described by that grammar.

So, there are tools which makes a parser for you and here you are, making it on
your own. Well, you have to crawl before you walk! It's like when you learn
[L'Hôpital](http://en.wikipedia.org/wiki/L%27H%C3%B4pital%27s_rule) in Calculus
(ignore my bad jokes if you've never taken that class before).

Jison is a Javascript port of an old tool for C which does the same called
Bison. Give it a context-free grammar and it will generate a parser for us,
pretty awesome! There are some limitations though, it doesn't parse a grammar if
it has some kind of ambiguity, but we can remove it by using presedence rules or
making some change to our rules.

For now let's see an example Jison grammar so we see the syntax:

    /* lexical grammar, tokenizer */
    %lex

    %%

    /* token definitions, for example: */
    " "+                    /* skip whitespace */
    \d+(\.\d+)?             return 'NUMBER';
    [a-zA-Z][a-zA-Z0-9_]*   return 'IDENTIFIER';
    \"(?:[^\"]|\.)*\"       return 'STRING';

    /* end of tokenizer */
    /lex

    /* define starting rule for our grammar */
    %start Program

    %%

    /* define grammar rules, for now a simple example which only matches numbers */

    Program
        : StatementList EOF
            { return $1 }
        ;

    StatementList
        : Statement StatementList
            { $$ = $1.concat([$2]); }
        | Statement
            { $$ = [$1]; }
        ;

    Statement
        : NUMBER
            { $$ = Number(yytext); }
        ;

As you can see, the format is

    RuleName
        : (TOKEN or RuleName)*
          { return value }
        ;

A rule can have several possible options, for example, we could create a
`Terminal` rule which matches a `STRING` token, a `NUMBER` token or an
`IDENTIFIER` token.

    Terminal
      : STRING
      | NUMBER
      | IDENTIFIER
      ;

Now that we know how to define Jison grammars, let's make a grammar and make all
our old tests from the hand-made grammar pass. We'll build the same AST but for
each node we'll use a `TYPE` property instead of `NAME` as using the latter
might make things a bit confusing sometimes.

You can see the full grammar [at the GitHub
repository](https://github.com/gosukiwi/creatingaproglang-src/blob/v0.3/src/grammar.jison),
in order to transform a grammar into a Javascript file we can use we'll need to
install and run Jison:

    npm install -g jison
    jison grammar.jison

Easy enough! Once we have our `grammar.js` file we can simply include it and use
the parser as needed:
    
    var parser = require('grammar.js').parser;
    var ast = parser.parse('a = 1');

You might notice some strange names in the grammar's return values, these are
predefined Jison variables we use to return and get data.

    $$:     The value of this variable will be the return value of the rule
    $1:     The value of the first match of the rule
    $2:     The value of the second match of the rule
    $n:     The value of the nth match of the rule
    yytext: The full text matched by the rule

Take your time and play around with the grammar, if you want, you can try
writing it yourself or making some toy grammars to get the hang of it. Once you
get used to it, it allows you to make complex parsers really quickly!

## Implementing IF
Let's implement the `IF` statement yet again. The test is the same as our
previous test.

    it('should work with an expression', function () {
      assert.deepEqual([{
        TYPE: 'IF',
        CONDITION: { TYPE: 'NUMBER', VALUE: 1 },
        BODY: [{ TYPE: 'ASSIGNMENT', LHS: { TYPE: 'IDENTIFIER', VALUE: 'a' }, RHS: { TYPE: 'STRING', VALUE: 'hello' } }]
      }], parse('if 1\na = "hello"\nend'));
    });

As for the grammar:

    /* ... */ 

    Statement
        : FunctionDefinition
            { $$ = $1; }
        | FunctionCall
            { $$ = $1; }
        | If
            { $$ = $1; }
        | Assignment
            { $$ = $1; }
        ;

    /* ... */

    If
        : IF Expression NEWLINE OptionalStatementList END
            { $$ = { TYPE: 'IF', CONDITION: $2, BODY: $4 } }
        ;

    /* ... */

It greatly helps we already have defined a lot of rules! This makes the test
pass, but now we'll need to add binary operations! To do this we'll need to
account for operator precedence. We can choose any order we want, for example,
[here's Javascript operator precedence
table](http://msdn.microsoft.com/en-us/library/ie/z3ks45k7(v=vs.94).aspx).

Because we'll compile to Javascript it's useful to have similar precedence. In
Jison we can set precedence using the `%left` rule, meaning the first rule
specified has higher precedence than the ones after. For example

    # given
    a + b * c
    # rules
    %left +
    %left *
    # produces
    (a) + (b * c)

### Our URBY parser
I've worked on our grammar and completed the `If` statement --- which includes
the using `else` and also the `elsif` case. Another thing this version adds is a
couple more of binary operations, namely division, multiplication, substraction
and addition. You can see the source code at the github repo as
[v0.5](https://github.com/gosukiwi/creatingaproglang-src/releases/tag/v0.5).

## Alternatives
Jison isn't the only parser generator out there, for Javascript there's
[PEG.js](http://pegjs.majda.cz/) which doesn't parse context-free grammars but
it parses a subset of them, it parses grammars without left-side recursion,
meaning rules like

  Expression
    : Expression OPERATOR Expression

Wont work, that's because precedence is set by using the Grammar rules in order
to make the grammar not ambiguos.

For other languages there are other alternatives of course, C beeing the oldest
and most solid one, it has [Bison](http://www.gnu.org/software/bison/), Ruby has
[Racc](http://i.loveruby.net/en/projects/racc/doc/usage.html) and Python has
[PLY](http://www.dabeaz.com/ply/). 

Some languages such as PHP don't have any parser generator out there as the time
of this writing, but most languages do, you are just a google search away from
them if you are interested!

## Exercise
If you want, a good exercise is to go ahead and implement the `while` statement,
if you get stuck, I made a
[v0.6](https://github.com/gosukiwi/creatingaproglang-src/releases/tag/v0.6)
which contains the source code to implement this feature.

If you feel particularly adventurous you can try to translate our little grammar
to another tool, such as PEG.js, PLY, or Racc.
