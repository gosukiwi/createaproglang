# Lexing
As stated before, lexing is the first step we go though when creating a 
compiler, lexing or tokenizing is a just a process which takes a string as 
input, divides the string onto chunks called tokens, which are just valid words
in our language, the tokenizer then returns all tokens as an array, it's 
important to note that it must also identify invalid tokens and throw 
meaningful errors.

The language we'll create is called URYB and it's pretty similar to Ruby. In 
URYB, variables are defined as follows

    name = "Mike"

So if we pass that input to the tokenizer, we could divide the string onto
the following tokens

![How a lexer works](images/ch1-lexer.png)

As you can see, spaces are omitted, but every other string/word must be a valid
token. This is pretty common for tokenizers as we only care about significant 
tokens, comments could also be omitted if you so please. The tokenizer is quite 
easy to do on your own using regular expressions, or you can also use a library
like [Lex](http://dinosaur.compilertools.net/#lex), some libraries even take
care of lexing __and__ parsing!

In this book I'll use both approaches eventually, but first I'll do everything
from scratch, as it's good to know how stuff actually works!

## What we'll build
Before jumping into our lovely text editor and start pumping code, let's see 
how URY looks like

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
and `end`, nevertheless we'll need something a bit more powerful for 
identifiers and strings! In URY an identifier is a secuence of letters and 
numbers with no space, it must start with a letter and it can contain 
underscores, the following are valid identifiers

    myVar
    var1
    person_name

The following though are __invalid__ identifiers

    1var # must start with letter
    my var # cannot contain space

The definition of identifier is just something I came up with and it's pretty
common across the most popular programming languages, especially those which
syntax is derived from C, you can of course choose any rules to define your
identifiers, for example, you can let identifiers start with a number! That's
the beauty of creating your very own language! Remember though that changing 
something just for the sake of it makes it's not a good idea since users are
forced to learn new syntax and rules. That's why a lot of languages stick with 
a syntax similar to C or another popular language (such as Ruby or Python).

Strings are defined as anyting enclosed by quotes, the following are valid strings

    "I'm a simple string"

    "I'm a
    multiline string"

    "My name is \"Mike\""

A [regular expression](http://en.wikipedia.org/wiki/Regular_expression) will do
just fine to match identifier and strings. If you don't know what regular 
expressions are you can check out [MDN's documentation on Regular Expressions](https://developer.mozilla.org/en/docs/Web/JavaScript/Guide/Regular_Expressions),
if you want to play around with them you can do so in the browser using
[Debuggex](https://www.debuggex.com/).

Now that we have an idea on what tokens we'll look for and how to match them 
let's get our hands dirty and create a tokenizer for URY! I'll create a file
named `tokenizer.js` somewhere in our file system, personally I'm working
on `<user>/node/URY`, and as source code normally resides in a folder named 
`src` I created such folder and put inside our `tokenizer.js` file, where
you create the file is completely up to you but if you stick to the conventions
used in this book it will be easier for you to follow along later on.

All code in this book can be found in the [Github repository](https://github.com/gosukiwi/creatingaproglang-src),
so if you get a weird error just download the code and do a diff, or just
use my code! The idea is to give you a reference to check any possible errors 
against. You will notice the repository also has a folder named `test`, as
good developers we are we'll write unit tests for our code, which will live
there. We'll use [Mocha](http://visionmedia.github.io/mocha/) as our testing 
library, as it's lightweight and quite versatile, it allows you to use any
assert library you choose, we'll just use Node's `assert` for this book.

To get Mocha just run

    npm install -g mocha

And we're good to go! Let's start off by writing our tests first, inside
`test/tokenizer.js` write a simple test case


    /* global describe, it */
    'use strict';

    var assert = require('assert'),
        Tokenizer = require('../src/tokenizer.js'),
        tokenizer = new Tokenizer();

    describe('Tokenizer', function () {
        it('should tokenize identifiers', function () {
            assert.deepEqual([
                { 'NAME': 'IDENTIFIER', 'VALUE': 'name' },
                { 'NAME': 'EQUAL', 'VALUE': '=' },
                { 'NAME': 'STRING', 'VALUE': 'Mike' }
            ], tokenizer.tokenize('name = "Mike"'));
        });
    });

You can run the test with the command `mocha test/`, which will of course fail
as we haven't done anything yet! Let's make this test pass! Write the following
code inside `src/tokenizer.js` 


    'use strict';

    /* A token used by the tokenizer */
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

        // checks whether an string matches this token 
        this.isMatch = function (str) {
            var match = str.match(this.regex);

            if(match) {
                this.text = this.filter ? this.filter(match[0]) : match[0];
                this.length = match[0].length;
                return true;
            }

            return false;
        };

        // returns a plain javascript object representation of
        // this token
        this.plain = function () {
            return {
                'NAME': this.name,
                'VALUE': this.text
            };
        };
    }

    /* Tokenizer object constructor */
    function Tokenizer() {
        this.tokens = [
            // a string
            new Token('STRING', '^"(?:[^\\"]|\\.)*"', function (str) {
                return str.substr(1, str.length - 2);
            }),
            // an identifier
            new Token('IDENTIFIER', '^[a-zA-Z][a-zA-Z0-9_]*'),
            // an equal
            new Token('EQUAL', '^='),
            // a new line
            new Token('NEWLINE', '^\n'),
        ];

        // these tokens are matched but are not added to our output
        this.ignoredTokens = [
            new Token('SPACE', '^ '),
            new Token('TAB', '^\t'),
            new Token('RETURN', '^\r'),
        ];
    }

    Tokenizer.prototype.tokenize = function (input) {
        var start = 0,
            length = input.length,
            output = [];

        while(start < length) {
            var str = input.substr(start),
                matched = false;

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

Code! ...
