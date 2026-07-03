# Parsing and the AST

## What parsing adds

After lexing, the compiler has a sequence of Tokens, but that still is not a program structure yet.

The compiler still needs to know:

- which tokens form a function definition
- which tokens form an `if` statement
- how operator precedence should bind
- where an expression starts and ends

That is the job of parsing.

## What an AST is

An AST, or Abstract Syntax Tree, is the structured skeleton of the program.

For example, a function in the AST is no longer just text. It has explicit fields such as:

- return type
- function name
- parameter list
- function body

Expressions work the same way:

- a binary expression has an operator, a left operand, and a right operand
- an `if` statement has a condition, a then branch, and an else branch

So the AST turns the program from a linear token stream into a hierarchical structure.

## The parser in this project uses recursive descent

The current implementation uses a handwritten recursive-descent parser.

That is a good fit for a first-generation compiler like this because:

- grammar rules map directly to functions
- error reporting stays clear
- syntax extensions are easier to control

In the code, the separation is explicit:

- `parse_program`
- `_parse_top_level_decl`
- `_parse_statement`
- `_parse_if`
- `_parse_while`
- `_parse_for`
- a layered set of expression parsing routines

## Why expressions are parsed in layers

Expressions cannot simply be read left to right because precedence matters.

For example:

```c
a + b * c
```

must be understood as `a + (b * c)`, not `(a + b) * c`.

So the parser splits expression parsing by precedence levels, such as:

- assignment
- logical or
- logical and
- equality
- comparison
- additive
- multiplicative
- unary
- postfix
- primary

This layered structure is one of the standard recursive-descent techniques.

## What high-level structures the current parser supports

From the implementation, the current core syntax includes:

- top-level global variable declarations
- function definitions
- blocks
- `if / else`
- `while`
- `for`
- `return`
- local variable declarations
- expression statements

On the expression side, it supports:

- literals
- names
- unary operators
- binary operators
- assignment
- function calls
- array indexing

## Why "only named functions can be called" matters

In the current implementation, a function call requires the callee to be a name rather than an arbitrary expression.

That means:

- `foo(1, 2)` is supported
- more complex function-pointer-style calls are not

This is not an accident. It is a deliberate boundary on the current grammar and semantic model.

## Why declarators are parsed separately

The parser separates the type specifier from the declarator.

Examples such as:

- `int x`
- `int *p`
- `char s[8]`

share a base type like `int` or `char`, but the declarator still has to express:

- pointer levels
- array size
- the final declared name

So parsing a declaration is more than just recognizing a type keyword.

## What parser errors tell you

When the parser fails, it does not just say "the program is bad". It tends to say things like:

- expected `;`
- expected `)`
- expected a type specifier
- unexpected token at this point in the grammar

That reflects a parser that is moving through explicit grammar expectations, not guessing vaguely.

## Why the AST becomes the main input for later stages

From this point on, later phases mostly stop working directly on the Token stream and start working from the AST.

That is because:

- semantic analysis is easier on a structured tree
- the IR builder is easier to write from statement and expression nodes
- the AST can also feed the higher-level backend path directly

So the AST is one of the most important intermediate representations in the project.

## How to inspect this layer

You can run:

```bash
python3 -m c_core_compiler input.c --emit-ast
```

If lexing is fine but the structural interpretation is wrong, this stage usually exposes the problem most clearly.
