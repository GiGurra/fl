# Why Fluffy?

Or: problems this language might solve if it existed.

## The Billion Dollar Mistake (Nulls)

Most languages have null/nil/None. It causes crashes everywhere.

**Fluffy's approach:** Union types make optionality explicit.

```fluffy
// No null. Use union types.
findUser(id: int) {found: User} | {notFound: true} {
    // ...
}

// Caller must handle both cases
match findUser(42) {
    {found: user} -> greet(user)
    {notFound: _} -> println("User not found")
}
```

You literally cannot forget to handle the "not found" case.

## The Stringly Typed Problem

Too much code passes strings around when it should use proper types.

**Fluffy's approach:** Structural typing makes creating types trivial.

```fluffy
// Instead of passing string status around
// Just use a constraint
where running = .status == "running"
where stopped = .status == "stopped"

// Now the compiler enforces valid states
start(machine stopped) machine running { ... }
stop(machine running) machine stopped { ... }

// Can't call stop on a stopped machine - compile error!
```

## The Exception Spaghetti

Try/catch blocks everywhere. Errors caught and re-thrown. Nobody knows what can fail.

**Fluffy's approach:** No exceptions. Panics crash. Handle errors explicitly with union returns.

```fluffy
// Clear error handling
readFile(path: string) {content: string} | {error: string} {
    // ...
}

// Unrecoverable? Just panic. Let supervisor handle it.
panic("Database connection lost")
```

## The Mutable State Nightmare

Shared mutable state causes race conditions, hard-to-trace bugs, and sleepless nights.

**Fluffy's approach:** Everything is immutable. State transitions create new values.

```fluffy
// No mutation possible
account = {balance: 100}
newAccount = account with {balance: account.balance + 50}

// Original unchanged
// No race conditions
// No "who changed this?"
```

## The "Works in Dev, Crashes in Prod"

Runtime type errors, missing fields, unexpected nulls...

**Fluffy's approach:** Compile-time constraints catch issues before deployment.

```fluffy
// If this compiles, the constraint is satisfied
processOrder(order where .items.length > 0 && .total > 0) {
    // Guaranteed non-empty order with positive total
    // No runtime checks needed
}
```

## The Concurrency Chaos

Threads, locks, deadlocks, race conditions...

**Fluffy's approach:** Actor model with supervision (like Erlang).

```fluffy
// Isolated processes, message passing
worker = spawn_typed(processItems)
worker <- {items: myItems}

// Crashes are handled by supervisors, not try/catch
```

## The Honest Answer

Would Fluffy actually solve these problems better than existing languages?

**Maybe.** The ideas are borrowed from languages that do solve these problems:

- Structural typing: TypeScript, Go
- Compile-time constraints: Zig, Idris, Agda
- Immutability: Haskell, Clojure
- Actor model: Erlang, Elixir
- Union types: Rust, OCaml

Fluffy is really just "what if we combined the nice parts?"

Whether that combination works well... we'd need a compiler to find out.
