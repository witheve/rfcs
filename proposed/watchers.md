# Watchers: Seamless external I/O

## Summary

This RFC proposes a pattern for generic API-driven I/O in Eve. The aim is to achieve a simple, fast, and correct form of FFI while preserving our current semantics.

## Motivation

Within Eve, asychrony, malformed data, and other real-world issues are carefully abstracted away to provide a clean and intuitive programming environment. The need for external I/O threatens to re-introduce some of these issues into the language. We've designed first-party interfaces for the file system and processes which strategically assert and read facts in the system through standard channels to obviate this. This proposal aims to make those patterns generic and easy to implement purely in-language when possible (e.g., JSON API's like Twitter), or through an easily extensible path in the executor when not (e.g., manipulating the scene graph of a native 3D engine).

## Design

### Terminology

- *Context* - See modules RFC.
- *Block* - An Eve code block.
- *Watcher* - An API endpoint, data provider, or both.
- *Executor* - The C portion of the runtime.

### Semantics

A watcher consists of 1 or more contexts (collectively the watcher's scope), and an optional executor system that handles inserting or extracting facts in external sources. Each context may contain a set of blocks. A block requires the watcher when it matches on or binds/commits into the watcher's scope. A watcher is lazily initialized once it has been required, meaning that an arbitrary number of watchers may be available without impacting performance unless they are used. Once a watcher has been initialized, the blocks contained in its contexts are evaluated normally as if they were part of the program created by the user.

There are two forms of watchers. A primitive watcher includes an executor system which manages pulling in and sending off facts in the external world. The JSON, File, and Process watchers are all primitive watchers. in the JSON case, it externalizes facts tagged `#json-request` in the appropriate context by sending them as http requests. If these requests have a specified body, the body record is converted into a standard JSON structure and included in the request. Similarly, if the response contains JSON, it is parsed back into records in the same shape and inserted back into Eve.

A simple watcher is written in pure Eve and relies on an existing primitive watcher for interacting with the outside world. A Twitter watcher might provide a nice API for interacting with tweets on top of the existing JSON primitive watcher by transforming facts in its context and inserting those into the primitive watcher's context and vice versa.

Policy concerns and integrity constraints are inherited from the contexts comprising the watcher.

### Syntax

Syntax is essentially unchanged from traditional contexts. The sole distinction is that watchers exist in a separate document from the current evaluation and are only injected in when they are initialized. While primitive watchers may interact with the database, they may only do so by inserting, removing, or reading EAVs from their scopes.

### Changes to the runtime

- Blocks now reside in contexts
- The scope of a block is its document (which currently maps 1:1 with a file, but may not in the future)
- Only the current document is evaluated by default
- Documents may be registered as watchers in the executor
  * Watcher documents *must* only read and write into contexts in their scope
- When the current document matches/binds/commits in a context provided by a watcher available in the current runtime, that watcher's document is merged into the evaluation
  * This may happen statically at compilation time or dynamically JIT, but the watcher is guaranteed to be initialized prior to use
  * If the other document is a primitive watcher, it's executor system is guaranteed to be initialized statically at compilation time (in the event that it needs to react to external events and trigger changes in the evaluation)

### Implementation
@TODO: Write me

### Drawbacks
@TODO: Write me
