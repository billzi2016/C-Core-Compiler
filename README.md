# C-Core-Compiler

`C-Core-Compiler` is a controlled C subset compiler project implemented from scratch in Python.

The first-generation goal is not extreme optimization. The priority is to build a compiler pipeline that is logically clear, structurally complete, testable, and explainable. The project transforms source code step by step into Tokens, AST, and IR, then generates normalized backend C code, and finally invokes the system toolchain to produce an executable.

This project is developed continuously in an SDD-style workflow. The goal is not a one-off demo, but an engineering-oriented compiler project with clear structure, complete tests, and room for maintenance and evolution. The first generation follows common industrial baseline expectations first: clear layering, well-defined responsibilities, stable interfaces, traceable errors, and regression-friendly tests.

There are two main reasons behind this design:

- The first generation prioritizes readability, verifiability, and cross-platform executability
- The backend starts with normalized C output so that real executables can be produced quickly on both macOS and Linux

## Project Documentation

If you want a more structured documentation entry, start here:

- Documentation site directory: [`docs-site/`](docs-site/)
- English docs home: [`docs-site/docs/en/index.md`](docs-site/docs/en/index.md)
- Compiler implementation overview: [`docs-site/docs/en/project/how-this-compiler-works.md`](docs-site/docs/en/project/how-this-compiler-works.md)
- GitHub Actions workflow page: [docs workflow on GitHub](https://github.com/billzi2016/C-Core-Compiler/actions/workflows/docs.yml)
- GitHub Actions workflow source: [`.github/workflows/docs.yml`](https://github.com/billzi2016/C-Core-Compiler/blob/main/.github/workflows/docs.yml)

If you want the project explained by compiler stage, continue with:

- [`Architecture Overview and the Two Compilation Paths`](docs-site/docs/en/project/compiler-overview.md)
- [`Repository Map`](docs-site/docs/en/project/repository-map.md)
- [`Lexing and Tokens`](docs-site/docs/en/project/lexer-and-tokens.md)
- [`Parsing and the AST`](docs-site/docs/en/project/parser-and-ast.md)
- [`Semantic Analysis and the Symbol Table`](docs-site/docs/en/project/semantic-analysis.md)
- [`IR and Lowering`](docs-site/docs/en/project/ir-and-lowering.md)
- [`Backend and System Toolchain`](docs-site/docs/en/project/backend-and-toolchain.md)
- [`CLI Guide`](docs-site/docs/en/user-guide/cli-guide.md)
- [`Examples Guide`](docs-site/docs/en/user-guide/examples-guide.md)

## First-Generation Supported Scope

- `int` type
- Global variable declarations
- Local variable declarations
- Function definitions
- Parameter passing
- Function calls
- `if / else`
- `while`
- `for`
- `return`
- Unary operators: `+ - !`
- Binary operators: `+ - * / %`
- Comparison operators: `== != < <= > >=`
- Logical operators: `&& ||`

## Currently Supported Extended Capabilities

- `char`
- Character literals
- String literals
- Fixed-length array declarations
- Array indexing
- Basic pointer declarations
- Address-of via `&`
- Dereference via `*`

## Compilation Pipeline

1. The Lexer splits the character stream into Tokens
2. The Parser converts the Token sequence into an AST
3. The Semantic Analyzer performs symbol and basic semantic checks
4. The IR Builder lowers the AST into three-address-style IR
5. The Backend generates normalized C code from the IR
6. The Toolchain Driver invokes `clang` or `cc` to produce the final executable

## Quick Start

```bash
python3 -m unittest discover -s tests -v
python3 -m c_core_compiler examples/hello.c -o build/hello
./build/hello
```

If you want to directly see standard output, it is recommended to run examples with the `_stdout` suffix:

```bash
python3 -m c_core_compiler examples/fib_stdout.c -o build/fib_stdout
./build/fib_stdout
```

If you want a visually obvious demo output, the full sequence example is a good starting point:

```bash
python3 -m c_core_compiler examples/fib_sequence_stdout.c -o build/fib_sequence_stdout
./build/fib_sequence_stdout
```

## The Difference Between `return` and `stdout`

Examples in this project are divided into two categories with different purposes:

- `return`-style examples:
  - Express the result through the process exit code
  - Better suited for compiler correctness checks, scripts, and automated tests
- `stdout`-style examples:
  - Print the result directly to standard output
  - Better suited for humans to read and for public demonstrations

That maps to the build result directories like this:

- `build/results_return/`: mainly inspect `exit.txt`
- `build/results_stdout/`: mainly inspect `stdout.txt`

For example:

- The result of `fib.c` is `return fib(6);`, so the exit code is `8`
- `fib_sequence_stdout.c` prints `1 1 2 3 5 8 13` line by line, so it is more suitable for direct presentation

## Common Commands

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

Emit backend code:

```bash
python3 -m c_core_compiler examples/hello.c --emit-c
```

Compatibility-preserved option:

```bash
python3 -m c_core_compiler examples/hello.c --emit-asm
```

In the first generation, `--emit-asm` outputs the normalized C code generated by the current backend. This preserves a consistent debugging entry point. If the backend is later replaced with a native assembly backend, the same option can continue to be used.

## Example Run Commands

### Basic Examples

`hello.c`

```bash
python3 -m c_core_compiler examples/hello.c -o build/hello
./build/hello
```

`factorial.c`

```bash
python3 -m c_core_compiler examples/factorial.c -o build/factorial
./build/factorial
```

`fib.c`

```bash
python3 -m c_core_compiler examples/fib.c -o build/fib
./build/fib
```

`control_flow_demo.c`

```bash
python3 -m c_core_compiler examples/control_flow_demo.c -o build/control_flow_demo
./build/control_flow_demo
```

`char_demo.c`

```bash
python3 -m c_core_compiler examples/char_demo.c -o build/char_demo
./build/char_demo
```

`array_demo.c`

```bash
python3 -m c_core_compiler examples/array_demo.c -o build/array_demo
./build/array_demo
```

`pointer_demo.c`

```bash
python3 -m c_core_compiler examples/pointer_demo.c -o build/pointer_demo
./build/pointer_demo
```

`string_demo.c`

```bash
python3 -m c_core_compiler examples/string_demo.c -o build/string_demo
./build/string_demo
```

### Examples with Standard Output

`hello_stdout.c`

```bash
python3 -m c_core_compiler examples/hello_stdout.c -o build/hello_stdout
./build/hello_stdout
```

`factorial_stdout.c`

```bash
python3 -m c_core_compiler examples/factorial_stdout.c -o build/factorial_stdout
./build/factorial_stdout
```

`fib_stdout.c`

```bash
python3 -m c_core_compiler examples/fib_stdout.c -o build/fib_stdout
./build/fib_stdout
```

`fib_sequence_stdout.c`

```bash
python3 -m c_core_compiler examples/fib_sequence_stdout.c -o build/fib_sequence_stdout
./build/fib_sequence_stdout
```

`control_flow_demo_stdout.c`

```bash
python3 -m c_core_compiler examples/control_flow_demo_stdout.c -o build/control_flow_demo_stdout
./build/control_flow_demo_stdout
```

`char_demo_stdout.c`

```bash
python3 -m c_core_compiler examples/char_demo_stdout.c -o build/char_demo_stdout
./build/char_demo_stdout
```

`array_demo_stdout.c`

```bash
python3 -m c_core_compiler examples/array_demo_stdout.c -o build/array_demo_stdout
./build/array_demo_stdout
```

`pointer_demo_stdout.c`

```bash
python3 -m c_core_compiler examples/pointer_demo_stdout.c -o build/pointer_demo_stdout
./build/pointer_demo_stdout
```

`string_demo_stdout.c`

```bash
python3 -m c_core_compiler examples/string_demo_stdout.c -o build/string_demo_stdout
./build/string_demo_stdout
```

## Run All Examples in Batch

Batch-compile and run all basic examples, then save the results under `build/results_return/<example_name>/`:

```bash
mkdir -p build/results_return
for f in examples/*.c; do
  if [[ "$f" == *_stdout.c ]]; then
    continue
  fi
  name=$(basename "$f" .c)
  mkdir -p "build/results_return/$name"
  PYTHONPATH=src python3 -m c_core_compiler "$f" -o "build/results_return/$name/program"
  "build/results_return/$name/program" > "build/results_return/$name/stdout.txt" 2> "build/results_return/$name/stderr.txt"
  printf "%s\n" "$?" > "build/results_return/$name/exit.txt"
done
```

Batch-compile and run all examples with standard output, then save the results under `build/results_stdout/<example_name>/`:

```bash
mkdir -p build/results_stdout
for f in examples/*_stdout.c; do
  name=$(basename "$f" .c)
  mkdir -p "build/results_stdout/$name"
  PYTHONPATH=src python3 -m c_core_compiler "$f" -o "build/results_stdout/$name/program"
  "build/results_stdout/$name/program" > "build/results_stdout/$name/stdout.txt" 2> "build/results_stdout/$name/stderr.txt"
  printf "%s\n" "$?" > "build/results_stdout/$name/exit.txt"
done
```

## Example Result Files

There is an important distinction here:

- `return`
  - Represents the process exit code
  - Better suited for verification-oriented examples
  - Better for testing whether the final computed value is correct

- `stdout`
  - Represents what the program prints to standard output
  - Better suited for presentation-oriented examples
  - Better for directly showing what a person sees after the program runs

Therefore:

- `build/results_return/` is mainly about exit codes
- `build/results_stdout/` is mainly about standard output

Basic example run results are stored in:

- `build/results_return/`
- One subdirectory per example, for example: `build/results_return/factorial/`
- Summary table: `build/results_return/run_results.txt`

Standard-output example run results are stored in:

- `build/results_stdout/`
- One subdirectory per example, for example: `build/results_stdout/fib_sequence_stdout/`
- Summary table: `build/results_stdout/run_results.txt`

If you want the most presentation-friendly result, prioritize:

- `examples/fib_sequence_stdout.c`
- Result file: `build/results_stdout/fib_sequence_stdout/stdout.txt`

## Directory Notes

- `src/c_core_compiler/`: compiler implementation
- `tests/`: unit tests, snapshot tests, and end-to-end tests
- `examples/`: demonstration and regression example programs
- `PRD.md`: project scope and goals
- `TASKS.md`: phase-one and phase-two task lists
- `ARCHITECTURE.md`: module design and data flow
- `TESTING.md`: testing strategy and coverage notes
- `CONTRIBUTING.md`: collaboration and commit conventions
- `DEVELOPMENT.md`: development workflow and module notes
- `RELEASE.md`: version summary and limitations
- `EXAMPLES.md`: example notes and common commands

## Acknowledgements

Thanks to the book *Compilers: Principles, Techniques, and Tools* for helping me build a more systematic compiler knowledge framework.

Thanks to `https://pandolia.net/tinyc/`, which gave me a more hands-on understanding of compiler principles and helped me think more clearly about how to break a compiler pipeline into an engineering structure that is implementable, verifiable, and sustainable to iterate on.
