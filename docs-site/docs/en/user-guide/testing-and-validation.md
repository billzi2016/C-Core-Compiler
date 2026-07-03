# Testing and Validation

## Why testing is explained explicitly here

This project is not trying to prove only that "an example sometimes runs". It is trying to lock down the behavior of each compiler stage as the project evolves.

Without layered tests, compiler projects often drift into a bad pattern:

- a new feature is added
- an older behavior changes silently
- the only symptom appears much later in an end-to-end failure

That is why testing here is not a side activity. It is part of the structure.

## What the current tests are validating

From the existing test suite and `TESTING.md`, the current focus includes:

- whether Tokens are correct
- whether AST structures are correct
- whether semantic errors surface early
- whether the IR stays stable
- whether backend output has the expected structure
- whether the CLI behaves correctly
- whether end-to-end compilation can produce real executables

## Why tests are layered

A compiler is not a single-function program. It is a multi-stage pipeline.

Layered tests matter because:

- lexer failures do not need to be discovered only at the backend
- AST mistakes do not need to be inferred from final binary behavior
- IR mistakes can be inspected directly at the lowering level

That dramatically narrows the search space when something breaks.

## How to understand the test layers

### 1. Lexing tests

They validate:

- keywords
- identifiers
- integers
- comments
- whitespace
- operators
- invalid-character handling

### 2. Parsing tests

They validate:

- expression precedence
- function definitions
- declarations
- assignment
- control flow
- function calls

### 3. Semantic tests

They validate:

- duplicate definitions
- undefined symbols
- arity mismatches
- invalid assignment targets

### 4. IR tests

They validate:

- conditional lowering
- loop lowering
- calls and returns
- labels and jump structure

### 5. Backend tests

They validate:

- whether key structures appear in the generated code
- whether labels, locals, and temporaries look reasonable
- whether return paths are complete

### 6. CLI tests

They validate:

- argument parsing
- `--emit-*` inspection output
- basic command-line behavior

### 7. End-to-end tests

They validate:

- whether executables can actually be produced
- whether the final behavior matches expectations

## Why end-to-end tests depend on the system toolchain

Because this project ultimately invokes `clang` or `cc` to produce executables.

So end-to-end tests check whether a usable host compiler exists. If it does not, the tests usually skip only the binary-producing parts instead of failing the entire suite.

## How to run tests

The simplest full command is:

```bash
python3 -m unittest discover -s tests -v
```

If you are only changing one layer, it is still better to narrow your attention:

- when changing the lexer, focus first on token-related tests
- when changing the parser, focus on AST and syntax tests
- when changing the IR builder, focus on IR and end-to-end behavior
- when changing the backend, focus on generated code and end-to-end tests

## Why docs and tests should be read together

This project works especially well if you read documentation and tests side by side:

- after [Lexing and Tokens](../project/lexer-and-tokens.md), inspect the lexer tests
- after [Parsing and the AST](../project/parser-and-ast.md), inspect the parser and AST tests
- after [IR and Lowering](../project/ir-and-lowering.md), inspect the IR tests

That makes it much easier to see that each stage is not only described conceptually, but also constrained by executable checks.
