# IR and Lowering

## Why have another layer after the AST

A common first question is: if the AST is already structured, why transform it again?

Because the AST is mainly good at representing syntax, while the backend prefers something more explicit about actions and control flow.

An `if`, `while`, or `for` node in the AST is still a high-level construct. The backend cares more directly about things like:

- how the condition is evaluated
- where control jumps
- where a loop begins
- where a loop ends

That is the value of the IR layer.

## What the IR looks like in this project

The current IR is not SSA and not especially heavy-weight. It is a compact three-address-style representation.

Its main building blocks are:

- globals
- functions
- temporaries
- instructions
- labels

Common instructions include:

- `const`
- `copy`
- `load`
- `store`
- `unary`
- `binary`
- `jump`
- `cjump`
- `call`
- `return`

## Why it is closer to an action sequence

An `if` statement may be a single AST node, but in the IR it is expanded into something closer to:

1. evaluate the condition
2. perform a conditional jump
3. enter the then label
4. execute the then-side logic
5. jump to the end
6. optionally enter the else label

At that point, the backend no longer has to interpret high-level control flow. It can simply emit the explicit sequence.

## What lowering means

You can think of lowering as expanding higher-level structure into lower-level structure.

That is what `IRBuilder` does in this project.

It does not merely copy the AST. It rewrites the AST into:

- more explicit control flow
- more explicit intermediate values
- a shape that is easier for the backend to consume

## Why temporaries matter

Once expressions are broken into smaller evaluation steps, the compiler needs places to store intermediate results.

For example:

```c
a + b * c
```

can be expanded as:

```text
t0 = b * c
t1 = a + t0
```

Here `t0` and `t1` are temporaries.

They make it possible to turn a nested expression into a sequence of simple steps.

## How control flow gets expanded

### `if`

`if` is lowered into:

- condition evaluation
- a conditional jump
- then label
- else label
- end label

### `while`

`while` is lowered into:

- condition label
- body label
- end label

with explicit jump edges forming the loop.

### `for`

`for` is normalized into four segments:

- initialization
- condition
- body
- update

That is important because it turns one surface-level syntactic form into a backend-friendly standard shape.

## Why `&&` and `||` are not ordinary binary operators

Because they have short-circuit semantics.

For example:

```c
a && b
```

should not evaluate `b` if `a` is already false.

So lowering cannot treat these as ordinary `binary` operations. It must use branching and labels to preserve the decision of whether the right-hand side should even be evaluated.

The current project does that explicitly in the IR builder.

## Where the current IR stops

One very important point is that the current IR mainly covers the basic `int` subset.

If the AST contains:

- `char`
- arrays
- pointers
- string literals
- address-of or dereference

then the compiler marks the program as requiring the AST backend instead of lowering through this IR.

So the IR is not yet the single unified representation for every supported language feature.

## Is that a problem

It is not a bug. It is a staged boundary.

More precisely, the project is making an engineering choice:

- first make the IR path clean for the core subset
- first make the more complex features usable
- later decide how to move them into a unified IR

That is more honest and maintainable than pretending everything is unified prematurely.

## How to inspect this layer

You can run:

```bash
python3 -m c_core_compiler input.c --emit-ir
```

or:

```bash
python3 -m c_core_compiler input.c --emit-ir-dot
```

The first is good for reading linear IR text. The second is useful for visualizing control flow.

If you want to understand the middle-layer mindset of this compiler, this is one of the most important pages to study.
