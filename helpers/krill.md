# Krill

Krill is the OpenClaw AI instance that functions as our support bot in **#help** and **#ideas**. It has full context of the OpenClaw docs and codebase, and answers support questions automatically.

## Hosting

- Hosted on [exe.dev](https://exe.dev) (Peter's account)
- Only **Shadow** has access

**If Krill goes down directly ping Shadow.**

## How Krill Works

Krill cannot see a user's system. Krill has access to the latest docs + codebase and only information it has is what you include in your message. This means vague questions like "it doesn't work" are almost impossible for Krill to help with.

## Before Posting to Krill

Check the docs first — this is usually the fastest way to solve your issue:
[FAQ: I'm stuck, what's the fastest way to get unstuck?](https://docs.openclaw.ai/help/faq#im-stuck-whats-the-fastest-way-to-get-unstuck)

The recommended approach is to use a **local AI agent** that can see your machine and setup, which is often far more effective than asking in chat.

If you still need help, include:

1. **What you're trying to do** (goal / expected behaviour)
2. **What happened instead** (exact error message)
3. **What you ran or clicked** (commands, config snippet, etc.)
4. **Your environment** (OS, install method, versions)
5. **Relevant logs** (smallest useful snippet)

Posts like "it doesn't work" without details are almost impossible to debug.

## Tips for Getting Good Answers

- **Be specific.** Krill performs best with clear, detailed questions.
- **Paste error messages.** Krill can parse and diagnose errors quickly when given the exact text.
- **Include code snippets.** If your question involves config or code, paste the relevant section.
- **One question per message.** Splitting complex issues into focused questions yields better results.
- **Follow up with context.** If Krill's first answer doesn't solve your problem, reply with what you tried and what happened.
- **One Problem per thread.** If your initial question/issue is resolved, mark it as completed please and create a new thread for any new issues.


## Limitations

- Krill **cannot** access your filesystem, terminal, or browser.
- Krill **cannot** run code or commands on your behalf.
- Krill's knowledge is based on the docs and codebase — it may not know about very recent changes that haven't been synced yet.
- For issues that require seeing your system state, a local AI agent is a better tool.
