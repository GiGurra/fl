# Fluffy-Lang (fl)

[![Docs](https://img.shields.io/badge/docs-GitHub%20Pages-blue)](https://gigurra.github.io/fl/)

A programming language that doesn't exist yet.

## What is this?

Fluffy-Lang is a thought experiment exploring what a language might look like if you combined:

- Structural typing (everything is a shape)
- Compile-time constraints (like Zig's comptime)
- Immutable data with state transitions
- BEAM-style actors and supervision
- No exceptions, only panics

There's no compiler. There's no runtime. Just ideas and vibes.

## A Taste

```fluffy
// Structural typing - types are shapes
person = {name: "Alice", isCalm: true}

// Compile-time constraints
where calm = .isCalm
where angry = !.isCalm

// Immutable state transitions
calmDown(d angry) {
    d with {isCalm: true}
}

// Pattern matching on union types
match msg {
    {type: "text", content} -> handleText(content)
    {type: "error", code} -> logError(code)
}
```

## Status

This is a thought experiment with lots of LLM-generated text. The syntax is made up. The features are aspirational. The compiler is imaginary.

But hey, it's fluffy.

## Documentation

See the [full documentation](https://gigurra.github.io/fl/) for more imaginary features.

## License

MIT (for the README, at least)
