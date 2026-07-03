# Lexing and Tokens

## What lexical analysis is solving

Source code starts as a stream of characters.

The compiler should not try to do all later work directly on raw text, because every later stage would otherwise keep repeating low-level questions such as:

- is this a keyword?
- is this an identifier?
- is this `=` a standalone assignment operator or part of something else?
- where does this string literal end?

Lexical analysis solves that by first turning the source into a sequence of structured units: Tokens.

## A practical way to think about Tokens

A Token is a piece of source text that has already been categorized.

For example:

```c
return a + 1;
```

becomes something roughly like:

- `return`: keyword
- `a`: identifier
- `+`: operator
- `1`: integer literal
- `;`: semicolon

That means the parser does not need to keep guessing what each character slice means.

## The lexer in this project is handwritten

The project does not use a lexer generator. It implements a handwritten `Lexer`.

That choice has clear benefits here:

- the control flow is easy to follow
- error positions are easy to manage
- adding a new token category is direct

Its core loop is basically:

1. skip whitespace and comments
2. inspect the current character
3. choose a scanning branch
4. emit one Token
5. continue until end of input

## What it currently scans

From the current implementation, the lexer supports:

- identifiers
- keywords
- decimal integers
- character literals
- string literals
- single-character operators and delimiters
- double-character operators

It also skips:

- spaces, tabs, and newlines
- `//` line comments
- `/* ... */` block comments

## Why double-character operators must be recognized first

Operators such as:

- `==`
- `!=`
- `<=`
- `>=`
- `&&`
- `||`

must be recognized before the lexer falls back to single-character tokens.

Otherwise `==` would be misread as two separate `=` tokens. So the lexer first checks whether the next two characters form a valid operator before trying the one-character branch.

## Why source positions matter

Each Token in this project records a line and column.

That sounds like a detail, but it matters a lot because later diagnostics depend on it:

- lexical errors need to point to bad characters
- parse errors need to report where expected syntax is missing
- semantic errors also need meaningful locations

If this information is missing early, debugging gets much worse later.

## Why strings and character literals need their own logic

They are not as simple as "read until the next space".

Examples include:

- `'a'`
- `'\n'`
- `"hello"`
- `"a\nb"`

The lexer must deal with:

- opening and closing quotes
- escape sequences
- unterminated literals

So they are scanned by dedicated logic rather than by the general identifier/number flow.

## What this stage does not do

The lexer only cuts the text into meaningful pieces. It does not understand program structure.

So it knows that `if` is a keyword and `(` is a left parenthesis, but it does not know:

- what the full condition of the `if` is
- whether the parentheses are structurally valid
- whether the surrounding expression is legal

Those questions belong to the parser.

## Why the token output is useful

If you run:

```bash
python3 -m c_core_compiler input.c --emit-tokens
```

you can quickly answer questions such as:

- did the lexer recognize the literal correctly?
- was a comment skipped as expected?
- was a double-character operator split incorrectly?

That makes token emission one of the most useful first debugging steps.
