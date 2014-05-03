# Lexing
Lexing or tokenizing is a just a process which takes a string as input, divides
the string onto tokens, and returns them, it can also identify invalid tokens.

In URY, variables are defined as follows

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
