# Expression associativity

## Right-to-left associativity:

    (1 << (2 << (3 << 4)))

This one is simpler to parse. Consider the following grammar:

    shift-operator
      integer-constant
      integer-constant '<<' shift-operator

    integer-constant
      [0-9]+

With this the parser would try to match integer-constant, '<<' and then
the recursive shift-operator. Resulting in this tree:

    shift-operator(<<):
      integer-constant(1)
      shift-operator(<<):
        integer-constant(2)
        shift-operator(<<):
          integer-constant(3)
          integer-constant(4)

## Left-to-right associativity:

    (((1 << 2) << 3) << 4)

Now things get a little complicated here. Grammar:

    shift-operator
      integer-constant
      shift-operator '<<' integer-constant

This would result in an inifite recursion if we had followed the previous
parser logic. There are simple solutions, one of them is to parse
those rules as a list of nodes (e.g. const, '<<', const, '<<' ...),
then either build the tree backwards, or keep a state for accumulation.
For example:

    lhs = parse_integer_constant();

    loop
    {
      op = parse_shift_operator();

      if (success(op))
      {
        rhs = parse_integer_constant();
        add_node(to=op, from=lhs);
        add_node(to=op, from=rhs);
        lhs = op; //< next iteration will have lhs as the previous node.
      }
      else
        break; //< end of grammar
    }

    return lhs; //< final expression

This parser would produce the following tree:

    shift-operator(<<):
      shift-operator(<<):
        shift-operator(<<):
            integer-constant(1)
            integer-constant(2)
        integer-constant(3)
      integer-constant(4)

