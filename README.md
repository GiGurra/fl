# Fluffy-Lang (fl)

A modern programming language for the BEAM (Erlang VM) emphasizing structural typing, compile-time constraints, and immutable data. Fluffy-lang combines the flexibility of structural typing with the safety of static constraints and the robustness of the BEAM platform.

## Core Concepts

### Structural Typing

Everything in Fluffy-lang is defined by its shape, not its name. Types and constraints are inferred from structure, with names serving as optional hints.

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
expanded: Person+ = {name: "Bob", isCalm: true, age: 25, extra: "field"}
println(expanded.extra)  // "field", compiles and works
```

### Union Types

Fluffy uses union types for representing different variants, similar to Erlang's tagged tuples:

```fluffy
Message = 
    | {type: "text", content: string}
    | {type: "image", url: string, size: int}
    | {type: "error", code: int, message: string}

processNumber(n: int): {ok: int} | {error: string} {
    if n < 0 {
        return {error: "number cannot be negative"}
    }
    return {ok: n * 2}
}

// Pattern matching on unions
match msg {
    {type: "text", content} => handleText(content)
    {type: "image", url, size} => handleImage(url, size)
    {type: "error", code, message} => logError(code, message)
}
```

### Constraints

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

The compiler can infer relationships between constraints:

```fluffy
comptime proof(int with _ < limit0, int with _ < limit1) bool {
    return limit0 < limit1
}

myNumber: int with _ < 50 = 42
myNumber: int with _ < 51 = myNumber  // Works because 50 < 51
```

### Runtime Constraint Verification

Comptime proofs can be used at runtime to verify data:

```fluffy
where shortString = string with _.length <= 10

someString = readFromOutsideWorld()
if !someString is shortString {
    return {error: "string too long"}
}
// someString is now known to be a short string
processShortString(someString)
```

### Immutable State Transitions

All data is immutable. State changes create new values using the `with` operator:

```fluffy
calmDown(angry d) -> calm {
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
pid = spawn(someFunction)

// Send message
pid <- message

// Receive messages
receive {
    {type: "hello"} => handleHello()
    {type: "bye"} => handleBye()
    _ => handleDefault()
}
```

### Modules and Imports

```fluffy
// Importing modules
import {Http, Json} from "std/web"
import Math from "std/math"

// Exporting definitions
export {
    Person,
    process
}
```

### Pattern Matching

First-class support for pattern matching with constraints:

```fluffy
process(msg) {
    match msg {
        {command: "start"} where .value > 0 => 
            msg with {status: "running"}
        {command: "stop"} => 
            msg with {status: "stopped"}
        m where .isCalm => 
            handleCalm(m)
        _ => 
            msg with {error: "invalid command"}
    }
}
```

## Why Fluffy-Lang?

- **Zero Runtime Overhead for Constraints**: All constraints are checked at compile-time
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

start(s: State where stopped): {ok: State where running} | {error: string} {
    if s.value < 0 {
        return {error: "invalid starting value"}
    }
    
    return {ok: s with {
        status: "running",
        lastStart: timestamp()
    }}
}

stop(s: State where running): State where stopped {
    s with {status: "stopped"}
}
```

## Current Status

⚠️ Fluffy-lang is currently in early development. The syntax and features are subject to change.

First experimental compiler to be written in Go, targeting the BEAM platform.

## Contributing

We welcome contributions! Please read our [contributing guidelines](CONTRIBUTING.md) before submitting pull requests.
```

This revised README:
1. Better emphasizes the BEAM connection
2. Incorporates union types
3. Shows more consistent syntax
4. Includes modules and concurrency
5. Provides more complete examples
6. Has a clearer structure

Would you like me to explain any part in more detail or make further adjustments?
