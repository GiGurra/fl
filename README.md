# Fluffy-Lang (fl)

A modern programming language emphasizing structural typing, compile-time constraints, and immutable data. Fluffy-lang
combines the flexibility of structural typing with the safety of static constraints.

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

// Equivalent of the above
person: Person = {name: "Alice", isCalm: true}

// Extra fields are fine
expanded: Person with {extra: string} = {name: "Bob", isCalm: true, age: 25, extra: "field"}
```

### Comptime

Fluffy-lang is a comptime language, meaning all constraints are checked at compile-time. This allows for zero runtime
overhead and guarantees about the program's behavior.

The language also offers a `comptime` keyword to run code at compile-time.

```fluffy
comptime {
    // This fluffy code runs at compile-time
    x = 42
    y = x * 2
}
```

### Constraints

Constraints define compile-time verifiable conditions. They can be named for convenience but work based on structure.

```fluffy

// Using constraints
process(d where .isCalm) {
    // d is guaranteed to be calm here
}

// Constraints can be named too
where calm = .isCalm
where angry = !.isCalm

// Using constraints
process(d where calm) {
    // d is guaranteed to be calm here
}

if person is calm {
    // person is calm in this scope
}
```

```fluffy
myNumber = 42
myNumber: int with _ < 50 = 42
``` 

### Compile constraint extension and inference

Suppose you have verified a variable fulfills a constraint, for example, `int with _ < 50`.
We all know this also complies with `int with _ < 51`, but how would the compiler know this for any
logical constraints?

Enter comptime constraint extension and inference:

```fluffy
comptime proof can_infer(int with _ < limit0, int with _ < limit1) bool {
    return limit0 < limit1
}
comptime proof can_infer(comptime x int, int with _ < limit1) bool {
    return x < limit1
}
```

The compiler can at this point infer that `int with _ < 50` also complies with `int with _ < 51`, so you can write:

```fluffy
myNumber: int with _ < 50 = 42
myNumber: int with _ < 51 = myNumber
```

and call any functions requiring `int with _ < 51` with `myNumber` as an argument.

```fluffy
processNumber(n: int with _ < 51) {
    // n is guaranteed to be less than 51
}

// constant value known at comptime, the limit can be inferred
// using comptime proof can_infer(comptime x int, int with _ < limit1)
myNumber = 42

processNumber(myNumber)
```

### Immutable State Transitions

All data is immutable. State changes create new values using the `with` operator.

```fl
calmDown(angry d) -> calm {
    d with {isCalm: true}
}

// Preserves all other fields
person = {name: "Bob", mood: "grumpy", isCalm: false}
calmPerson = calmDown(person)
// Result: {name: "Bob", mood: "grumpy", isCalm: true}
```

### Expression-Based with Early Returns

Functions are expression-based with implicit returns, but support explicit returns for early exits.

```fl
processStuff(angry d) -> calm {
    if !validateInput(d) {
        return d with {error: "invalid input"}
    }

    match d {
        {value: v} where v < 0 => {
            if shouldReject(v) {
                return d with {error: "negative value rejected"}
            }
        }
        _ => {}
    }

    // Happy path - implicit return
    d with {isCalm: true}
}
```

### Pattern Matching

First-class support for pattern matching with constraints.

```fl
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

## Runtime environment

FLuffy runs on the erlang vm/beam, and is designed to be a functional language with a focus on immutability and
concurrency. There are no futures, mutable variables, etc. The ideas is to be simple and easy to reason about.

Just like Erlang, each process has its own heap and resource management, complements of the BEAM.

## Why Fluffy-Lang?

- **Zero Runtime Overhead for constraints**: All constraints are checked at compile-time
- **Type Safety Without Ceremony**: Structural typing with static guarantees
- **Immutable by Default**: Safe and predictable state management
- **Expression-Based**: Clean and functional style
- **Flexible Constraints**: Power of dependent types with simplicity of structural typing

### Type Hints and Aliases

Names come before types, and type aliases are optional conveniences.

```fluffy
// Basic variable declaration
d: {isCalm: bool} = {isCalm: true}

// Type aliases for readability
Person = {
    name: string,
    isCalm: bool,
    age: int
}

// These are equivalent
bob: Person = {name: "Bob", isCalm: true, age: 30}
bob: {name: string, isCalm: bool, age: int} = {name: "Bob", isCalm: true, age: 30}
bob = {name: "Bob", isCalm: true, age: 30}  // type inferred

// Constraints can be named too
where calm = self.isCalm
where angry = !self.isCalm

// Type alias with constraint
CalmPerson = Person where calm

// These are equivalent
process(p: CalmPerson) { ... }
process(p: Person where calm) { ... }
process(p: {name: string, isCalm: bool, age: int} where .isCalm) { ... }
```

### Example: State Machine

```fluffy
// Type aliases (optional)
State = {
    status: string,
    value: int
}

// These functions work with any matching shape, not just State
start(s: {status: string, value: int} where stopped) -> running {
    if s.value < 0 {
        return s with {error: "invalid starting value"}
    }
    
    s with {
        status: "running",
        lastStart: timestamp()
    }
}

// Using type alias for readability
stop(s: State where running) -> stopped = s with {status: "stopped"}

main() {
    // Type aliases make intent clear but aren't required
    machine: State = {status: "stopped", value: 42}
    
    // This works too
    machine2 = {status: "stopped", value: 42, extra: "field"}
    
    running = start(machine)
    stopped = stop(running)
}
```

## Current Status

⚠️ Fluffy-lang is currently in early development. The syntax and features are subject to change.

## Contributing

We welcome contributions! Please read our [contributing guidelines](CONTRIBUTING.md) before submitting pull requests.
