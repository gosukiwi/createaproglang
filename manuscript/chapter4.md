# Meet URYB our toy language
The language we'll create looks pretty much like ruby, but simpler. Let's call
it _URYB_. Here's how it looks like:

    # comments use hashes
    a = 2 #assignment

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

So far we'll only work on these features, we'll add more features later on, feel
free to change anything you want! That's the point of creating your very own
programming language!
