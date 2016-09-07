# Watchers: Seamless external I/O

## Summary

This RFC proposes a pattern for generic API-driven I/O in Eve. The aim is to achieve a simple, fast, and correct form of FFI while preserving our current semantics.

## Motivation

Within Eve, asychrony, malformed data, and other real-world issues are carefully abstracted away to provide a clean and intuitive programming environment. The need for external I/O threatens to re-introduce some of these issues into the language. We've come up with an EAV-based interface to the outside world to avoid this. This proposal aims to make that pattern generic and easy to implement. Potential use-cases include extending Eve to communicate over IPC or to control the scene graph for a native 3D renderer.

## Non-Motivation
We aim to provide watchers to handle common forms of I/O like JSON APIs and the file system as part of the standard library. This means that anything which uses those I/O methods may be implemented purely in-language. For example, an `@tweet` context that lazily fetches tweets as they are requested may be provided as a standard module on top of the existing JSON watcher. Watchers should essentially never be specialized for communicating with a single source. Instead, they should be treated as transport methods that modules may use to query and act on the outside world.

## Design

### Terminology

- *Block* - An Eve code block.
- *Context* - See modules RFC.
- *Watcher* - An API endpoint, data provider, or both.
- *Executor* - The C portion of the runtime.

### Semantics

A watcher consists of one or more contexts (collectively the watcher's scope), and an optional executor component. A running evaluation requires the watcher when it matches on or binds/commits into the watcher's scope. A watcher is lazily initialized once it has been required, meaning that an arbitrary number of watchers may be available without impacting performance unless they are used. Once a watcher has been initialized, its blocks are merged into the running evaluation.

Once initialized, watchers may push state from the outside world into the database or trigger various actions based on records in the database. In both cases, it does this by reading/writing records into its context(s). For example, the JSON watcher dispatches HTTP requests when records tagged `#json-request` are added to its context. If the request object has a body attribute containing a record, it gets converted into a standard JSON string and sent with the request. If the response contains JSON, it is mapped into a record that is inserted into its context and attached to the request as it's response attribute. The request's status and code attributes will also be updated as appropriate.


@FIXME: @convolvatron needs to review this:

Example usage of the JSON watcher:

Save an author to the db.

```
  commit
    [#author @me id: 1]
```

Request each author's information from the jsonplaceholder domain. Note that `#profile` tag and `author` attribute here are attached for our own convenience, to re-associate the request with our author when it has returned. The JSON watcher completely ignores them.

```
  match
    author = [@me id]
  commit @JSON
    [#json-request #profile url: "http://jsonplaceholder.typicode.com/users/{{id}}" author]
```

Apply the response as the author's profile attribute.

```
  match @JSON
    [#json-request #profile author response]
  commit
    author.profile := response
```

Report any errors that occur when querying for user profiles.

```
  match @JSON
    [#json-request #profile author status: "failed" code]
  match
    name = author.name
  commit
    [#div class: "error" text: "Failed to retrieve profile for {{name}}. Got code: {{code}}"]
```

### Syntax

Syntax is essentially unchanged from traditional contexts. The sole distinction is that watchers exist in a context external to the current evaluation and are only injected when they are initialized. While watchers may interact with the database, they may only do so by inserting, removing, or reading EAVs from their scopes.

### Changes to the runtime

- Contexts are required by evaluations as they are matched on our bound/commited into
- When a watcher context is required it initializes its C component and may then read/write records in the DB
  * Watchers may only manipulate contexts in their scope
- New watchers may be registered in the executor
  * Specifics TBD


### Implementation

The framework for watchers lives in `csrc/watchers.c`. It comprises the following types and functions.

`typedef closure(watcher_initializer, evaluation)`
`typedef closure(watcher_listener, bag context, vector dirty_eavs)`

`boolean watcher_register(string name, vector scopes, watcher_initializer init)` - Register a new watcher with a continuation to construct it prior to first use.

`void watcher_listen(string context, watcher_listener listener)` - Listen to batched EAV changes on the named context. NOTE: context must be in watcher scope.

`void watcher_inject(string context, vector eavs)` - Inject the given EAVs into the named context. NOTE: context must be in watcher scope.

NOTE: A forthcoming higher-level API may allow listening to a record pattern for updated records and directly injecting records instead of EAVs.

Tangentially, it would be very, very beneficial to be able to query records from a set of EAVs, e.g.

### Drawbacks
@TODO: Write me
