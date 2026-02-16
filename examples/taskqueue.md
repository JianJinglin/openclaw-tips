# 📋 Example TASKQUEUE.md

A simple task queue pattern for managing async work. Drop in your workspace.

```markdown
# TASKQUEUE.md

## In Progress
- [ ] #3 Research vector database options for the RAG pipeline
  - Started: 2025-02-15
  - Notes: Comparing Pinecone, Weaviate, Chroma

## Queued
- [ ] #4 Write unit tests for auth module
- [ ] #5 Update API docs with new endpoints
- [ ] #6 Set up monitoring alerts for production

## Completed
- [x] #1 Set up CI/CD pipeline — Done 2025-02-14
- [x] #2 Migrate database to PostgreSQL — Done 2025-02-15

## Blocked
- [ ] #7 Deploy to staging — Blocked on: #4 (needs tests first)
```

## How to Use This Pattern

### With Heartbeats

Add to your HEARTBEAT.md:

```markdown
- [ ] Check TASKQUEUE.md — pick up next queued task if nothing is in progress
```

The agent will automatically work through your queue during heartbeats.

### With Sub-Agents

For parallelism, the agent can spawn sub-agents for independent tasks:

```
Agent reads TASKQUEUE.md →
  Spawns sub-agent for #4 (tests)
  Spawns sub-agent for #5 (docs)
  Both run in parallel, announce when done
  Agent updates TASKQUEUE.md with results
```

### Manual Workflow

Just tell your agent:

> "Add 'refactor the payment module' to the task queue"
> "What's the next task? Start working on it"
> "Mark #3 as complete"

## Tips

- **Keep it flat** — avoid complex priority systems. Ordered list = priority.
- **One "In Progress" at a time** — prevents context-switching
- **Include context** — notes, links, and blockers help the agent pick up where you left off
- **Use with memory** — the agent logs progress in daily memory files, task queue is the overview
- **Clean up regularly** — move completed items to a separate archive after a week
