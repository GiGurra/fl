# Constraints

Constraints are compile-time predicates that restrict what values are valid.

## Basic Constraints

```fluffy
// Inline constraint
x: int with _ > 0 = 42

// The _ refers to the value being constrained
// This won't compile:
y: int with _ > 0 = -5  // ERROR: constraint not satisfied
```

## Named Constraints

```fluffy
// Define reusable constraints
where positive = int with _ > 0
where percentage = int with _ >= 0 && _ <= 100
where nonEmpty = string with _.length > 0

// Use them
score: percentage = 85
name: nonEmpty = "Alice"
```

## Structural Constraints

Constraints can reference fields.

```fluffy
// Constraint on a field
where calm = .isCalm
where angry = !.isCalm
where hasName = .name != ""

// Use in function signatures
greet(person where hasName) {
    println("Hello, " + person.name)
}
```

## Function Parameter Constraints

```fluffy
// Only accepts calm persons
meditate(p where calm) {
    println(p.name + " meditates peacefully")
}

// Only accepts angry persons, returns calm person
calmDown(p where angry) p where calm {
    p with {isCalm: true}
}
```

The compiler verifies these at call sites.

## Compile-Time Proofs

When the compiler can't automatically prove a constraint relationship, you provide a proof.

```fluffy
// Tell the compiler: if x < a and a < b, then x < b
comptime proof(int with _ < limit0, int with _ < limit1) bool {
    limit0 < limit1
}

x: int with _ < 50 = 42
y: int with _ < 100 = x  // Works! Compiler uses proof: 50 < 100
```

## Runtime Verification

When data comes from outside (user input, files, network), verify at runtime.

```fluffy
where validEmail = string with _.contains("@")

rawInput = readFromUser()

// Runtime check, compile-time type narrowing
if rawInput is validEmail {
    // Here, rawInput is known to be validEmail
    sendEmail(rawInput)
} else {
    println("Invalid email")
}
```

## Combining Constraints

```fluffy
where adult = .age >= 18
where hasLicense = .license != nil
where canDrive = adult && hasLicense

// Function requires combined constraint
rentCar(person where canDrive) {
    // Guaranteed: age >= 18 AND has license
}
```

## Constraints vs Types

| Types | Constraints |
|-------|-------------|
| What shape is this? | What properties does it have? |
| `{name: string, age: int}` | `.age >= 18` |
| Checked by structure | Checked by predicate |
| Always known | May need runtime check |

## The Dream

Imagine a world where:

```fluffy
// Array that's always sorted
where sorted = array with isSorted(_)

// Function that maintains sortedness
insert(arr sorted, value) arr sorted {
    // Compiler verifies the implementation maintains sorted order
}
```

Whether this is actually achievable... ask the imaginary compiler.
