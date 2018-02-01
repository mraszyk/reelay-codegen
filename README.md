# reelay

`reelay` is an experimental package to generate sequential machines from formal specifications such as regular expressions.

# install

This package requires `python3`,  `antlr4-python3-runtime`, and `pandas`.

The easiest way to install `reelay` is to run the following command (that requires `git` installed in your system). 

    pip install git+https://github.com/doganulus/reelay.git

# usage

It is an experimental package. We examine several uses of sequential machines generated by formal specifications.

## Pattern matching over text based on sequential machines

`reelay` has a command-line utility of `re2cpp` to generate a `c++` program that matches a regular expression over text.

reelay [-h] [--with-re2] PATTERN

Pattern Syntax:     
  
    Expr = a                   (Letter)
         : Expr1 | Expr2       (Union)
         : Expr1 ; Expr2       (Concatenation)
         : Expr1 *             (Zero-or-more Repetition)
         : Expr1 +             (One-or-more Repetition)
         : Expr1 ?             (Zero-or-one Repetition)

Optional Arguments:

    -h, --help  show this help message and exit
    --with-re2  generate equivalent code using Google's RE2 to match

### example

The installation will provide a command-line script. Then we execute `reelay` for a user-provided regular expression as follows:

    re2cpp '(((a;b)|b)*);b;a'

It generates a simple `C++` program called `matcher-reelay.cpp` to be compiled any `C++`. For example,

    g++ -O2 -o matcher-reelay matcher-reelay.cpp

Now we can check whether a string (given in a file) matches the regular expression.

    ./matcher-reelay textfile.txt

The option `--with-re2` would generate similar `matcher-re2.cpp` and `matcher-re2-nfa.cpp` programs that uses Google's RE2 engine for comparison purposes.

There are still rough edges inside such as the memory trick to force RE2 to generate an NFA. Be aware :)

## Pattern matching over pandas data frames

Consider a pandas DataFrame with Boolean columns. 

    import numpy as np
    import pandas as pd
    from reelay.pandas import regexp

    d = {
         'p1': [False, True,  False,  True,  False,  False, True,  True, True, True], 
         'p2': [False,  False, True, False, True, False,  False, False, True, True]
        }
    df = pd.DataFrame(data=d)

Similar to pandas' arithmetic expression functionality over data frames, we would like evaluate regular expression such as  

    expression = 'p1;p2;p1'

which species a pattern of `p1`, `p2`, `p1` holds sequentially. The operator `;` denotes concatenation. All other regular operators described above are supported.

We evaluate this expression over a data frame as follows:

    df = regexp.eval(df, expression)

The resulting frame would the following data.

     [False, False, False, True, False, False, False, False, False, True]

Optionally, you can give a name to the expression

    named_expression = 'out = p1;p2;p1'

and can evaluate it in place as follows.

    df = regexp.eval(df, named_expression, inplace=True)

          p1     p2    out
    0  False  False  False
    1   True  False  False
    2  False   True  False
    3   True  False   True
    4  False   True  False
    5  False  False  False
    6   True  False  False
    7   True  False  False
    8   True   True  False
    9   True   True   True

Now you can see that `out` is `True` at a position if there exists a sequence of `p1` is True, then `p2` is True, and then `p1` is True. Thus we found the endpoint of the pattern given. This behavior is standard for pattern matchers.

Moreover, you can define custom predicate over Boolean and numerical variables. For that you need to pass your predicate as a local dictionary:

    def my_predicate(a, b):
        return (a or b)    

    df = regexp.eval(df, 'my_predicate(p1, p2); p2', inplace=True, local_dict={'my_predicate' : my_predicate})

This regular expression matches a sequence where `p1` or `p2` holds and then `p2` hold at the next step.


