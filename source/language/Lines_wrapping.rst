Lines wrapping
==============

Any statement that is too long in Pine Script can be placed on a few
lines. Syntactically, **a statement must begin at the beginning of the
line. If it wraps to the next line then the continuation of the
statement must begin with one or several (different from multiple of 4)
spaces**. For example, the expression::

    a = open + high + low + close

may be wrapped as

::

    a = open +
          high +
              low +
                 close

The long ``plot`` call line may be wrapped as

::

    plot(correlation(src, ovr, length),
       color=purple,
       style=area,
       transp=40)

Statements inside user functions also can be wrapped to several lines.
However, since syntactically a local statement must begin with an
indentation (4 spaces or 1 tab) then, when splitting it onto the
following line, the continuation of the statement must start with more
than one indentation (and not equal to multiple of 4 spaces). For
example:

::

    updown(s) =>
        isEqual = s == s[1]
        isGrowing = s > s[1]
        ud = isEqual ?
               0 :
               isGrowing ?
                   (nz(ud[1]) <= 0 ?
                         1 :
                       nz(ud[1])+1) :
                   (nz(ud[1]) >= 0 ?
                       -1 :
                       nz(ud[1])-1)

This rule also applies to comments. Don't use comments combined
with lines wrapping. Following code would NOT compile::

    //@version=2
    study("My Script")
    c = open > close ? red :
      high > high[1] ? lime : // a comment
      low < low[1] ? blue : black
    bgcolor(c)

Compiler fails with an error:
``Add to Chart operation failed, reason: line 3: no viable alternative at input '|E|'``.
To make this pine work, simply remove the ``// a comment`` comment. This
limitation is inconvenient... We hope it could be removed in future Pine
releases.
