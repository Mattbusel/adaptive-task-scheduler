
# Adaptive Task Scheduler (Rust + Tokio)

A proof-of-concept runtime for **dependency-aware, forced task promotion** in Rust using `tokio`.

This prototype demonstrates how async I/O tasks can be dynamically **reclassified and promoted to CPU-bound threads** when their dependencies are blocking higher-priority tasks, a technique inspired by work-stealing schedulers and adaptive runtimes.

---

##  Why This Matters

Modern async runtimes (like `tokio`) treat I/O and CPU-bound tasks differently, often strictly. But in real-world applications, **task importance and dependencies change over time**. This project introduces a runtime that can:

- Detect when a dependent task is stalling
- Promote a blocking I/O task to a CPU thread (`tokio::spawn_blocking`)
- Simulate task stealing and priority bumping

Think of it as a step toward a **self-optimizing, latency-aware task graph engine**.

---

##  Features

- Task registration with dependencies
- Async I/O task simulation with `tokio::sleep`
- Blocking detection via dependency monitoring
-  Dynamic reclassification from I/O to CPU
-  Logging to trace execution flow

---

##  Architecture Overview

```text
+------------------+
| Task Manager     | ◄── Registers all tasks and dependencies
+------------------+
        │
        ▼
+--------------------------+
| Tokio Runtime (I/O pool) |
| - Runs default tasks     |
+--------------------------+
        │
[Waits too long?] → promote
        │
        ▼
+---------------------------+
| CPU-bound Thread Pool     |
| - Runs promoted tasks     |
+---------------------------+
````

Each task:

* Has a unique ID
* Knows what tasks it depends on
* Can be upgraded from an I/O future to a blocking operation if needed

---

##  Crates Used

* [`tokio`](https://crates.io/crates/tokio) — Async runtime
* [`dashmap`](https://crates.io/crates/dashmap) — For concurrent task registry
* (Optional) [`uuid`](https://crates.io/crates/uuid) — For unique task IDs

---

##  Running the Example

```bash
git clone https://github.com/Mattbusel/adaptive-task-scheduler.git
cd adaptive-task-scheduler
cargo run
```

You should see logs like:

```
[spawn] Task A started (depends on Task B)
[spawn] Task B started
[wait]  Task A still waiting on B... triggering promotion
[promote] Task B moved to CPU thread
[complete] Task B done
[complete] Task A done
```

---

##  Ideas for Expansion

* DAG-based dependency scheduler
* Prioritized task queues
* Real-time load metrics (CPU / I/O saturation)
* LLM inference server that adapts execution paths based on queue pressure
* Integration with `loom` to test concurrency correctness

---

##  Inspiration

* Erlang’s scheduler and supervision model
* Google's Borg runtime
* Rayon’s work-stealing and task forking
* Real-time OSes and priority inheritance

---

##  Author

Created by Matthew Busel.
If you like weird schedulers, async runtimes, or cosmic LLM designs, let's talk.

---

##  License

MIT License

```




```

adaptive-task-scheduler/
├── src/
│   └── main.rs
├── Cargo.toml
└── README.md


