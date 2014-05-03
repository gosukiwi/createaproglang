# Introduction

Well hello there! And welcome! I'm Federico Ramirez, a web developer from 
Argentina. I'm by no means a professional when it comes to developing 
programming languages, nevertheless I've played around with them for a while, 
I've created some toy languages and taken some courses on the matter.

This book is designed to be pragmatic, meaning it aims to get right into 
coding and specifically at getting results quickly, it assumes basic programming 
knowledge from the reader and it __heavily__ skips the theory part of the
cration of programming languages, the only theory I find hard to skip is the
basics of grammars, but don't worry! It will be short and painless.

This book will give you a general overview of the development of a compiler,
you are encouraged to dive deeper onto each step of the process.

## The steps of writing a compiler

Let's jump right into the subject and talk a bit about compilers. What is a 
compiler? A compiler is just a program which translates the source code of a
programming language onto another programming language. That might sound 
silly if you are new to programming but it's actually quite powerful! If 
compilers didn't exist we would all be writing assembler and worrying about 
platform specific issues!

Compilers gives us a sweet layer of abstraction level, for example when you 
program in Ruby or PHP you don't worry about low level stuff such as memory
allocation and OS specific stuff, this is good for some scenarios such as web 
development, but it might not be so good if you want to write something like a 
database which requires a lot of low-level micro management, most likely you'll
end up using something like C.

Compilers have been around for quite some time, the process of creating a 
compiler used to be hard and lengthy, consuming a lot of resources; Nowadays 
though, it's way easier as there are known tools and algorithms to ease up the
development of a language.

The process of creating a programming language can be decomposed as follows:

  * Lexing or Lexical Analysis
  * Parsing or Semantic Analysis
  * Interpreting / Code Generation

You can of course, create your own process and do as you think it's right, but
this is the way most programming languages are created, and it has quite a lot
of computer science theory behind it.

In the following chapters we'll dive onto each of the steps and implement
a toy language called URY which has a syntax similar to Ruby. All the code
we'll write will be in Javascript and Ruby, as they are the most popular 
languages in Github and I want a lot of people to understand this!

## Installing tools

As we'll be using Javascript and Ruby we need to install them! If you are using
Linux it's as easy as use your package manager, for example Ubuntu/Debian users
can do

    sudo apt-get install node ruby

And we'll be just fine. If you have a Mac you can use Brew to do so.
  
    brew install node ruby
    
On windows this gets trickier, but it's still quite easy, in the traditional
way, just download the setups and run them, or you can use [Chocolatey](https://chocolatey.org/)!
You can now do

    cinst node ruby

And that's it! I assume you have a text editor already, but if you don't I
recommend Textmate on OSX, Gedit on Linux and Notepad++ on Windows. Personally
I use Vim.

