
# Chapter 1: you're not a grown kid.

First of all, we're implementing the compiler right now.

    But that's nothing to do with learning C!

Actually, you will need to work with a vast set of tools, provided by C and its
standard library. It's okay to assume you might learn by doing contrived/toy code, however,
for a better study, implementing real world code and/or complex algorithms will lead
your brain to a better understanding of programming concepts and paradigms.

Our objective here is to emit machine code from text, then it's safe to assume
we will treat problems like the following ones:

 + Make sense out of text;
 + Analyse wrong grammar and/or unknown syntax;
 + Emit clean error messages;
 + Emit optimized machine code;

And other problems that we might encounter as we go on.

## Item1: Transforming text into useful information

How can we read from a string and transform whatever inside that into
something we can manage later? There are different ways to achieve
this, and for simplicity, [LR parser](https://en.wikipedia.org/wiki/LR_parser) is
the choice. If you want further reading about parsers,
[this](https://en.wikipedia.org/wiki/Category:Parsing_algorithms) might help you.

Long story short: LR parsers read input from left to right, bottom-up. It means
that this kind of parser is linear in time, as it's going to read byte by byte, like
an iterator, and small expressions are parsed before bigger ones (hence, bottom-up). This way,
we can write rules like operator precedence (which determines how the expression will be
executed later).

To start with some code, let's write down a simple program to parse four basic operations:
addition, substraction, multiplication and division. It will generate a tree of expressions,
then execute it.

Our design will be as follows: a function will take an array, and will try to parse it (e.g.,
parse an operation). If something ever goes wrong, we can report the error right from that
function, because, by that point, it's easy to know what happened. When the function
makes sure it parsed all correct, it returns a struct containing the exact parsed piece
of string, and a pointer to the end of that string. Why, you ask? Because we need to know
where the expression ends, so other functions can start from there.

Of course, we need grammar, so this is how our expressions will be evaluated:

    expression := ["-"] term {("+"|"-") term}
    
    term := factor {("*"|"/") factor}
    
    factor := number | "(" expression ")"

Let's examine it from bottom-up:

 + `Factor` is made of a number, or an expression inside parentheses.
 + `Term` is made of a factor alone, or one followed by a
multiplication or division by another factor (and it can go further, forming a list).
 + `Expression` is made of a term (that might, or not, be followed by minus) alone, or one
followed by an addition or subtraction of another term.

`{x}` means a list of zero or more x's, `[x]` means none or one x,
`(x)` means obligatory one x, and `|` means "one of".

Okay, this is cool and all, but what are we achieving with all that strange syntax?
Actually, nothing, because we didn't write any code yet. However, if
you see the big picture, we've just set a rule of precedence! `Factors`
can only exist inside `Terms`, and `Terms` can only exist inside `Expressions`.
If the parser receives something like this:

    42 - 6 * 12

We should expect to see it parsed like this:

    expression

Heh, of course, first it's expected to see an expression. But what does an
expression tells us?

    term ("-" term)

Hm, so the expression turned into two terms with a subtraction operation.
If we go further, those terms get translated to:

    factor ("-" (factor ("*" factor)))

Now we're getting somewhere. Finally, after parsing everything, we get:

    number ("-" (number ("*" number)))

The final result is `42 - (6 * 12)`. And that's exactly what we
desire: multiplication and division with higher precedence.
They are evaluated before addition and subtraction.

```c
// TODO
    
```

## Item2: A better face (or how to make it smile)

## Item3: Code generation


