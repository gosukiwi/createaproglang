# Introduction

Hello there! I'm Federico Ramirez, a web developer from Argentina. In this book
I'll do my best to help you create a toy programming language and allow you
easily find more resources on your own so you can improve our little language or
create your own.

I'm by no means a professional when it comes to developing programming
languages, nevertheless I've played around with them for a while, I've created
some toy languages and taken some courses on the matter. As a starting
developer I found it quite hard to find information on the matter, this is a
topic which contains so much theory! Unless you are taking a college class it's
hard to get started on your own. I want to change that.

This book is designed to be pragmatic, meaning it aims to get right into coding
and specifically at getting results. It assumes basic programming knowledge from
the reader and it _heavily_ skips the theory part of the cration of programming
languages, the only theory I find hard to skip is the basics of grammars, but
don't worry! It will be short and painless.

This book will give you a general overview of the development of a compiler,
you are encouraged to dive deeper onto each step of the process.

In this book we'll use Javascript, if you don't know Javascript don't worry,
just check out [Eloquent Javascript](http://eloquentjavascript.net/) and come
back to this book, I'll wait, don't worry! We will just use vanilla Javascript
so no need to learn jQuery, or any other library. If you know Javascript feel
free to translate the code examples to other languages, I'd love to see your
work! This book was created to help all developers ease the steep learning curve
this topic has, it would be a huge help if the code examples where available in
more languages.

## Getting our environment ready

As we'll be using Javascript to create our little language we need a way to
execute Javascript code locally. For that we'll use Node. If you are using Linux
it's as easy as using your package manager, for example Ubuntu/Debian users can
just run

    sudo apt-get install node

Easy peasy! If you really want to though, you can install it from source, which
isn't hard at all, but it can be daunting to some users. If you do choose to
install from source it's recommended to install it in your home directory so you
don't need to use `sudo` to install npm packages, a lot of people don't like to
give `npm` admin permissions! The following is a generic way to install node in
your user folder which should work in every Linux distro and even on OSX, it
installs into `/home/<user>/.node`

    wget http://nodejs.org/dist/v0.10.28/node-v0.10.28.tar.gz # change this depending on the latest node version!
    tar -xvzf node-v0.10.28.tar.gz
    cd node-v0.10.28
    ./configure --prefix=~/.node && make && make install

And that's it! Now all you have to do is add it to your `PATH`, which you
can easily do by appending this to your `.profile` or `.bashrc` file

    export PATH=$HOME/.node/bin:$PATH

If you have a Mac and don't want to install from source you can simply use
[Homebrew](http://brew.sh/) and just run

    brew install node
    
On windows this gets a bit messy, but it's still quite easy, as with all Window
apps just download the [Node installer](http://nodejs.org/download/) 
and execute it. Alternatively you can use [Chocolatey](https://chocolatey.org/) and 
run

    cinst node

And that's it! The method you choose is up to you.

Once you have node we'll need a text editor, which I assume you have already, 
but if you don't I recommend [Sublime Text](http://www.sublimetext.com/),
it's potent, works nicely out of the box, has a bunch of plugins and it's easy 
to configure, it even works on Windows, Linux and Mac!

An alternative to Sublime, which is really similar but at the time of writing
this in beta, is [Atom](https://atom.io/).

Personally I use [Vim](http://www.vim.org/), if you are up for a challenge I 
suggest checking it out! It's worth every second you spend learning it.

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
a toy language called URYB which has a syntax similar to Ruby. All the code
we'll write will be in Javascript, as it's the most popular language in Github
and I want a lot of people to understand this!

The source code for this book can be found at
[https://github.com/gosukiwi/creatingaproglang-src](GitHub), just clone it if
you get stuck in any step of the process to easily see the full source code used
in this book and compare it with yours, or use it as a base for a project.
