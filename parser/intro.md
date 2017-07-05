# Chapter 1: you're not a grown kid.

Our objective here is to emit machine code from text, then it's safe to assume
we will address problems like the following:

 + Make sense out of text;
 + Analyse wrong grammar and/or unknown syntax;
 + Generate clear error messages;
 + Emit optimized machine code;

And other problems that we might encounter as we go on.

## Item1: Transforming text into useful information

How can we read from a string and transform whatever inside it into
something we can manage later? There are different ways to achieve
this, and [LR parser](https://en.wikipedia.org/wiki/LR_parser) is
the one we will be using due to its simplicity. If you want further reading about parsers,
[this](https://en.wikipedia.org/wiki/Category:Parsing_algorithms) might help you.

Long story short: LR parsers read input from left
to right, [bottom-up](https://en.wikipedia.org/wiki/Bottom-up_parsing). It means
that this kind of parser works in linear time, as it's going to read byte by byte, like
an iterator, and small expressions are parsed before bigger ones (hence, bottom-up). This way,
we can write rules like operator precedence (which determines how the expression will be
executed later).

To start with some code, let's make a simple program to parse four basic operations:
addition, substraction, multiplication and division. It will generate a tree of expressions,
then execute it.

Our design will be as follows: a function will take an array, and will try to parse it (e.g.,
parse an operation). If something ever goes wrong, we can report the error right from that
function, because, by that point, it's easy to figure out what happened. When the function
makes sure it parsed it all correct, it returns a struct containing the exact parsed piece
of string, and a pointer to the end of that. Why, you ask? Because we need to know
where the expression ends, so other functions can start from there.

Of course, we need grammar, so this is how our expressions will be evaluated:

    expression
      '-'? term 
      expression '-' term
      expression '+' term

    term
      factor
      term '*' factor
      term '/' factor

    factor
      number
      '(' expression ')'

    number
      [0-9]+

Let's examine it from the bottom up (because it's easier to reason about):

+ A `factor` may match a `number` (a sequence of digits), or an `expression`
  inside parentheses.
+ A `term` matches either a `factor`, or a `term` following either a '*' or '/' and a `factor`.
+ An `expression` matches either a `term` with an optional negative sign '-',
  or an `expression` following either an addition, or subtraction sign, and a `term`.

Okay, this is cool and all, but what are we achieving with all that strange syntax?
Actually, nothing, because we didn't write any code yet. However, if
you see the big picture, we've just set a rule of precedence! `factors`
can only exist inside `terms`, and `terms` can only exist inside `expressions`.
If the parser receives something like this:

    42 - 6 * 12

We should expect to see it parsed like this:

    expression

Heh, of course, first it's expected to see an expression. But what does an
expression tell us? Which one of the possible matches can we go with?

Well, the best match for `42 - 6` seems to be the second one:

    expression '-' term

Because there's a subtraction sign and a number before it, the
first match `'-'? term` wouldn't make sense. But the second match
fits it perfectly. Now, the next step is to expand that first `expression`.
Notice how we're going one match by one, this is how the theory works,
you match one grammar at a time, then the final result should the
the best match. Anyway, using `expression` to match `42`, we have
no other choice other than `'-'? term`, because term can go
alone (without the negative sign) in the first place, we are left
with just `term`:

    term '-' term

Now, matching the first `term`, there's also no other choice
better than `factor`, because the other last two options
don't fit the `42` text.

    factor '-' term

You got the idea. Let's just go on and expand the rest:

    number '-' term
    number '-' (term '*' factor)
    number '-' (factor '*' factor)
    number '-' (number '*' factor)
    number '-' (number '*' number)

Now we're getting somewhere. Finally, after parsing everything, we get:

    '42' '-' ('6' '*' '12')

The final result is `42 - (6 * 12)`. And that's exactly what we
want: multiplication and division with higher precedence.
They are evaluated before addition and subtraction. If we first had
the multiplication to happen before subtraction, we'd end up with
something like this:

    ('42' '*' '6') '-' '12'

In other words: `(42 * 6) - 12`.

But... What really is precedence, after all? Probably you've heard about it
in your math classes. It defines when each operation will be evaluated.
For example, `3 * 2 + 2` will result in `8`, but `3 + 2 * 2` results in `7`, and
not `12`. That's because multiplications occur first. So `3 + 2 * 2` is the same as
`3 + (2 * 2)`, and `3 * 2 + 2` is exactly `(3 * 2) + 2`.

+  TODO
