# ECMAScript Catch Guards

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

There are numerous issues with the current limitation of only a single unguarded catch-clause:

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

## The Proposal

This document proposes new syntax that extends the current specification of the catch-clause with guards. The following example demonstrates how the proposed changes would behave:

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

An error would figuratively flow through all catch-clauses until one is found that either has at least one guard that matches (such that `error instanceof Guard === true`), or no guards at all. Unlike with case-clauses, if one catch-clause matches, the error will not propagate to succeeding clauses.

This also implies the following:

```js
try {
  throw new SyntaxError();
} catch {
  // Got it here
} catch SyntaxError {
  // This block will never execute
}
```

Which means, by extent, that this change is fully backwards-compatible.

Note that if multiple catch-clauses without guards are present, only the first can ever execute.

\
If the error "flows" though all catch-clauses (no catch-all-clause without guards is present), the error will be re-thrown: See [use cases](#use-cases)

## Open Questions / Thoughts

- Should the [same set of expressions that are valid for case-clauses](https://tc39.es/ecma262/#_ref_19602) be allowed for catch guards? If so, a different delimiter would need to be chosen, as `|` could also mean a bitwise or expression.

- Should more than one unguarded catch-clause raise a `SyntaxError`? Unlike with, for example, two `default:` clauses in a switch-block, the execution path would remain unambiguous and following catch-clauses would merely be unreachable. This would make more sense as an ESLint rule than to forbid by specification.

## Comparison with other proposals

- https://github.com/michaelficarra/proposal-catch-guards:

  Option 1: `as` is a well-established keyword in TypeScript and makes no sense in the context used here.

  Option 2, 3: Feels inconsistent with other ECMAScript and doesn't read smoothly in a way I cannot put in words.

  Option 4: As already stated, conflicts with type systems. (Confusing behaviour with `catch (err: any/unknown)`, which would then refer to the _type_ `any`/`unknown` instead of the class)

- https://github.com/mpcsh/proposal-catch-guards / https://github.com/wmsbill/proposal-catch-guards/pull/5/files

  > Consider the following code snippet:

  ```ts
  try {
    apiRequest();
  } catch match (${InternalServerError}) {
    log({ level: 'ERROR', payload: ... });
  }
  ```

  I don't understand this proposal - where is the variable with which I access the error within the catch-body?

- https://github.com/wmsbill/proposal-catch-guards/pull/8/files

  Doesn't look right. A `,` stands for some sort of list, which this is certainly not. `ErrorType` is nothing like `err`.


---

You can browse the [ecmarkup output](https://jeengbe.github.io/proposal-multi-catch/)
or browse the [source](https://github.com/jeengbe/proposal-multi-catch/blob/HEAD/spec.emu).
