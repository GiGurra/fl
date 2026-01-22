# Concurrency

Fluffy uses an actor model inspired by Erlang/BEAM.

!!! note "Extra Imaginary"
    The concurrency model is even more hand-wavy than the rest of Fluffy.
    Consider this "vibes-based concurrency design."

## Processes

Processes are lightweight, isolated units of execution. Think goroutines or Erlang processes, not OS threads.

```fluffy
// Spawn a new process
pid = spawn_typed(myFunction)

// The process runs independently
// It has its own memory, can't access ours
```

## Message Passing

Processes communicate by sending messages.

```fluffy
// Send a message
pid <- {type: "greet", name: "Alice"}

// Receive messages (in the spawned process)
receive {
    {type: "greet", name} -> println("Hello, " + name)
    {type: "bye"} -> println("Goodbye!")
    _ -> println("Unknown message")
}
```

## Typed Spawning

`spawn_typed` knows what messages the process accepts.

```fluffy
// Function with known message type
handler(msg: {type: string, data: int}) {
    match msg {
        {type: "add", data} -> // ...
        {type: "sub", data} -> // ...
    }
}

pid = spawn_typed(handler)

// Compiler knows valid message types
pid <- {type: "add", data: 5}   // OK
pid <- {type: "wat", data: 5}   // Possibly OK (matches shape)
pid <- {wrong: "shape"}          // ERROR: wrong message type
```

## Let It Crash

No try/catch. If something goes wrong, the process crashes.

```fluffy
process() {
    data = riskyOperation()  // Might panic
    // If it panics, this process dies
    // But other processes are unaffected
}
```

## Supervision

Supervisors monitor processes and handle crashes.

```fluffy
// Conceptual - syntax TBD
supervisor = spawn_supervisor({
    strategy: "one_for_one",
    children: [
        {id: "worker1", start: worker1Function},
        {id: "worker2", start: worker2Function},
    ]
})

// If worker1 crashes, supervisor restarts it
// worker2 keeps running
```

Supervision strategies (borrowed from Erlang):

| Strategy | Behavior |
|----------|----------|
| `one_for_one` | Restart only the crashed process |
| `one_for_all` | Restart all children if one crashes |
| `rest_for_one` | Restart crashed process and all started after it |

## Why Actors?

### Isolation

Each process has its own memory. No shared state means no race conditions.

### Fault Tolerance

Crashes are contained. One bad process doesn't bring down the system.

### Distribution

Message passing works the same locally or across network. (In theory.)

### Simplicity

No locks, mutexes, semaphores. Just send and receive.

## The Catch

Actor model concurrency is powerful but has its own complexities:

- Message ordering
- Mailbox overflow
- Deadlocks (yes, still possible)
- Debugging distributed state

But those are problems for the hypothetical implementation team.

## Example: Worker Pool

```fluffy
// Worker that processes jobs
worker() {
    receive {
        {job: data, replyTo: pid} -> {
            result = process(data)
            pid <- {result: result}
            worker()  // Loop
        }
        {shutdown: true} -> {
            println("Worker shutting down")
        }
    }
}

// Spawn pool
workers = [spawn_typed(worker) for _ in range(10)]

// Distribute work
for job in jobs {
    workers[random(10)] <- {job: job, replyTo: self()}
}
```

Elegant? Perhaps. Real? Not yet.
