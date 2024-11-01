# TCL Parser

In the process of writing a TCL parser in C. Method inspired from "[Regular Expression Matching: the Virtual Machine Approach](https://swtch.com/~rsc/regexp/regexp2.html) ([archived](http://archive.is/jQdWg))".

## Grammar syntax and logic

Mostly follows BNF and regexp conventions with some additions:

* ```!``` postfix, backtrack before the element if anything after fails

The terminals and non terminals can return success, fail or error. An error means the grammar can not parse the input, where as a fail means it still possible to try alternate paths in the grammar.

For grammar ```X = A B C | D E```:

* on ```A``` fail, try from ```D E```.
* on ```B``` fail, throw ```error```.
* on ```D``` fail, return ```fail```.
* on ```E``` fail, throw ```error```.

For grammar like ```X = A! B | A C```:

* on ```A``` success, ```B``` fail, backtrack, try from ```A C```.

For grammar ```X = A? B```:

* on ```A``` success and ```B``` fail, throw ```error```.


## The Grammar:
```
vidn = [_a-zA-Z0-9]+
varray = '(' [^)]* ')'
vstr = '{' [^}]* '}'

var = '$'! (vidn varray? | varray | vstr)
cmd = '[' main? ']'

sesc = '\\'! [^\r\n]
qesc = '\\' (eol | any)
besc = '\\' [{}]

srsqr = @depth0 ']'
schr = [^\s\t\r\n;\]]+
bchr = [^{}]+
qchr = [^"]+

sstr = (var|cmd|sesc|schr|srsqr)+
bstr = '{' (besc|bchr|bstr)* '}'
qstr = '"' (var|cmd|qesc|qchr)* '"'

word = sstr|bstr|qstr

eol = '\r'? '\n'
spc = ([\s\t] | '\\'! eol)+
sep = ';' | eol
ws = ([\s\t;] | eol)*

stmt = word (spc word?)*
cmnt = '#' [^\r\n]*

main= ws? (cmnt|stmt)? (sep ws? (cmnt|stmt)?)* '\\'?

```

(grammar
    (main (opt ws) (opt (or cmt stmt)) (many0 sep (opt ws) (opt (or cmnt stmt))))
    (cmnt "#" (many0 (nset "\r\n")))
    )
    
(ws (cmnt (stmt )))


OPT
OR
stmnt
cmnt
OPT
ws

1 2 + 3 *

ws = [\s\t\r\n]*
str = '"' [^"]* '"'
brkt = '{' main '}'
stmt = (str ws (str | brkt))
main = ws (stmt ws)*




(grammar
    (main (and ws (many1 (and stmt ws))))
    (stmt (and str ws (or str brkt)))
    (brkt (and "{" main "}")
    (str (and "\"" (many0 (nset "\"")) "\""))
    (ws (set "\s\t\r\n"))
    )

## Compiled representation

TODO ...

```
MAIN:
    call WS
    call CMNT
    js A
    call STMT
.A:
    call SEP
    jf B
    call WS
    call CMNT
    js A
    call STMT
    jmp A
.B
    parse '\\'
    jmp SUCCESS

    
CMNT:
    char '#'
    jf FAIL
.A:
    nchar '\n'
    js A
    jmp SUCCESS

STMT:
    call WORD
    jf FAIL
.A:
    call SPC
    jf SUCCESS
    call WORD
    jmp A

WS:
    char '\s'
    js WS
    char '\t'
    js WS
    char ';'
    js WS
    call eol
    js WS
    jmp SUCCESS
    
EOL:
    char '\r'
    jf A
    char '\n'
    jf ERROR
.A
    char '\n'
    jf FAIL
    jmp SUCCESS
    
SEP:
    char ';'
    js SUCCESS
    call eol
    js SUCCESS
    jmp FAIL

spc = ([\s\t]|'\\'! eol)+

SPC0:
    char '\s'
    js SUCCESS
    char '\t
    js SUCCESS
    char '\\''
    jf FAIL
    call eol
.A
    char '\n'
    jf FAIL
    jmp SUCCESS
    
SPC:
    call SPC0
    jf FAIL
.A:
    call SPC
    js A
    jmp success


WS0:
    char '\s'
    js SUCCESS
    char '\t'
    js SUCCESS
    char ';'
    jf SUCCESS
    char '\r'
    js SUCCESS
    char '\n'
    js SUCCESS
END:
    success
    
WSS0:
    call WS
    jf FAIL
WSS1:
    call WS
    js WSS1
    success
    


```