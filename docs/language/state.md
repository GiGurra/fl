# State & Immutability

All data in Fluffy is immutable. State changes create new values.

## The `with` Operator

```fluffy
person = {name: "Alice", age: 30, mood: "happy"}

// Create a new value with updated fields
olderPerson = person with {age: 31}
// Result: {name: "Alice", age: 31, mood: "happy"}

// Original is unchanged
println(person.age)  // 30
```

## Multiple Updates

```fluffy
// Update multiple fields at once
updated = person with {age: 31, mood: "excited"}

// Chain updates (each creates a new value)
result = person
    with {age: 31}
    with {mood: "excited"}
    with {name: "Alice Smith"}
```

## State Machines

Constraints make state machines type-safe.

```fluffy
where draft = .status == "draft"
where published = .status == "published"
where archived = .status == "archived"

// Can only publish drafts
publish(post draft) post published {
    post with {status: "published", publishedAt: now()}
}

// Can only archive published posts
archive(post published) post archived {
    post with {status: "archived", archivedAt: now()}
}

// Compile errors:
publish(publishedPost)  // ERROR: expected draft
archive(draftPost)      // ERROR: expected published
```

## Why Immutability?

### No Spooky Action at a Distance

```fluffy
// In mutable languages:
processA(data)
processB(data)  // Did processA change data? Who knows!

// In Fluffy:
dataA = processA(data)
dataB = processB(data)  // data is unchanged, guaranteed
```

### Thread Safety for Free

```fluffy
// Safe to share across processes
sharedConfig = {timeout: 30, retries: 3}

spawn { useConfig(sharedConfig) }
spawn { useConfig(sharedConfig) }
spawn { useConfig(sharedConfig) }

// No locks needed - it can't change
```

### Time Travel Debugging

```fluffy
// Keep history trivially
history = []
current = initialState

// On each action:
history = history ++ [current]
current = current with {/* updates */}

// "Undo" is just:
current = history.last
```

## Nested Updates

Updating nested structures:

```fluffy
company = {
    name: "Acme",
    address: {
        city: "Springfield",
        zip: "12345"
    }
}

// Update nested field
moved = company with {
    address: company.address with {city: "Shelbyville"}
}
```

Yes, this is verbose. A real implementation might have syntax sugar:

```fluffy
// Hypothetical syntax for deep updates
moved = company with {address.city: "Shelbyville"}
```

## Performance?

"But creating new objects all the time is slow!"

Structural sharing and persistent data structures make this efficient. The new object shares unchanged parts with the old.

```
Before: {a: 1, b: 2, c: 3}
                  │
After:  {a: 1, b: 2, c: 4}
         │     │     │
         └──shared──┘  new
```

But since there's no implementation, performance is theoretically infinite.
