# Help Thread Welcome Message

When a new thread is created in the help channel, Hermit automatically posts the following welcome message:

---

> Welcome to the help channel!
>
> Krill cannot see your system — it only knows what you tell it. The more details you include, the easier it is to help.
>
> If you haven't included it yet, please consider sending:
>
> What you're trying to do (goal / expected behaviour)
>
> What happened instead (exact error message)
>
> What you ran or clicked (commands, config snippet, etc.)
>
> Your environment (OS, install method, versions)
>
> Relevant logs (the smallest useful snippet)
>
> Posts like "it doesn't work" without details are very hard to debug.
>
> If new issues arise, please open a new thread/topic instead of continuing in this one. Keeping one issue per thread helps ensure answers stay accurate and makes it easier for others to find solutions later.

---

This template is defined in [hermit/src/events/threadCreateWelcome.ts](https://github.com/openclaw/hermit/blob/main/src/events/threadCreateWelcome.ts) and can be overridden via the `HELPER_THREAD_WELCOME_TEMPLATE` environment variable.
