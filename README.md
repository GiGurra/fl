# Fluffy-Lang (fl)

**NOTE: This is a thought experiment, with lots of LLM generated text. All syntax below is work in progress and subject to change.**

A programming language emphasizing structural typing, compile-time constraints, and immutable data.

## Core Concepts

### Structural Typing

Everything in Fluffy-lang is defined by its shape, not its name. Types and constraints are inferred from structure, with
names serving as optional hints.

```fluffy
// These are equivalent
person = {name: "Alice", isCalm: true}
person: {name: string, isCalm: bool} = {name: "Alice", isCalm: true}

// Optional type aliases for readability
Person = {
    name: string,
    isCalm: bool,
    age: int
}

// Extra fields are allowed
expanded: Person = {name: "Bob", isCalm: true, age: 25, extra: "field"}
println(expanded.extra)  // "field", compiles and works
println(expanded.extra2)  // does not compile
```

### Union Types

Fluffy uses union types for representing different variants:

```fluffy
Message = 
    {type: "text", content: string}
    | {type: "image", url: string, size: int}
    | {type: "error", code: int, message: string}

processNumber(n: int) {ok: int} | {error: string} {
    if n < 0 {
        // early_return is a reserved keyword. There is no regular 'return'
        early_return {error: "number cannot be negative"}
    }
    // The last expression is implicitly returned by default
    {ok: n * 2}
}

// Pattern matching on unions
match msg {
    {type: "text", content} -> handleText(content)
    {type: "image", url, size} -> handleImage(url, size)
    {type: "error", code, message} -> logError(code, message)
}
```

### Comptime Constraints

Constraints define compile-time verifiable conditions. They work based on structure and can be named for convenience.

```fluffy
// Using constraints
process(d where .isCalm) {
    // d is guaranteed to be calm here
}

// Named constraints
where calm = .isCalm
where angry = !.isCalm

if person is calm {
    // person is calm in this scope
}

// Numeric constraints
myNumber: int with _ < 50 = 42
```

### Compile-time Constraint Inference

The compiler can infer relationships between constraints, if you provide the proof (think Zig's comptime):

```fluffy
comptime proof(int with _ < limit0, int with _ < limit1) bool {
    limit0 < limit1
}

myNumber: int with _ < 50 = 42
myNumber: int with _ < 51 = myNumber  // Works because 50 < 51
```

### Runtime Constraint Verification

Comptime proofs and constraints can be used at runtime to verify data:

```fluffy
where shortString = string with _.length <= 10

someString = readFromOutsideWorld()
if !someString is shortString {
    early_return {error: "string too long"}
}
// someString is now known to be a short string
processShortString(someString)
```

### Immutable State Transitions

All data is immutable. State changes create new values using the `with` operator:

```fluffy
// omit return type to infer it. Think C++ templates
calmDown(d angry) {
    d with {isCalm: true}
}

person = {name: "Bob", mood: "grumpy", isCalm: false}
calmPerson = calmDown(person)
// Result: {name: "Bob", mood: "grumpy", isCalm: true}
```

### Concurrency (BEAM-based)

Fluffy leverages the BEAM's actor model for concurrency:

```fluffy
// Spawn a new process
pid = spawn_typed(someFunction)

// We also know the types of messages that our process can receive,
// as any function passed to spawn_typed must have a known type signature.
// We can also use the untyped spawn if we have to work with dynamic types.

// Send message
pid <- message

// Receive messages
receive {
    {type: "hello"} -> handleHello()
    {type: "bye"} -> handleBye()
    _ -> handleDefault()
}
```

### Modules and Imports

Just like erlang, we can access any module using a fully qualified name to the function or type:

```fluffy
some_module:some_function(...)
x: some_module:some_type = ...
```

If you wish to make them locally usable without using the fully qualified name, you can create local aliases:

```fluffy
some_type = some_module:some_type
```

There is, intentionally, no way to import/use a different module's functions or types without using the fully qualified
name or creating a local alias.

### Pattern Matching

First-class support for pattern matching with constraints:

```fluffy
process(msg) {
    match msg {
        {command: "start"} where .value > 0 -> 
            msg with {status: "running"}
        {command: "stop"} -> 
            msg with {status: "stopped"}
        m where .isCalm -> 
            handleCalm(m)
        _ -> 
            msg with {error: "invalid command"}
    }
}
```

or clause style:

```fluffy
process(msg where .command == "start" && .value > 0) {
    msg with {status: "running"}
} 
process(msg where .command == "stop") {
    msg with {status: "stopped"}
}
process(msg where .isCalm) {
    handleCalm(msg)
}
process(msg) {
    msg with {error: "invalid command"}
}
```

## Why Fluffy-Lang?

- **Zero Runtime Overhead for function parameter constraints**: All constraints are checked at compile-time
- **First-Class Union Types**: Powerful pattern matching with tagged tuples
- **Compile-Time Proofs**: Verify constraints at compile-time
- **Constraints usable both compile and runtime**: Use constraints to verify data at runtime and lift types
- **Type Safety Without Ceremony**: Structural typing with static guarantees
- **BEAM Platform**: Proven runtime system with excellent concurrency and fault tolerance
- **Immutable by Default**: Safe and predictable state management
- **Expression-Based**: Clean and functional style
- **Flexible Constraints**: Power of dependent types with simplicity of structural typing

## Example: State Machine

```fluffy
State = {
    status: string,
    value: int
}

where stopped = .status == "stopped"
where running = .status == "running"

start(s: State where stopped) {ok: State where running} | {error: string} {
    if s.value < 0 {
        early_return {error: "invalid starting value"}
    }
    
    {ok: s with {
        status: "running",
        lastStart: timestamp()
    }}
}

stop(s: State where running) State where stopped {
    s with {status: "stopped"}
}
```

## Exceptions and panics

Fluffy-lang does not have exceptions. Fluffy has panics. Panics are used to signal unrecoverable errors, and are
enforced to crash the current process. You cannot catch panics, and they are not meant to be caught. Panics are
used to signal that something went wrong and the process should crash.

Because of how the BEAM works, panics are not as bad as they sound. The BEAM is designed to handle process crashes
gracefully, and the supervisor tree will restart the process if it crashes.

```fluffy
panic("Something went wrong")
```

There are no `try` or `catch` keywords.

## Defer

Fluffy has golang style defer. Defer is used to run a function at the end of the current scope, regardless of how the
scope exits. This is useful for cleanup tasks.

NOTE: While golang's defer runs the deferred function at the end of the current function, Fluffy's defer runs the
deferred function at the end of the current scope.

```fluffy
defer {
    println("This will run at the end of the scope")
}
```

## Loops

Fluffy has Loops. It does however not have mutable variables, so loops are mostly for calling creating external side
effects, like tests and database interactions etc

## Current Status

⚠️ Fluffy-lang is currently in early development. The syntax and features are subject to change.

First experimental compiler to be written in Go, targeting the BEAM platform.

## Contributing

We welcome contributions! Please read our [contributing guidelines](CONTRIBUTING.md) before submitting pull requests.
