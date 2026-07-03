# CLI Guide

## Why this page goes deeper than a command list

Many projects document their CLI by listing:

- which flag to type
- what text gets printed

For a compiler project, that is not enough.

The CLI here is not just a launcher. It is the observation interface for the compilation pipeline.

If you only memorize that:

- `--emit-tokens` prints Tokens
- `--emit-ast` prints an AST
- `--emit-ir` prints IR

you may still not understand:

- why Tokens should often be checked before final output
- whether a bad AST indicates a syntax problem or something later
- what it means when IR falls back for advanced features
- whether `--emit-c` is validating the frontend, the backend, or the whole bridge to the toolchain

So this page is not only about how to invoke the CLI. It is about:

- what the CLI corresponds to in compiler terms
- which compiler layer each output mode is showing
- how to use the CLI to localize problems stage by stage

## Where the CLI sits in the compiler

At the code level, the CLI is not the compiler core itself. It is the orchestration entry point for the compilation stages.

Its responsibilities are roughly:

1. read the input source file
2. decide which stage to stop at based on flags
3. invoke the shared compilation pipeline
4. print an intermediate result or continue to a final executable

In the current project, this is mainly coordinated by `src/c_core_compiler/cli.py` and `src/c_core_compiler/pipeline.py`.

One practical way to think about it is:

- `pipeline.py` defines how compilation actually flows
- `cli.py` defines which layer the user wants to inspect

## The most basic full compilation command

The standard usage looks like:

```bash
python3 -m c_core_compiler examples/hello.c -o build/hello
```

This does not just "run a program". It means:

1. read `examples/hello.c`
2. execute the full compilation pipeline
3. generate backend C
4. invoke the host system compiler
5. write the final executable to `build/hello`

The `-o` flag matters because it tells the CLI that you do not just want to inspect a stage. You want a real final program.

## Why `-o` is a boundary

In this project, the CLI has two main usage styles.

### 1. Inspection mode

Examples include:

- `--emit-tokens`
- `--emit-ast`
- `--emit-ir`
- `--emit-ir-dot`
- `--emit-c`
- `--emit-asm`

These modes stop at a layer and print something. They do not continue to binary generation.

### 2. Build mode

For example:

```bash
python3 -m c_core_compiler input.c -o build/program
```

This mode runs the whole chain and produces a final executable.

So `-o` is best understood as the signal that you are moving from pipeline inspection into actual artifact generation.

## What `--emit-tokens` is showing in compiler terms

Command:

```bash
python3 -m c_core_compiler input.c --emit-tokens
```

This observes lexical analysis.

It is not answering "does the program run?" It is answering questions such as:

- were keywords recognized as keywords?
- were identifiers split correctly?
- were integer literals recognized?
- were comments skipped correctly?
- were double-character operators split incorrectly?
- were character and string literals scanned correctly?

In compiler terms, this is the front-most layer. If the Tokens are already wrong, every later stage is built on bad input.

### When Tokens should be the first thing you inspect

For example:

- `==` seems to behave like two `=` tokens
- string or character literal handling looks wrong
- comments appear to leak into code
- an error seems to happen suspiciously early

These are good reasons to start with `--emit-tokens`.

### What this layer cannot answer

Correct Tokens do not guarantee correct syntax.

For example:

- whether parentheses match structurally
- whether an `if` condition is grouped correctly
- whether expression precedence is right

Those belong to the AST layer.

## What `--emit-ast` is showing in compiler terms

Command:

```bash
python3 -m c_core_compiler input.c --emit-ast
```

This observes whether the parser organized the Tokens into the right program structure.

It is concerned with things like:

- how top-level declarations were split
- how function definitions attach parameters and bodies
- whether `if / while / for` were structured correctly
- whether expression precedence is correct
- whether assignment, calls, and indexing were recognized properly

In compiler terms, this validates whether the syntax tree is the one you expect.

### Why the AST is such a central observation point

Because from this stage onward, the rest of the pipeline mainly stops using raw Tokens and begins using the AST.

That means if the AST is wrong:

- semantic analysis sees the wrong structure
- the IR builder lowers the wrong structure
- the backend eventually emits the wrong result

So the AST is one of the most important checkpoints in the whole pipeline.

### When the AST is the right place to inspect

Examples include:

- suspicious precedence, such as `a + b * c`
- an `else` branch that seems attached to the wrong `if`
- strange `for` behavior
- calls and indexing that seem parsed incorrectly

## What `--emit-ir` is showing in compiler terms

Command:

```bash
python3 -m c_core_compiler input.c --emit-ir
```

This observes whether high-level syntax has been lowered into a more explicit intermediate representation.

At this point, you are no longer mainly checking what the code "looks like". You are checking:

- how temporaries were allocated
- how control-flow labels were created
- how branches were expanded
- how loops were rewritten into jump structures
- how calls and returns were made explicit

In compiler terms, this is the intermediate representation layer.

### Why IR matters so much

Because the AST is still a high-level structural representation, while the IR is closer to the form the backend actually wants.

For example:

- in the AST, `if` is still a language construct
- in the IR, `if` has already become condition evaluation, jumps, and labels

Once this step is right, the backend no longer needs to guess control-flow meaning.

### When IR is the right place to inspect

This is especially important if you suspect lowering or control-flow expansion problems, such as:

- loop behavior seems wrong
- short-circuit logic `&& ||` behaves incorrectly
- return paths look strange
- some control-flow region seems expanded incorrectly

## What `--emit-ir-dot` is showing in compiler terms

Command:

```bash
python3 -m c_core_compiler input.c --emit-ir-dot
```

This shows the same IR as `--emit-ir`, but in a representation that is easier for control-flow inspection.

Its value is:

- it makes branch structure easier to see
- it makes loops easier to inspect
- it makes label relationships clearer

If linear IR text feels too dense to reason about control flow, the DOT form is often the better view.

## What `--emit-c` is showing in compiler terms

Command:

```bash
python3 -m c_core_compiler input.c --emit-c
```

This observes the final code the project itself is about to hand to the system toolchain.

It helps answer questions such as:

- did the backend translate the IR into normalized C correctly?
- did the current program go through the IR backend or the AST backend?
- is the emitted structure regular and compilable?

This stage is very close to the final executable, but it is still the project's own output, not the system compiler's output.

### Why `--emit-c` is especially important

Because it sits right on the boundary between the project's internal logic and the external system toolchain.

If the final build fails, there are usually two major possibilities:

1. the project generated bad backend C
2. the host compiler failed later on otherwise reasonable generated C

`--emit-c` and the generated `.generated.c` file are the main tools for separating those two cases.

## Why `--emit-asm` currently behaves like `--emit-c`

By name, many readers expect:

- `--emit-c` to output C
- `--emit-asm` to output assembly

But in the current first-generation implementation, `--emit-asm` is preserved as a compatibility-style interface and still emits normalized C.

That is not meant to confuse the model. It is meant to preserve a stable debugging entry point:

- today the backend emits C
- later, if the backend becomes a native assembly backend, the same user habit can remain

## What `--target` means in principle

The CLI also exposes:

```bash
--target
```

Its meaning is not "switch frontend grammar". It tells the backend/toolchain side which target configuration to use.

In the current project, this is better understood as:

- backend target metadata selection
- preparation for platform-specific output organization

rather than a full industrial-style switch between mature native code generators.

## How the CLI relates to the two compilation paths

This is especially important.

The current project does not use one single internal route for every program:

- basic `int` subset: `AST -> IR -> optimization -> Backend C`
- extended features such as `char`, arrays, pointers, and strings: `AST -> AST Backend`

The CLI does not force you to choose the path manually. The source program content naturally determines the path.

That means:

- `--emit-ir` does not always imply a full conventional IR path for advanced-feature programs
- `--emit-c` is often the best place to confirm which backend route was actually used

So the CLI is not only a result printer. It is also a window into the pipeline split.

## A practical debugging order

If compilation or behavior looks wrong, a very effective order is:

1. inspect `--emit-tokens`
2. inspect `--emit-ast`
3. if it is a basic `int` subset case, inspect `--emit-ir`
4. inspect `--emit-ir-dot` if control flow is hard to read
5. inspect `--emit-c`

The principle behind this order is:

- start from the earliest representation
- validate one stage responsibility at a time
- avoid mixing lexical, syntactic, semantic, and backend questions together

That is one of the core habits in compiler debugging: localize by layer.

## How to use the CLI when learning compiler theory

Many people treat the CLI only as a way to run examples. For learning, a much better method is:

1. pick a very small example
2. inspect Tokens
3. inspect the AST
4. inspect the IR
5. inspect backend C
6. only then run the binary

That way, you actually watch source code change representation step by step.

This teaches much more than looking only at the final behavior.

## One-sentence summary

In this project, the CLI is not just a command entry point. It is the layer-by-layer observation interface for the compiler pipeline, and each flag is most useful when you understand which compiler concept it is validating.
