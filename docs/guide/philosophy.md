# Philosophy

The guiding principles behind Fluffy-Lang's design. Or what passes for design when there's no actual implementation.

## If It Compiles, It Works

The holy grail. Push as much verification as possible to compile time.

```fluffy
// The compiler knows this can never fail at runtime
process(data where .isValid && .size < 1000) {
    // We're guaranteed valid, reasonably-sized data
}
```

No runtime checks for things that can be proven statically.

## Shapes Over Names

Names are hints, not contracts. Structure is truth.

```fluffy
// These are the same type
{x: int, y: int}
Point = {x: int, y: int}
Coordinate = {x: int, y: int}

// A function accepting any of these
distance(p: {x: int, y: int}) { ... }
```

If it walks like a duck and quacks like a duck, it's a `{walks: bool, quacks: bool}`.

## Immutability by Default

Mutable state is the root of many bugs. So we don't have it.

```fluffy
person = {name: "Alice", age: 30}

// This doesn't mutate, it creates a new value
olderPerson = person with {age: 31}

// person is still {name: "Alice", age: 30}
```

## Let It Crash

From Erlang: instead of defensive programming everywhere, let processes crash and have supervisors handle recovery.

```fluffy
// No try/catch. If something goes wrong, crash.
data = parseInput(rawData)  // might panic

// A supervisor somewhere will restart us if needed
```

This simplifies code dramatically. No error handling spaghetti.

## Constraints Are Types

Traditional types answer "what shape is this data?"

Constraints answer "what properties does this data have?"

```fluffy
// Traditional type
x: int = 42

// With constraint
x: int with _ > 0 && _ < 100 = 42

// Named constraint
where percentage = int with _ >= 0 && _ <= 100
score: percentage = 85
```

Constraints are checked at compile time when possible, runtime when necessary.

## Explicit Over Magic

No hidden behavior. No implicit conversions. No spooky action at a distance.

```fluffy
// Want to use a module's function? Name it.
result = some_module:some_function(data)

// No global imports polluting namespace
// No "where did this function come from?"
```

## Expression-Based

Everything is an expression. Everything returns a value.

```fluffy
result = if condition {
    computeA()
} else {
    computeB()
}

status = match state {
    {running: true} -> "active"
    _ -> "idle"
}
```

No statements vs expressions distinction. It's expressions all the way down.

## The Uncomfortable Truth

These are nice principles. They might even be good principles.

But without an implementation, they're just words on a page.

That's okay. Sometimes exploring ideas is valuable even without building them. And if someone wants to build it... PRs welcome?
