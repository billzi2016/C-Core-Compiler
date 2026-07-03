# Getting Started

## Who this page is for

If you are new to this project, the first obstacles are usually not compiler theory. They are practical questions such as:

- which command should I run first?
- how do I confirm the project is actually working?
- which examples are best for a first pass?
- should I start from source code, docs, or intermediate outputs?

This page answers those questions.

## The first thing to do

Start by running the test suite so you know the current environment is basically healthy:

```bash
python3 -m unittest discover -s tests -v
```

This helps confirm:

- the Python environment is usable
- the current code state passes the existing tests
- if example execution fails later, the problem is more likely to be toolchain or local setup related

## The first example to run

The smallest recommended starting point is `examples/hello.c`:

```bash
python3 -m c_core_compiler examples/hello.c -o build/hello
./build/hello
```

This is a good first example because:

- the syntax is simple
- the compilation path is short
- failure diagnosis stays narrow

If this example does not work, it is not a good idea to jump straight to arrays, pointers, or strings.

## If you want directly visible output

Many of the basic examples communicate results through the process exit code rather than printing.

If you want output you can read immediately, start with the `_stdout` examples:

```bash
python3 -m c_core_compiler examples/fib_stdout.c -o build/fib_stdout
./build/fib_stdout
```

Or use the more visually obvious sequence demo:

```bash
python3 -m c_core_compiler examples/fib_sequence_stdout.c -o build/fib_sequence_stdout
./build/fib_sequence_stdout
```

## How to interpret the results

There are two common result styles in this project.

### 1. `return`-style

The program expresses its result through the process exit code.

This is better for:

- automated testing
- scripting
- small correctness checks

### 2. `stdout`-style

The program prints its result directly.

This is better for:

- human inspection
- demos
- teaching

## Recommended reading order for newcomers

If you want to run the project while also understanding it, this is a good order:

1. start with the [documentation home](index.md)
2. read [How This Compiler Works](project/how-this-compiler-works.md)
3. then read [Architecture Overview and the Two Compilation Paths](project/compiler-overview.md)
4. run `--emit-tokens`, `--emit-ast`, and `--emit-c`
5. only then go deeper into the stage-by-stage implementation docs

## The most useful inspection commands

Emit Tokens:

```bash
python3 -m c_core_compiler examples/hello.c --emit-tokens
```

Emit AST:

```bash
python3 -m c_core_compiler examples/hello.c --emit-ast
```

Emit IR:

```bash
python3 -m c_core_compiler examples/hello.c --emit-ir
```

Emit backend C:

```bash
python3 -m c_core_compiler examples/hello.c --emit-c
```

These commands matter because they let you inspect the pipeline stage by stage instead of treating the compiler as a black box.

## What to read next

Depending on your goal, continue with:

- for architecture and implementation ideas: [How This Compiler Works](project/how-this-compiler-works.md)
- for repository orientation: [Repository Map](project/repository-map.md)
- for a stage-by-stage command-line inspection model: [CLI Guide](user-guide/cli-guide.md)
- for understanding what each sample program is really teaching: [Examples Guide](user-guide/examples-guide.md)
- for validation strategy: [Testing and Validation](user-guide/testing-and-validation.md)
