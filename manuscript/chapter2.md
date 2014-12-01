# Lexing
As stated before, lexing is the first step we go though when creating a
compiler, lexing or tokenizing is a just a process which takes a string as
input, divides the string onto chunks called tokens, which are just valid words
in the language we want to create. The tokenizer then returns all tokens as an
array, it's important to note that it must also identify invalid tokens and
throw meaningful errors.

The language we'll create is called URYB and it's pretty similar to Ruby. In 
URYB, variables are defined as follows:

    name = "Mike"

So if we pass that input to the tokenizer, we could divide the string onto
the following tokens:

![How a lexer works](images/ch1-lexer.png)

As you can see, spaces are omitted, but every other string/word must be a valid
token. This is a pretty common practice for tokenizers as we only care about
significant tokens, comments could also be omitted if you so please. The
tokenizer is quite easy to do on your own using regular expressions, or you can
also use a library like [Lex](http://dinosaur.compilertools.net/#lex), some
libraries even take care of lexing __and__ parsing! Isn't that great?

In this book I'll use both approaches eventually, but first I'll do everything
from scratch, as it's a pretty good exercise to understand how everything fits
together.

## What we'll build
Before jumping into our lovely text editor and start pumping code like a mad
scientist, let's see how URYB looks like

```ruby
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
```

If we were to represent the input as tokens it'd look something like this

    IDENTIFIER EQUALS NUMBER

    DEF IDENTIFIER OPEN_PARENS IDENTIFIER COMMA IDENTIFIER CLOSE_PARENS
    RETURN IDENTIFIER OPERATOR IDENTIFIER
    END

    IF IDENTIFIER IS NUMBER
    IDENTIFIER STRING
    END

    WHILE IDENTIFIER OPERATOR NUMBER
    IDENTIFIER EQUALS IDENTIFIER OPERATOR NUMBER
    END

Note that spaces and comments are omitted. The names are irrelevant, you can 
name your tokens whatever you want.

We can easily match some tokens by just string comparisons, such as `def`, `if`
and `end`, nevertheless we'll need something a bit more powerful for identifiers
and strings! In URYB an identifier is a secuence of letters and numbers with no
space, it must start with a letter and it can contain underscores, the following
are valid identifiers

    myVar
    var1
    person_name

The following though are _invalid_ identifiers

    1var # must start with letter
    my var # cannot contain space

This definition of identifier is just something I came up with and it's pretty
common across the most popular programming languages, especially those which
syntax is derived from C, you can of course choose any rules to define your
identifiers, for example, you can let identifiers start with a number! That's
the beauty of creating your very own language! Remember though that changing
something just because you can it's not always a good idea since users are
forced to learn new syntax and rules. That's why a lot of languages stick with a
syntax similar to C or another popular language (such as Ruby or Python). On the
other hand a different syntax might be what you are looking for if your language
is designed to work in a domain-specific environment, for example a [Shading
Language](http://en.wikipedia.org/wiki/Shading_language).

In URYB, Strings are defined as anyting enclosed by quotes. Quotes can be
escaped by using the `\` character. the following are valid strings

    "I'm a simple string"

    "I'm a
    multiline string"

    "My name is \"Mike\""

A [regular expression](http://en.wikipedia.org/wiki/Regular_expression) will do
just fine to match identifier and strings. If you don't know what regular
expressions are you can check out [MDN's documentation on Regular
Expressions](https://developer.mozilla.org/en/docs/Web/JavaScript/Guide/Regular_Expressions),
if you want to play around with them you can do so in the browser using
[Debuggex](https://www.debuggex.com/) or [Rubular](http://rubular.com). As a
quick summary, regular expressions allow us to accept certain inputs and reject
others in a very compact and standarized way.

Now that we have an idea on what tokens we are looking for and how to match them
let's get our hands dirty and create a tokenizer for URYB! Go ahead and create a
file named `tokenizer.js` somewhere in your file system, personally I'm working
on `<user>/node/URY`, and because source code normally resides in a folder named
`src` I created such folder and put inside our `tokenizer.js` file, where you
create the file is completely up to you but if you stick to the conventions used
in this book it will be easier for you to follow along later on.

Remember all the code in this book can be found in the [Github
repository](https://github.com/gosukiwi/creatingaproglang-src), so if you get a
weird error just download the code and do a diff, or just use my code! The idea
is to give you a reference to check any possible errors against. You will notice
the repository also has a folder named `test`, because we are good developers
we'll write unit tests for our code, which will live there. We'll use
[Mocha](http://visionmedia.github.io/mocha/) as our testing library, as it's
lightweight and quite versatile, it allows you to use any assert library you
choose, we'll just use Node's `assert` for this book.

To get Mocha just run

    npm install -g mocha

And we're good to go! Let's start off by writing our tests first, create a file
`test/tokenizer.js` and write a simple test case inside.

    /* global describe, it */
    'use strict';

    describe('Tokenizer', function () {
      var assert    = require('assert');
      var Tokenizer = require('../src/tokenizer.js');
      var tokenizer = new Tokenizer();

      it('should tokenize identifiers', function () {
        assert.deepEqual([
          { 'NAME': 'IDENTIFIER', 'VALUE': 'name' },
          { 'NAME': 'EQUAL', 'VALUE': '=' },
          { 'NAME': 'STRING', 'VALUE': 'Mike' }
          ], tokenizer.tokenize('name = "Mike"'));
      });
    });

You can run the test with the command `mocha test/`, which will of course fail
as we haven't done anything yet! Okay, let's now make our test pass. First of
all, let's make a Token constructor function, each token will be represented by
a Token object.

    'use strict';

    /**
     * Constructor function for the Token object.
     */
    function Token(name, regex, filter) {
        this.name = name;
        this.regex = regex;
        this.filter = filter;

        // the length of the string this token consumes of the input string
        // this is not the same as this.text.length as it applies to the match
        // of the regular expression, the text can be filtered using this.filter
        // thus changing the actual length of the match
        this.length = 0;

        // the matched text of this token once isMatched is called, with filter
        // applied if defined
        this.text = '';
    }

Inside our function we define some attributes all tokens must have. The name
attribute is easy enough to deduce, it's just the name of the token we are
storing, for example, a string, a number, a `def`, etc. But what about the
`regex` and `filter` attributes? Well each token is in charge of matching itself
against an input string, it does so using a regular expression, if the token
matches against a string it then adds the result to the text property. 

But what if we want to make some changes to the matched text? For example, all
strings are enclosed in quotes, we don't want to save the quotes, just the text
inside, we also want to cast numbers to Javascript numbers, not just use them as
strings. For this, we can define a filter function, if existant, we'll apply it
to the result of the regular expression match before saving it tothe text
property.

Let's define the `isMatch` function, which will compare our token against an
string.

    /**
     * Does the input string match this token?
     */
    Token.prototype.isMatch = function (str) {
      var match = str.match(this.regex);

      if(match) {
        this.text = this.filter ? this.filter(match[0]) : match[0];
        this.length = match[0].length;
        return true;
      }

      return false;
    };

As you can see, we match our token against the string using a regular expression
defined in the constructor, if it matches we apply a filter if defined or else
just save the text as it is into the `text` property of the token, finally we
return true. If the token does not match, we return false.

The next function is a simple one, just transform this token into a plain
Javascript object, we'll need this later on.

    /**
     * Return a plain javascript representation of this token
     */
    Token.prototype.plain = function () {
      return {
        'NAME': this.name,
        'VALUE': this.text
      };
    };

That's it for the Token object. Next, let's create a tokenizer, this object has
a bunch of tokens, and it parses a string against all of them until if finds a
match, if it doesn't find a match, just errors out.

When a match is found, it saves the length of the string the match consumes so
it can omit the first part of the input string and try again, matching against
all tokens. This is repeated until the input string has been completely
consumed.

    /* Tokenizer constructor function */
    function Tokenizer() {
        this.tokens = [
            // a string
            new Token('STRING', '^"(?:[^\\"]|\\.)*"', function (str) {
                return str.substr(1, str.length - 2);
            }),
            // an identifier
            new Token('IDENTIFIER', '^[a-za-z][a-za-z0-9_]*'),
            // an equal
            new Token('EQUAL', '^='),
        ];

        // these tokens are matched but are not added to our output
        this.ignoredTokens = [
            new Token('SPACE', '^ '),
            new Token('TAB', '^\t'),
            new Token('RETURN', '^\r'),
            // a single line comment
            new Token('COMMENT', '^#[^\n]*'),
        ];
    }

    Tokenizer.prototype.tokenize = function (input) {
        var start  = 0;
        var length = input.length;
        var output = [];

        while(start < length) {
            var str     = input.substr(start);
            var matched = false;

            // for each token defined create a regular expression for the token
            for(var idx in this.tokens) {
                var token = this.tokens[idx];
                if(token.isMatch(str)) {
                    output.push(token.plain());
                    start += token.length;

                    // turn on the matched flag
                    matched = true;

                    // we already found the token! stop trying
                    break;
                }
            }

            // we didn't match any token... try the ignored ones
            if(!matched) {
                for(idx in this.ignoredTokens) {
                    var ignoredToken = this.ignoredTokens[idx];
                    if(ignoredToken.isMatch(str)) {
                        start += ignoredToken.length;
                        matched = true;
                        break;
                    }
                }
            }

            // we still haven't matched any? it's an error!
            if(!matched) {
                throw 'Could not match token for input ' + str;
            }
        }

        return output;
    };

    module.exports = Tokenizer;

That was a lot of code! But don't worry, it's not that complex, the important
part is we defined our Tokenizer and can easily add any tokens we need, if we
run the tests now by doing `mocha test/` it should pass.

Let's now add a bit more complexity, we are missing a lot of tokens we said we'd
add to our language! Let's write some tests. Inside our tokenizer tests let's
add a few more cases:

    it('should tokenize numbers', function () {
      assert.deepEqual([
        { 'NAME': 'IDENTIFIER', 'VALUE': 'name' },
        { 'NAME': 'EQUAL', 'VALUE': '=' },
        { 'NAME': 'NUMBER', 'VALUE': 11 }
        ], tokenizer.tokenize('name = 11'));

      assert.deepEqual([
        { 'NAME': 'IDENTIFIER', 'VALUE': 'name' },
        { 'NAME': 'EQUAL', 'VALUE': '=' },
        { 'NAME': 'NUMBER', 'VALUE': 3.14 }
        ], tokenizer.tokenize('name = 3.14'));
    });

    it('should tokenize keywords', function () {
      assert.deepEqual([
        { 'NAME': 'IF', 'VALUE': 'if' },
        { 'NAME': 'IDENTIFIER', 'VALUE': 'a' },
        { 'NAME': 'EQUALEQUAL', 'VALUE': '==' },
        { 'NAME': 'IDENTIFIER', 'VALUE': 'b' },
        { 'NAME': 'NEWLINE', 'VALUE': '\n' },
        { 'NAME': 'IDENTIFIER', 'VALUE': 'print' },
        { 'NAME': 'PARENS_OPEN', 'VALUE': '(' },
        { 'NAME': 'STRING', 'VALUE': 'they are equal!' },
        { 'NAME': 'PARENS_CLOSE', 'VALUE': ')' },
        { 'NAME': 'NEWLINE', 'VALUE': '\n' },
        { 'NAME': 'END', 'VALUE': 'end' },
        { 'NAME': 'NEWLINE', 'VALUE': '\n' },
        ], tokenizer.tokenize('if a == b\nprint("they are equal!")\nend\n'));

      assert.deepEqual([
        { 'NAME': 'WHILE', 'VALUE': 'while' },
        { 'NAME': 'TRUE', 'VALUE': 'true' },
        { 'NAME': 'NEWLINE', 'VALUE': '\n' },
        { 'NAME': 'IDENTIFIER', 'VALUE': 'print' },
        { 'NAME': 'PARENS_OPEN', 'VALUE': '(' },
        { 'NAME': 'STRING', 'VALUE': 'HI!' },
        { 'NAME': 'PARENS_CLOSE', 'VALUE': ')' },
        { 'NAME': 'NEWLINE', 'VALUE': '\n' },
        { 'NAME': 'END', 'VALUE': 'end' },
        ], tokenizer.tokenize('while true\nprint("HI!")\nend'));
    });

We added quite some tests! The first one simply tests for a number token, the
latter tests test the keywords of the language. The input string is valid URYB
code, but it doesn't have to, as long as the tokens are valid the Tokenizer will
generate a token array, it's the parser job to make sure they are in the right
order and actually mean something.

In order to pass these tests, let's add a bit more tokens to our tokenizer:

    /* Tokenizer constructor function */
    function Tokenizer() {
        this.tokens = [
            // keywords...
            new Token('WHILE', '^while'),
            new Token('IF', '^if'),
            new Token('END', '^end'),
            new Token('TRUE', '^true'),
            new Token('FALSE', '^false'),
            // parentheses
            new Token('PARENS_OPEN', '^\\('),
            new Token('PARENS_CLOSE', '^\\)'),
            // a string
            new Token('STRING', '^"(?:[^\\"]|\\.)*"', function (str) {
                return str.substr(1, str.length - 2);
            }),
            // a number
            new Token('NUMBER', '^\\d+(\\.\\d+)?', function (str) {
              // hack to cast to either float or int
              return (+str);
            }),
            // an identifier
            new Token('IDENTIFIER', '^[a-za-z][a-za-z0-9_]*'),
            // a comparison equal
            new Token('EQUALEQUAL', '^=='),
            // an equal
            new Token('EQUAL', '^='),
            // a comma
            new Token('COMMA', '^,'),
            // a new line
            new Token('NEWLINE', '^\n'),
        ];

        // these tokens are matched but are not added to our output
        this.ignoredTokens = [
            new Token('SPACE', '^ '),
            new Token('TAB', '^\t'),
            new Token('RETURN', '^\r'),
            // a single line comment
            new Token('COMMENT', '^#[^\n]*'),
        ];
    }

Phew! Those were some tokens. When adding your own tokens, make sure to test the
regular expression using something like Debuggex or Rubular in order to make
sure it works. If you run the tests now, they should pass, if they don't,
remember you can always check the original source at the GitHub repo.

We now have a pretty basic Tokenizer but it works just fine for us! We can
easily add and remove tokens, so we can improve this later on if we want to.

In the next section we'll look at the next step of the compiler creation process
in which we'll turn a list of tokens into an Abstract Syntax Tree, giving some
meaning to our tokens.
