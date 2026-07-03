# Semantic Analysis and the Symbol Table

## Why syntax is not enough

A program can be syntactically valid and still be wrong.

For example:

```c
int main() {
    x = 1;
    return 0;
}
```

This may be parsable as syntax, but if `x` was never declared, it is semantically invalid.

That is why semantic analysis exists.

## What semantic analysis checks

Semantic analysis checks whether the program structure is meaningful and legal.

In this project, the main checks include:

- top-level names must not collide
- parameter names must not collide
- local variable names must not collide
- variables must be defined before use
- called functions must exist
- argument counts must match
- global initializers must be constant expressions
- assignment targets must be valid lvalues
- functions must contain an explicit `return`

If these issues were only discovered in the backend, debugging would be much harder.

## What a symbol table is

A symbol table is the compiler's name registry.

It answers questions like:

- does this name exist?
- is it a variable or a function?
- in which scope is it valid?
- if it is a function, how many parameters does it take?

This project uses `SymbolTable` to manage a scope stack. During semantic analysis, the compiler repeatedly:

- enters a scope
- registers names
- looks names up
- exits a scope

## Why scopes must be explicit

Consider:

```c
int main() {
    int x = 1;
    {
        int y = x + 1;
        return y;
    }
}
```

Here:

- `x` is defined in the outer block
- `y` is only valid inside the inner block

Without an explicit scope stack, it is much harder to reason correctly about which names are visible at each point.

That is why semantic analysis pushes and pops scopes as it traverses blocks.

## Why the project collects top-level names first

Before deeply checking function bodies, semantic analysis first scans the top-level declarations.

That has two benefits:

- it detects duplicate top-level names early
- it allows functions to call one another regardless of order

Without that pre-pass, cross-function references would be much more awkward to validate.

## What "assignment target must be valid" means

Not every expression can appear on the left side of `=`.

For example:

- `x = 1` is valid
- `arr[i] = 1` is valid
- `*p = 1` is valid
- `(a + b) = 1` is not

The current implementation treats only three categories as assignable:

- names
- array indexing expressions
- dereference expressions

That is a classic lvalue check.

## Why global initializers are restricted to constants

The current implementation requires global variable initializers to be computable at compile time.

For example:

- `int x = 3 + 4;` is allowed
- `int x = foo();` is not

This is a common and practical staged restriction because it keeps backend handling of globals much simpler.

## Why builtin functions are registered here

The project currently defines one minimal builtin function: `print_int`.

Its purpose is to give example programs a way to produce visible output without requiring a full standard library model first.

Since it should behave like a callable function, the cleanest place to introduce it is the semantic layer's symbol system.

That way later function-call validation can treat it like an ordinary known function.

## Why this stage matters so much

The semantic layer is where many "looks like code, but is not actually legal" cases get caught in a systematic way.

Its value is:

- errors surface earlier
- diagnostics are closer to the programmer's mental model
- later stages can assume the input has already been sanity-checked

That substantially reduces complexity in the IR builder and backend.
