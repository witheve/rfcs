# Data and code modularity through "contexts"

## Summary

In this RFC we present our idea for how modules work in Eve. Modularity has a spectrum of meanings, ranging anywhere from a collection of functions to package management systems. Modularity in this context refers to a collection of code that can be packaged, distributed, and referenced in other Eve programs to enable code reuse.

## Motivation

In order for Eve to succeed, Eve programs cannot exist in isolation. That is to say, Eve programmers should be able to leverage work done by the Eve community to accelerate the development of their own applications. To do this, programmers need a way to pull in the work of others into their own projects.

- Reuse of code
- Reuse of data
- Encapsulation

## Design

### Terminology

Context
Alternatives: module, library, bag, bucket, batch, container, volume, dataset

### Design Goals

### Semantics

Contexts are containers of facts and code.

- Composition
  - Creating contexts that are compositions vs runtime composition
  - Types
    - Completely isolated loading of a context
    - Write isolated loading, including rebinding
    - Completely merged
    - Data only?
    - Code only?
  - Logs
  - Integrity constraints

- Built-in contexts
  - session
  - global
  - browser
  - file

### Syntax

```
  // match against all bags in the default composition
  match
  // match against only galaga
  match @galaga
  // match against galaga and instance
  match (@galaga, @instance)
  // match against math and the default composition
  match (@math, @default)
```

```
  // bind into session, these are equivalent
  bind
  bind @session
  // bind into galaga
  bind @galaga
  // bind into both galaga and instance
  bind (@galaga, @instance)

  // commit into session, these are equivalent
  commit
  commit @session
  // commit into galaga
  commit @galaga
  // commit into galaga and instance
  commit (@galaga, @instance)
```

```
  bind
    [#context @galaga url: ""]
```

### Changes to the runtime

- Match can optionally take a set of contexts to search in. If none is given, we use the default composition.
- Bind/Commit can optionally take a set of contexts to write to. If none is given, we use the default composition.
- We can load a context by creating a `[#context]` record.
- contexts can be added to the default composition through attributes on a context record.
- Every context internally has both bound and committed facts, which means each user-level context is two internal contexts.

## Implementation

## Drawbacks

## Alternatives

## Risks