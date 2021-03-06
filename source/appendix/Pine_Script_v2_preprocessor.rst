Pine Script v2 preprocessor
---------------------------

Algorithm of ``@version=2`` Pine Script preprocessor in pseudo-code:

#. Remove comments.
#. Replace ``\r\n`` or ``\r`` with just ``\n``.
#. Add ``\n`` to the end of the text if it's missing.
#. Lines that contain only whitespace replace with just empty strings.
#. Add ``|INDENT|`` tokens. They indicate that statement is in a block of
   code, such as function body, ``if`` or ``for`` body. Every tab or
   four spaces are replaced with token ``|INDENT|``.
#. Add ``|B|`` and ``|E|`` tokens. They indicate line begin and line
   end. Replace empty lines with ``|EMPTY|`` tokens.
#. Join lines that represent one splitted statement.
#. Add code block tokens (``|BEGIN|`` --- beginning of the block,
   ``|END|`` --- end of the block, ``|PE|`` --- possible end of the block).

This is illustrated by example. Initial Pine Script text. Note there is
a comment ``//@version=2`` on the first line, it's a directive that helps
to choose correct Pine Script preprocessor/parser version.

::

    "//@version=2
    study('Preprocessor example')
    fun(x, y) =>
        if close > open // This line has one indent
            x + y // This line has two indents
        else 
            x - y
        // Some whitespace and a comment

    a = sma(close, 10)
    b = fun(a, 123)
    c = security(tickerid, period, b)
    plot(c, title='Out', color=c > c[1] ? lime : red, // This statement will be continued on the next line
         style=linebr, trackprice=true) // It's prefixed with 5 spaces, so it won't be considered as an indent
    alertcondition(c > 100)"

After steps 1), 2), 3) and 4) we have::

    "study('Preprocessor example')
    fun(x, y) =>
        if close > open 
            x + y 
        else 
            x - y
        

    a = sma(close, 10)
    b = fun(a, 123)
    c = security(tickerid, period, b)
    plot(c, title='Out', color=c > c[1] ? lime : red, 
         style=linebr, trackprice=true)
    alertcondition(c > 100)
    "

After step 5). Every 4 spaces has been replaced with ``|INDENT|`` token.
Note that 8 spaces replaced with two ``|INDENT|`` tokens and so on::

    "
    study('Preprocessor example')
    fun(x, y) =>
    |INDENT|if close > open 
    |INDENT||INDENT|x + y 
    |INDENT|else 
    |INDENT||INDENT|x - y
        

    a = sma(close, 10)
    b = fun(a, 123)
    c = security(tickerid, period, b)
    plot(c, title='Out', color=c > c[1] ? lime : red, 
    |INDENT| style=linebr, trackprice=true) 
    alertcondition(c > 100)
    "

After step 6)::

    "|EMPTY|
    |B|study('Preprocessor example')|E|
    |B|fun(x, y) =>|E|
    |B||INDENT|if close > open |E|
    |B||INDENT||INDENT|x + y |E|
    |B||INDENT|else |E|
    |B||INDENT||INDENT|x - y|E|
    |EMPTY|
    |EMPTY|
    |B|a = sma(close, 10)|E|
    |B|b = fun(a, 123)|E|
    |B|c = security(tickerid, period, b)|E|
    |B|plot(c, title='Out', color=c > c[1] ? lime : red, |E|
    |B||INDENT| style=linebr, trackprice=true) |E|
    |B|alertcondition(c > 100)|E|
    |EMPTY|"

After step 7). Note that line with ``plot(c, title=`` has been joined
with the next line::

    "|EMPTY|
    |B|study('Preprocessor example')|E|
    |B|fun(x, y) =>|E|
    |B||INDENT|if close > open |E|
    |B||INDENT||INDENT|x + y |E|
    |B||INDENT|else |E|
    |B||INDENT||INDENT|x - y|E|
    |EMPTY|
    |EMPTY|
    |B|a = sma(close, 10)|E|
    |B|b = fun(a, 123)|E|
    |B|c = security(tickerid, period, b)|E|
    |B|plot(c, title='Out', color=c > c[1] ? lime : red, style=linebr, trackprice=true) |E|
    |EMPTY|
    |B|alertcondition(c > 100)|E|
    |EMPTY|"

After step 8)::

    "|EMPTY|
    |B|study('Preprocessor example')|E|
    |B|fun(x, y) =>|E|
    |BEGIN||B|if close > open |E|
    |BEGIN||B|x + y |E||END||PE|
    |B|else |E|
    |BEGIN||B|x - y|E|
    |EMPTY|
    |EMPTY||END||PE||END||PE|
    |B|a = sma(close, 10)|E|
    |B|b = fun(a, 123)|E|
    |B|c = security(tickerid, period, b)|E|
    |B|plot(c, title='Out', color=c > c[1] ? lime : red, style=linebr, trackprice=true) |E|
    |EMPTY|
    |B|alertcondition(c > 100)|E|
    |EMPTY|"

Done. This text is ready to be processed by Pine Script lexer and
parser. There are lexer and parser grammars for your reference.

After the lexer/parser processing, we'd have an AST::

    "
    (FUN_CALL study (FUN_ARGS 'Preprocessor example'))
    (FUN_DEF fun (FUN_DEF_EXPR (FUN_HEAD x y) (FUN_BODY (FUN_RET (IF_THEN_ELSE (> close open) 
    THEN (FUN_BODY (FUN_RET (+ x y))) 
    ELSE (FUN_BODY (FUN_RET (- x y))))))))
    (VAR_DEF a (FUN_CALL sma (FUN_ARGS close 10)))
    (VAR_DEF b (FUN_CALL fun (FUN_ARGS a 123)))
    (VAR_DEF c (FUN_CALL security (FUN_ARGS tickerid period b)))
    (FUN_CALL plot (FUN_ARGS c (KW_ARG title 'Out') (KW_ARG color (? (> c (SQBR c 1)) lime red)) (KW_ARG style linebr) (KW_ARG trackprice true)))
    (FUN_CALL alertcondition (FUN_ARGS (> c 100)))
    "
