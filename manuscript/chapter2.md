# Lexing
As stated before, lexing is the first step we go though when creating a compiler
, lexing or tokenizing is a just a process which takes a string as input, divides
the string onto tokens, and returns them as an array, it can also identify 
invalid tokens.

The language we'll create is called URY and it's pretty similar to Ruby. In URY,
variables are defined as follows

    name = "Mike"

So if we pass that input to the tokenizer, the process would look something like
this

![How a lexer works](images/ch1-lexer.png)

As you can see, spaces are omitted, this is pretty common for tokenizers as we 
only care about significant tokens, comments could also be omitted if you so 
please. The tokenizer is quite easy to do on your own using regular expressions, 
or you can also use a library, some libraries take care of lexing __and__ 
parsing, so as you'll be using a library for parsing, might as well use it for 
lexing!

In this book I'll use both approaches eventually, but first I'll do everything
from scratch, as it's good to know how stuff actually works!

## Writing a tokenizer for URY

Let's get our hands dirty and create a tokenizer for URY! I'll create a file
named `tokenizer.js` which will contain a function which takes a string as
input and outputs a token array.

First let's see how URY looks like

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

If we were to divide that into tokens, we might get something like this

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
name your tokens whatever you want, the important part is that the program
can be decomposed into tokens.
