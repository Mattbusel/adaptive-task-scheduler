#  Adaptive Task Scheduler (Rust + Tokio)

A proof-of-concept runtime for **dependency-aware, forced task promotion** in Rust using `tokio`.

This prototype demonstrates how async I/O tasks can be dynamically **reclassified and promoted to CPU-bound threads** when their dependencies are blocking higher-priority tasks — a technique inspired by work-stealing schedulers and adaptive runtimes.

---

##  Why This Matters

Modern async runtimes (like `tokio`) treat I/O and CPU-bound tasks differently — often strictly. But in real-world applications, **task importance and dependencies change over time**. This project introduces a runtime that can:

- Detect when a dependent task is stalling
- Promote a blocking I/O task to a CPU thread (`tokio::spawn_blocking`)
- Simulate task stealing and priority bumping

Think of it as a step toward a **self-optimizing, latency-aware task graph engine**.

---

##  Features

-  Task registration with dependencies
-  Async I/O task simulation with `tokio::sleep`
-  Blocking detection via dependency monitoring
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
