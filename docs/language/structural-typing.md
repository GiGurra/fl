# Structural Typing

In Fluffy, types are defined by their shape, not their name.

## Basic Shapes

```fluffy
// A value with its inferred type
person = {name: "Alice", age: 30}

// Explicit type annotation (optional)
person: {name: string, age: int} = {name: "Alice", age: 30}

// Named type alias (just a shorthand)
Person = {name: string, age: int}
person: Person = {name: "Alice", age: 30}
```

All three are equivalent. The name `Person` is just documentation.

## Duck Typing, But Static

If two types have the same shape, they're the same type.

```fluffy
Point = {x: int, y: int}
Coordinate = {x: int, y: int}
Vector2D = {x: int, y: int}

// These are all the same type
p: Point = {x: 1, y: 2}
c: Coordinate = p       // Works!
v: Vector2D = c         // Works!
```

## Extra Fields Are Fine

A value can have more fields than the type specifies.

```fluffy
Person = {name: string, age: int}

// Extra 'email' field is allowed
employee: Person = {name: "Bob", age: 25, email: "bob@company.com"}

// You can access it
println(employee.email)  // "bob@company.com"

// But unknown fields are compile errors
println(employee.phone)  // ERROR: field 'phone' does not exist
```

This enables gradual typing and extensibility.

## Function Parameters

Functions accept any value that matches the shape.

```fluffy
greet(p: {name: string}) {
    println("Hello, " + p.name)
}

// All of these work
greet({name: "Alice"})
greet({name: "Bob", age: 30})
greet({name: "Charlie", email: "c@test.com", isAdmin: true})
```

Only `name: string` is required. Extra fields are ignored.

## Union Types

Multiple possible shapes combined.

```fluffy
Result = {ok: int} | {error: string}

divide(a: int, b: int) Result {
    if b == 0 {
        early_return {error: "division by zero"}
    }
    {ok: a / b}
}

// Must handle both variants
match divide(10, 2) {
    {ok: value} -> println("Result: " + value)
    {error: msg} -> println("Error: " + msg)
}
```

## Type Inference

Types are usually inferred from usage.

```fluffy
// Compiler infers the parameter and return types
double(x) {
    x * 2
}

// From usage, it knows x must support multiplication
// and returns whatever x * 2 returns

result = double(5)   // result: int
result = double(3.14) // result: float
```

## The Philosophy

Structural typing means:

- Less boilerplate (no interface declarations)
- More flexibility (any matching shape works)
- Better composition (combine shapes freely)
- Duck typing, but with compile-time safety

It's like TypeScript's structural typing, but as the only option.
