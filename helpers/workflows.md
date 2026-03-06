# Helper Workflows

Common workflows and situations that helpers encounter.

## When a Thread Goes Off-Topic

1. Politely redirect the conversation back to the original question.
2. If it continues, use `/helper warn-new-thread` to ask participants to move to a new thread.
3. If the thread is beyond recovery, use `/helper close-thread` to archive and lock it.
4. If the original question is still unanswered, re-surface it in a new thread after locking.

## When a Thread Gets Too Long

1. Use `/helper warn-new-thread` to warn users the thread is too long. This also asks Krill to summarize the conversation so far.
2. If the thread continues or spans multiple topics, use `/helper close-thread` to post a close notice and archive/lock it.
3. Ask the user to open a new thread with a clean summary including:
   - The original goal
   - What has been tried
   - The current error or blocker
4. Link the old thread in the new one for context.

## When Someone Asks for Unrelated Help

If someone asks for help that is **not related to OpenClaw** (e.g., general programming questions, other tools/products):

1. Be polite.
2. Let them know that **#help** is specifically for OpenClaw support.
3. Close/Lock the thread please.

## Escalation

- **Recurring issue/bug:** If the same issue keeps coming up, let your Team Lead and/or Admin know.
- **Krill is down:** Ping Shadow( Admin) directly.
- **Security issue:** Escalate to Shadow (Admin) immediately.

## Thread Hygiene

- When a question is resolved, use the discord context menu to mark the message as answer. 
- One issue per thread — if a new issue arises, use `/helper warn-new-thread` to ask for a new thread.
- Keep threads focused so that Answer Overflow can index them cleanly for others to find later.
