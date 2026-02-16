# HEARTBEAT.md

## Instructions
1. Read TASKQUEUE.md
2. If NOW has a task → continue it
3. If NOW is empty → move top NEXT item to NOW → start it
4. If nothing to do → HEARTBEAT_OK

## Constraints
- Night mode (11pm-8am): silent work only
- Max 3 unsolicited messages per hour
- Use sub-agents for heavy tasks
- Update TASKQUEUE.md after completing work
