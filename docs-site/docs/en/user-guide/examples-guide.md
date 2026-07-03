# Examples Guide

## Why `examples/` is more than a demo folder

In many repositories, an `examples/` directory is just a pile of demo scripts. In this project, it is closer to a teaching-oriented sample set for the language and the compiler pipeline.

These examples serve at least four roles:

1. showing which language features the compiler currently supports
2. giving the CLI and end-to-end flow small runnable inputs
3. helping readers understand the language boundary by difficulty
4. providing material for observing compilation stages step by step

So the right way to read `examples/` is not just as "things to run", but as:

- a description of the language subset boundary
- a sample set of compilation paths
- a debugging and learning roadmap

## Why many examples come in pairs

You will notice many paired files such as:

- `hello.c` and `hello_stdout.c`
- `fib.c` and `fib_stdout.c`
- `factorial.c` and `factorial_stdout.c`

That is intentional.

### `return`-style examples

Examples such as:

- `fib.c`
- `factorial.c`

usually express their result through `return`.

They are better suited for:

- automated tests
- exit-code validation
- small, crisp correctness checks

From a compiler-learning perspective, they are more about validating semantic correctness than about visible presentation.

### `stdout`-style examples

Examples such as:

- `fib_stdout.c`
- `fib_sequence_stdout.c`

print their results directly.

They are better suited for:

- human inspection
- teaching demos
- visible behavioral presentation

For a first pass through the project, these are often easier to grasp because the results are immediately visible.

## How to interpret different examples in compiler terms

### `hello.c`

This is the minimal starting point.

It is good for observing:

- the smallest function structure
- the shortest successful compilation path
- what the smallest backend output looks like

If you want your first look at Tokens, AST, and backend C, this is the right example.

### `factorial.c`

This example is useful for observing:

- function calls
- recursion
- return paths

If you want to understand how calls and returns appear in the AST, IR, and backend, this is a strong representative case.

### `fib.c`

This example is useful for observing:

- repeated recursive calls
- conditional branching
- composed expressions

It is more complex than `hello.c`, but it still belongs to the basic `int` subset and is a very good IR-path study sample.

### `control_flow_demo.c`

This example is especially useful for observing:

- `if / else`
- `while`
- `for`
- control-flow lowering

If you want to study:

- how labels are generated
- how jumps are formed
- what the IR control-flow graph looks like

then this example is more informative than simple arithmetic examples.

### `char_demo.c`

This example is useful for observing:

- `char`
- character literals

Its importance is that it usually moves beyond the simplest `int` subset, which makes it useful for understanding why extended features trigger a different backend route.

### `string_demo.c`

This example is useful for observing:

- string literals
- backend output for more advanced language features

If you are not yet fully sensitive to why not all features go through the same IR path, this example helps a lot.

### `array_demo.c`

This example is useful for observing:

- fixed-length array declarations
- indexing

In principle, it is especially good for understanding:

- why arrays make simple IR unification harder
- why the current project chooses the AST backend first for these features

### `pointer_demo.c`

This example is useful for observing:

- pointer declarations
- address-of `&`
- dereference `*`

This is one of the most important examples in the project because it directly touches more realistic memory-model questions.

If you want to understand why the first generation did not force every complex feature into the IR immediately, this is one of the best examples to study.

## Which examples are best for IR and which are best for the AST backend

### Better examples for studying the IR path

Prioritize:

- `hello.c`
- `factorial.c`
- `fib.c`
- `control_flow_demo.c`

These are closer to the core first-generation path:

```text
AST -> IR -> optimization -> Backend C
```

### Better examples for studying the AST backend path

Prioritize:

- `char_demo.c`
- `string_demo.c`
- `array_demo.c`
- `pointer_demo.c`

These are more likely to trigger the current extended-feature route:

```text
AST -> AST Backend -> Backend C
```

This group is especially valuable for understanding why the architecture currently has two backend paths.

## A good example-reading order for learning compiler ideas

Here is a practical sequence.

### Step 1: start with the smallest case

Start with:

- `hello.c`

Then inspect:

- Tokens
- AST
- backend C
- the final executable

The goal is to build an intuition for the minimal full path.

### Step 2: move to control flow

Then study:

- `control_flow_demo.c`

Pay attention to:

- how control-flow nodes look in the AST
- how labels and jumps appear in the IR
- how the DOT graph shape forms

### Step 3: move to calls and recursion

Then study:

- `factorial.c`
- `fib.c`

Pay attention to:

- call structure
- return paths
- the combination of branching and recursion

### Step 4: only then move to extended features

Then study:

- `char_demo.c`
- `string_demo.c`
- `array_demo.c`
- `pointer_demo.c`

The key idea here is:

- why these features are harder to unify into the current IR
- why the AST backend route is a reasonable first-generation choice

## The best way to study examples with the CLI

Do not only run the final executable.

A better approach is to run a sequence like:

```bash
python3 -m c_core_compiler examples/hello.c --emit-tokens
python3 -m c_core_compiler examples/hello.c --emit-ast
python3 -m c_core_compiler examples/hello.c --emit-ir
python3 -m c_core_compiler examples/hello.c --emit-c
```

Then repeat the same pattern with `control_flow_demo.c` and `pointer_demo.c`.

That way you are not just "looking at many examples". You are watching how the same compiler handles different language structures through different layers.

## One-sentence summary

In this project, `examples/` is not just a loose demo folder. It is a learning sample library for the language subset boundary, the split compilation paths, and the visible shape of each compiler stage.
