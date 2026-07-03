# What the Optimizer Actually Does

## First, set expectations correctly

This project has an optimizer, but it is not a large everything-optimizer.

Its current goal is deliberately restrained:

- perform safe, explainable optimizations first
- avoid algorithms that would add a lot of complexity too early
- keep the before/after behavior easy to inspect

That fits the first-generation scope well.

## What it currently does

The main categories are:

- constant propagation
- constant folding
- removal of obviously unreachable code

It also does one practical cleanup step:

- compaction of temporary names so the IR does not explode into too many one-shot temporaries

## What constant propagation means

If a temporary is already known to hold a constant, later uses of that temporary can reuse that fact.

For example:

```text
t0 = 3
t1 = t0
```

After propagation, `t1` can also be treated as `3`.

The current project intentionally limits this to temporaries, because temporaries in the IR are generated in a much more stable way than mutable local variables, which makes the rule easier to validate.

## What constant folding means

If an expression can be computed at compile time, there is no reason to leave it for runtime.

For example:

```text
t0 = 3
t1 = 4
t2 = t0 + t1
```

can become:

```text
t2 = 7
```

This kind of optimization is simple, visible, and very appropriate as an early optimization capability.

## What unreachable-code pruning means

After constant propagation and folding, a conditional jump may become deterministically true or false at compile time.

Once that happens, one side of the branch may be obviously unreachable.

For example:

- if the condition is always true, keep only the true branch
- if the condition is always false, keep only the false branch

This project performs a reachability pass over the explicit IR control flow to prune such cases.

## Why explicit control flow helps optimization

Because the IR already contains:

- labels
- jumps
- conditional jumps

That means the optimizer does not have to infer execution flow from high-level syntax. It can reason directly from the control-flow structure already present in the IR.

That is one of the main advantages of optimizing IR rather than raw AST structure.

## What temporary compaction is doing

This is not a full machine-level register allocator, but the idea is related.

The optimizer tracks:

- when a temporary is last used
- whether its name can be reused afterward

As a result, a long chain of single-use temporaries like `t0`, `t1`, `t2`, and so on can be rewritten into a smaller set of virtual register-like names such as `r0`, `r1`.

That has two benefits:

- the generated result becomes tighter
- it leaves a cleaner bridge toward future lower-level backend work

## Why the optimizer does not do more yet

Because "more" is not always better, especially in a first-generation project.

If the project started with heavy data-flow analysis, large global optimization passes, or alias analysis, the cost would rise quickly:

- harder to verify
- harder to explain
- harder to maintain

The current choice is to do a small amount of optimization well. That is a rational staging decision.

## Where the optimizer applies

The optimizer only runs on the IR path.

If the program goes through the AST backend path, this IR optimization pipeline does not participate.

So do not read "the project has an optimizer" as meaning "every supported program goes through the same optimization stage". That is not true yet.

## What its engineering value is

The optimizer matters not because it squeezes out maximum performance, but because it proves three things:

- the compiler is more than a direct translator
- the architecture already has a real optimization stage
- future optimization work can be added without redesigning the entire pipeline

That is an important milestone in a compiler project.
