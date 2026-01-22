# Fluffy-Lang

A programming language that doesn't exist yet.

!!! warning "Thought Experiment"
    There is no compiler. There is no runtime. There is no package manager.
    There is only this documentation and a dream.

## What is Fluffy-Lang?

Fluffy-Lang (fl) is an exploration of what a programming language might look like if you:

1. Took structural typing seriously
2. Made constraints a first-class citizen
3. Assumed all data is immutable
4. Borrowed Erlang's "let it crash" philosophy
5. Added just enough syntax sugar to make it pleasant

It's partially serious language design exploration, partially a joke, and entirely imaginary.

## The Elevator Pitch

```fluffy
// Types are just shapes
Person = {name: string, mood: string, isCalm: bool}

// Constraints are compile-time predicates
where calm = .isCalm
where angry = !.isCalm

// Functions can require constraints
calmDown(person angry) {
    person with {isCalm: true, mood: "relaxed"}
}

// The compiler proves this is valid
grumpyBob = {name: "Bob", mood: "grumpy", isCalm: false}
relaxedBob = calmDown(grumpyBob)  // compiles!

// This won't compile - Alice is already calm
happyAlice = {name: "Alice", mood: "happy", isCalm: true}
calmDown(happyAlice)  // ERROR: constraint 'angry' not satisfied
```

## Core Ideas

| Concept | Fluffy's Take |
|---------|---------------|
| Types | Shapes, not names |
| Constraints | Compile-time predicates on shapes |
| State | Immutable, transitions via `with` |
| Errors | Panics crash processes, supervisors handle it |
| Concurrency | BEAM-style actors |
| Pattern Matching | First-class, with constraints |

## Current Status

```
Compiler:     [                    ] 0%
Runtime:      [                    ] 0%
Std Library:  [                    ] 0%
Ideas:        [====================] 100%
Vibes:        [===================+] Fluffy
```

## Why "Fluffy"?

Because the language is soft, fuzzy around the edges, and doesn't really exist.

Also because naming things is hard and this name was available.

## Next Steps

- [Philosophy](guide/philosophy.md) - The thinking behind the fluff
- [Why Fluffy?](guide/why.md) - Problems this might solve (if it existed)
- [Structural Typing](language/structural-typing.md) - Everything is a shape
- [Constraints](language/constraints.md) - Compile-time superpowers
