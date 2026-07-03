# Backend and System Toolchain

## What the backend is doing

By the time the backend runs, earlier stages have already organized the program logic into a much more explicit form.

The backend does not need to rediscover semantics from scratch. Its job is to turn those prepared structures into target code that can actually be built.

In this project, that target is normalized C rather than native assembly.

## Why emit C first

This choice is central to the first-generation architecture.

Emitting normalized C has several advantages:

- it immediately enables executable generation through the system compiler
- it works more easily across macOS and Linux
- the intermediate output is much easier to read
- it avoids dealing immediately with ABI rules, assembly syntax, and object-file details

So this backend acts as a practical bridge to real executables.

## There are two backend modes

### 1. IR backend

If the program follows the basic `int` subset path, the backend consumes IR.

In that case, each IR instruction is translated into a relatively direct C statement, such as:

- `const` -> constant assignment
- `copy` -> ordinary assignment
- `binary` -> binary expression
- `jump` -> `goto`
- `cjump` -> `if ... goto ... else goto ...`
- `return` -> `return`

This path works because the earlier lowering phase has already made control flow explicit.

### 2. AST backend

If the program uses `char`, strings, arrays, pointers, or related features, the backend consumes the AST instead.

In that path, it directly renders C structures such as:

- pointer declarations
- array declarations
- indexing expressions
- string literals

This preserves more of the source-level structure.

## What "normalized C" means here

It does not mainly mean pretty C. It means regular and stable generated C.

For example:

- the output form is kept consistent
- locals and temporaries are explicitly declared
- control flow is made direct
- the result is easier to inspect and test

So the first goal of the backend output is to be compiler-friendly, not to look like hand-written application code.

## Why the system toolchain still matters

Even after C has been generated, the project still does not produce a final executable by itself.

The last step still belongs to the host toolchain, such as `clang` or `cc`.

Those tools handle:

- real C compilation
- object file generation
- linking
- platform-specific details

That is what `toolchain.py` is responsible for coordinating.

## What `ToolchainDriver` does

The current `ToolchainDriver` mainly does three things:

1. select the backend and generate backend text
2. write that result to an intermediate `.generated.c` file
3. invoke the system C compiler to produce the final executable

Writing the intermediate file is especially useful because:

- if the build fails, you can inspect the generated code directly
- users can see what the compiler actually emitted
- it supports regression testing and manual debugging

## Why this is a good first-generation design

If the project started with a native assembly backend, it would immediately need to solve:

- instruction selection
- register allocation
- calling conventions
- stack frame management
- platform differences

That would increase complexity dramatically.

The current design splits the problem more sensibly:

- make the frontend and middle layer solid first
- use normalized C to make the execution path real
- replace the backend later if lower-level generation becomes the next goal

That is a very reasonable engineering sequence.
