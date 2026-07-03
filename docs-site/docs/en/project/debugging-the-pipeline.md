# How to Debug the Whole Pipeline

## Why this project is good for stage-by-stage debugging

Many compiler projects are frustrating because all you really see is "the final result worked" or "the final result failed".

One advantage of this project is that it exposes multiple intermediate layers. That lets you ask more precise questions:

- did the lexer split tokens incorrectly?
- did the parser build the wrong structure?
- did the IR lowering go wrong?
- did the backend generate the wrong output?

That is useful both for learning and for debugging real failures.

## The most useful observation points

### `--emit-tokens`

```bash
python3 -m c_core_compiler input.c --emit-tokens
```

Good for checking:

- whether literals were recognized correctly
- whether comments were skipped properly
- whether double-character operators were split incorrectly

### `--emit-ast`

```bash
python3 -m c_core_compiler input.c --emit-ast
```

Good for checking:

- whether expression precedence was parsed correctly
- whether `if / while / for` structures were attached correctly
- whether top-level declarations were classified correctly

### `--emit-ir`

```bash
python3 -m c_core_compiler input.c --emit-ir
```

Good for checking:

- whether control flow was lowered correctly
- whether temporaries make sense
- whether assignments, jumps, and returns look correct

### `--emit-ir-dot`

```bash
python3 -m c_core_compiler input.c --emit-ir-dot
```

Good for checking:

- branch directions
- loop structure
- the overall control-flow graph shape

### `--emit-c`

```bash
python3 -m c_core_compiler input.c --emit-c
```

Good for checking:

- what the backend finally generated
- whether the program used the IR backend or the AST backend
- whether the output looks acceptable to the system compiler

## A practical debugging order

If the final program behavior is wrong, a good investigation order is:

1. inspect `--emit-tokens`
2. inspect `--emit-ast`
3. if it is a basic `int` subset case, inspect `--emit-ir`
4. inspect `--emit-c`

That narrows down which stage is actually responsible.

## How to tell which compilation path you are on

The rough rule is simple:

- if you see real IR instructions, you are on the IR path
- if the IR indicates a high-level-program fallback or the final C clearly retains arrays, pointers, strings, and similar higher-level constructs, you are usually on the AST backend path

This matters because the debugging strategy is different for each path.

## Why the intermediate generated file matters

When the compiler builds a real executable, it writes a `.generated.c` file.

That file is extremely useful for debugging because it helps answer a key question:

- did this project generate the wrong code, or did the system toolchain fail afterward?

If `.generated.c` already looks wrong, the issue is probably in the project's own frontend/backend pipeline.

If `.generated.c` looks reasonable but system compilation fails, the issue may be closer to the toolchain or platform environment.

## Debugging mistakes to avoid

- do not look only at final binary behavior
- do not ignore whether you are on the IR path or the AST backend path
- do not mix lexical, syntactic, and semantic problems into one vague category

The most effective way to debug a compiler is almost always to localize the issue by stage.
