# Error Handling

Fluffy has no exceptions. It has panics and union types.

## The Two Kinds of Errors

### Recoverable Errors

Use union types. The caller decides what to do.

```fluffy
readFile(path: string) {content: string} | {error: string} {
    if !fileExists(path) {
        early_return {error: "file not found: " + path}
    }
    {content: actuallyReadFile(path)}
}

// Caller must handle both cases
match readFile("config.json") {
    {content: data} -> parseConfig(data)
    {error: msg} -> {
        println("Failed: " + msg)
        useDefaults()
    }
}
```

### Unrecoverable Errors

Use panics. The process crashes. A supervisor handles it.

```fluffy
// This should never happen in correct code
// If it does, crash immediately
assert(denominator != 0, "division by zero")
result = numerator / denominator

// Or explicit panic
if corruptedState {
    panic("state corruption detected")
}
```

## No Try/Catch

There is no way to catch a panic. This is intentional.

```fluffy
// This does NOT exist in Fluffy:
try {
    riskyOperation()
} catch (e) {
    // nope
}
```

Why? Because:

1. **Exceptions hide control flow** - You can't tell what might throw
2. **Catch-all handlers mask bugs** - `catch (e) { log(e) }` hides problems
3. **Error handling becomes optional** - Easy to forget, hard to audit

## Union Types for Errors

Explicit, type-checked, impossible to ignore.

```fluffy
divide(a: int, b: int) {ok: int} | {error: string} {
    if b == 0 {
        early_return {error: "cannot divide by zero"}
    }
    {ok: a / b}
}

// This won't compile - must handle error case
result = divide(10, 0).ok  // ERROR: might be error variant

// This compiles
match divide(10, 0) {
    {ok: value} -> println(value)
    {error: msg} -> println("Error: " + msg)
}
```

## Early Return

The `early_return` keyword exits the function with a value.

```fluffy
process(data) {ok: Result} | {error: string} {
    if !data.isValid {
        early_return {error: "invalid data"}
    }

    parsed = parse(data)
    if parsed is {error: e} {
        early_return {error: "parse failed: " + e}
    }

    {ok: transform(parsed.value)}
}
```

No regular `return` keyword - the last expression is the return value.

## Panic vs Error

| Use Panic When | Use Error When |
|----------------|----------------|
| Bug in the code | Expected failure mode |
| Impossible state reached | User provided bad input |
| Invariant violated | External resource unavailable |
| Programming error | Business logic rejection |

```fluffy
// Panic: this is a bug
items = getItems()
if items.length == 0 {
    panic("getItems returned empty - this should never happen")
}

// Error: this is expected
validateEmail(input: string) {ok: Email} | {error: string} {
    if !input.contains("@") {
        early_return {error: "invalid email format"}
    }
    {ok: Email(input)}
}
```

## Supervisors Handle Panics

When a process panics, its supervisor decides what to do:

- Restart the process
- Restart related processes
- Escalate to parent supervisor
- Shut down gracefully

```fluffy
// Process crashes
worker() {
    data = fetchData()  // panics if network down
    process(data)
}

// Supervisor restarts it
// After 3 crashes in 1 minute, escalate
// Default: crash the application
```

## Defer for Cleanup

`defer` runs at end of scope, regardless of how it exits.

```fluffy
process() {
    file = openFile("data.txt")
    defer {
        closeFile(file)
    }

    // Even if we panic here, file gets closed
    riskyOperation(file)
}
```

## The Philosophy

> "Let it crash" - Joe Armstrong, Erlang creator

Instead of trying to handle every possible error inline, let processes crash and have supervisors manage recovery. This leads to simpler code and more robust systems.

At least in theory. In practice... we'd need to build it to find out.
