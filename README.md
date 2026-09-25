# Adaptive Task Scheduler (Rust + Tokio)

A design for a Tokio-based runtime that notices when an async task is holding up the tasks that depend on it, and promotes that task onto a dedicated thread so the chain unblocks.

> **Status: design notes only. There is no Rust code in this repository yet** (no `Cargo.toml`, no `src/`), so there is nothing to build or run. The sections below describe the intended prototype.

## Why

Async runtimes like Tokio handle I/O-bound futures on a small worker pool and expect CPU-heavy or blocking work to be moved off it explicitly with `spawn_blocking`. That decision is made once, when the code is written. In real systems, though, a task's importance changes: a slow, low-priority job can suddenly become the thing a latency-critical request is waiting on. This project explores a scheduler that makes that call at run time, based on the dependency graph.

It borrows from priority inheritance in real-time operating systems (a low-priority task holding a resource inherits the priority of the task waiting on it) and from work-stealing schedulers.

## Planned features

- Register tasks with explicit dependencies on other tasks
- Run tasks as ordinary Tokio futures by default (I/O simulated with `tokio::time::sleep`)
- Watch for dependents that have waited longer than a threshold
- Promote the blocking task to a dedicated thread via `tokio::task::spawn_blocking`
- Trace spawns, waits, promotions and completions in the log

## Intended architecture

```text
+------------------+
| Task Manager     | <-- registers tasks and their dependencies
+------------------+
        |
        v
+--------------------------+
| Tokio runtime (I/O pool) |
| - runs tasks by default  |
+--------------------------+
        |
  dependent waiting too long? -> promote
        |
        v
+---------------------------+
| Blocking thread pool      |
| - runs promoted tasks     |
+---------------------------+
```

Each task has a unique ID, knows which tasks it depends on, and can be moved from the async pool to a blocking thread when needed.

Planned crates: [`tokio`](https://crates.io/crates/tokio) for the runtime, [`dashmap`](https://crates.io/crates/dashmap) for a concurrent task registry, and optionally [`uuid`](https://crates.io/crates/uuid) for task IDs.

Intended layout:

```
adaptive-task-scheduler/
  Cargo.toml
  src/main.rs
  README.md
```

## What a first run should show

Once implemented, the demo is meant to print a trace along these lines (illustrative, not real output):

```
[spawn]    Task A started (depends on Task B)
[spawn]    Task B started
[wait]     Task A still waiting on B... triggering promotion
[promote]  Task B moved to blocking thread
[complete] Task B done
[complete] Task A done
```

## Open design questions

- A future that is already running cannot be moved to another thread mid-poll. Promotion probably means the task is written so it can be restarted or resumed on a blocking thread, or that "promotion" raises its priority in a custom queue instead.
- How to pick the wait threshold, fixed or adaptive to load.
- How to avoid promotion storms when many dependents stall at once.

## Ideas for later

- Full DAG scheduling with topological ordering
- Prioritized task queues
- Real-time CPU and I/O saturation metrics
- An LLM inference server that adapts execution paths under queue pressure
- Model checking with [`loom`](https://crates.io/crates/loom) for concurrency correctness

## Inspiration

- Erlang's scheduler and supervision model
- Google's Borg cluster manager
- Rayon's work stealing and task forking
- Real-time operating systems and priority inheritance

## Author

Matthew Busel. If you like odd schedulers and async runtimes, open an issue and say hello.
