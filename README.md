# Multiple catch clauses

## Status

Champion(s): n/a

Author(s): Jesper Engberg

Stage: -1

## Motivation

The aim of the proposed syntax is primarily better readability of (especially large) catch-blocks. It is not uncommon to find code such as the following generalized example:

```js
try {
  someFunction();
} catch (err) {
  if(err instanceof ErrorA) {
    // Do something
  } else if (err instanceof ErrorB) {
    // Do another thing
  } else {
    // Do a third thing
  }
}
```

There are numerous issues with the current limitation of only a single catch-clause:

- It forces developers to often fill the catch-block with several different functions. Not only does this not work well with the [SRP](https://en.wikipedia.org/wiki/Single-responsibility_principle), but it needlessly increases complexity by adding additional layer of indentation.

- It is prone to errors: accidentally omitting an `else` statement can lead to unexpected behaviour and novice developers might attempt a `return` statement to exit the catch-block from within an if-block. This is obviously not what happens.

- Numerous other programming languages support and benefit from multiple catch blocks.

- (Possibly runtime-implementation performance optimizations)

## Use cases

```js
try {
  JSON.parse(fs.readFileSync(file, "utf-8"));
} catch SyntaxError {
  // Corrupt save file; do nothing
}
```

is significantly less complex than

```js
try {
  JSON.parse(fs.readFileSync(file, "utf-8"));
} catch (err) {
  if (err instanceof SyntaxError) {
    // Corrupt save file; do nothing
  } else {
    throw err;
  }
}
```

Extended catch clauses (see above example) will be referred to as "annotated". Suggestions for a better name are welcome.

## The Actual Proposal

This document proposes new syntax that extends the current specification of the catch-clause. The following example demonstrates how the proposed changes would behave:

```js
try {
  someFunction();
} catch ErrorA (err) {
  // Do something
} catch ErrorB | ErrorC (err) {
  // Do another thing
} catch {
  // Do a third thing
} finally {
  // Clean up
}
```

would be functionally equivalent to

```js
try {
  someFunction();
} catch (err) {
  if(err instanceof ErrorA) {
    // Do something
  } else if (err instanceof ErrorB || err instanceof ErrorC) {
    // Do another thing
  } else {
    // Do a third thing
  }
} finally {
  // Clean up
}
```

An error would figuratively flow through all catch-clauses until one is found that either has at least one annotation that is a superclass of the error (such that `error instanceof Annotation === true`), or no annotations at all. Unlike with case-clauses, errors do not propagate from one catch-clause to the next.

This also implies the following:
```js
try {
  someFunction();
} catch {

} catch SyntaxError {
  // This block will never execute
}
```

This means, by extent, that this change is fully backwards-compatible.

Note that if multiple catch-clauses without arguments are present, only the first will ever possibly execute.

\
If the error "flows" though all catch-clauses (no catch-all-clause without annotations is present), the error will be re-thrown: See [use cases](#use-cases)

## Open Questions / Thoughts

- Should the [same set of expressions that are valid for case-clauses](https://tc39.es/ecma262/#_ref_19602) be allowed for annotations on catch-clauses? If so, a different delimiter would need to be chosen as `|` could also mean a bitwise or expression.

- Should more than one unannotated catch-clause raise a SyntaxError? Other than, for example two `default:` clauses in a switch-block, the execution path would remain unambiguous and following catch-clauses would merely be unreachable. This would make more sense as an ESLint rule than to forbid by specification.

---

You can browse the [ecmarkup output](https://jeengbe.github.io/proposal-multi-catch/)
or browse the [source](https://github.com/jeengbe/proposal-multi-catch/blob/HEAD/spec.emu).
