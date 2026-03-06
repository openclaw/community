# Helper Tools

This is a reference of the tools available to the Helper team.

## Krill (Support Bot)

OpenClaw AI instance in **#help** and **#ideas** that answers support questions automatically using the full docs and codebase context.

See [krill.md](krill.md) for detailed usage and guidelines.

## Hermit Commands

Hermit is our custom bot ([source](https://github.com/openclaw/hermit)). It provides three command groups:

### `/helper` — Thread moderation utilities

Commands for managing help threads. These also fire a webhook for tracking.

| Command | What it does |
|---|---|
| `/helper warn-new-thread` | Warns that the thread is too long and should be split. Asks Krill to summarize the conversation. Optionally mention a user. |
| `/helper close` | Close and lock the current thread (shorthand). Only works inside a thread. |
| `/helper close-thread` | Posts a close notice, then archives and locks the thread. Only works inside a thread. |

Hermit also automatically posts a **welcome message** when a new thread is created in the help channel, reminding users to include details (goal, error, commands, environment, logs). See [welcome-message.md](welcome-message.md) for the full template.

## Answer Overflow

Indexes **#help** threads for SEO so that common questions are discoverable via search engines. Managed at [answeroverflow.com](https://answeroverflow.com).