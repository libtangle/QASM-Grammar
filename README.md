# OPENQASM 2.0 Grammar

> This is the grammar for OPENQASM 2.0, written for Antlr4.
> NOTE: This grammar doesn't have 'include' statements, as that should be done in preprocessing

## Usage

You should have antlr4 installed: [here](http://www.antlr.org/download.html).

You can test the parser and see an AST like so:

```
$ antlr4 ./QASM.g4
$ javac QASM*.java
```

```
$ grun QASM mainprog -gui
```

## Examples

The following parse tree is from the QASM code below:

![Parse Tree](https://raw.github.com/qcgpu/QASM-Grammar/master/examples/antlr4_parse_tree.svg?sanitize=true)

```AS
// Repetition code syndrome measurement
OPENQASM 2.0;
qreg q[3];
qreg a[2];
creg c[3];
creg syn[2];
gate syndrome d1,d2,d3,a1,a2
{
  cx d1,a1; cx d2,a1;
  cx d2,a2; cx d3,a2;
}
x q[0]; // error
barrier q;
syndrome q[0],q[1],q[2],a[0],a[1];
measure a -> syn;
if(syn==1) x q[0];
if(syn==2) x q[2];
if(syn==3) x q[1];
measure q -> c;
```


## BNF Grammar

This grammar is based off the BNF grammar, detailed in [this paper](https://arxiv.org/pdf/1707.03429.pdf):

```bnf
mainprogram: "OPENQASM" real ";" program

program: statement | program statement

statement: decl
  :| gatedecl goplist }
  :| gatedecl }
  :| "opaque" id idlist ";"
  :| "opaque" id "( )" idlist ";"
  :| "opaque" id "(" idlist ")" idlist ";"
  :| qop
  :| "if (" id "==" nninteger ")" qop
  :| "barrier" anylist ";"

decl: "qreg" id [ nninteger ] ";" | "creg" id [ nninteger ] ";"

gatedecl: "gate" id idlist {
  :| "gate" id "( )" idlist {
  :| "gate" id "(" idlist ")" idlist {

goplist: uop
  :| "barrier" idlist ";"
  :| goplist uop
  :| goplist "barrier" idlist ";"

qop: uop
  :| "measure" argument "->" argument ";"
  :| "reset" argument ";"

uop: "U (" explist ")" argument ";"
  :| "CX" argument "," argument ";"
  :| id anylist ";" | id "( )" anylist ";"
  :| id "(" explist ")" anylist ";"

anylist: idlist | mixedlist

idlist: id | idlist "," id

mixedlist: id [ nninteger ] | miedlist "," id
  :| mixedlist "," id [ nninteger ]
  :| idlist "," id [ nninteger ]

argument: id | id [ nninteger ]

explist: exp | explist "," exp

exp: real | nninteger | "pi" | id
  :| exp + exp | exp - exp | exp * exp
  :| exp / exp | -exp | exp ^ exp
  :| "(" exp ")" | unaryop "(" exp ")"

unaryop: "sin" | "cos" | "tan" | "exp" | "ln" | "sqrt"

id        := [a-z][A-Za-z0-9_]*
real      := ([0-9]+\.[0-9]*|[0-9]*\.[0-9]+)([eE][-+]?[0-9]+)?
nninteger := [1-9]+[0-9]*|0
```

## Licence 

Copyright Adam Kelly, MIT License