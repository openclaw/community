# Moderation Reference

Use this as a practical reference when a conversation needs intervention. Start with help, explanation, or redirection when that is enough.

## Approved Moderation Tools

OpenClaw uses Barnacle, a whitelabel instance of [Sapphire](https://sapph.xyz), for Discord moderation and moderation records.

Common commands:

- `/warn`: records a warning without restricting the user's ability to participate.
- `/mute`: temporarily times out a user while they can still read the server.
- `/ban`: removes a user and prevents them from rejoining for the selected duration.
- `/caselist`: shows a user's moderation history.

Use a clear reason and include the relevant message or evidence. If the action relates to voice chat, use `/report-vc` whenever possible so the incident has a record.

Use Barnacle rather than native Discord moderation tools when Barnacle is available. Keeping actions in one system makes context and review easier.

## Choosing a Response

- Explain or redirect when the issue appears accidental.
- Use a DM when a private conversation is more appropriate.
- Remove content when it is harmful, unsafe, or clearly disruptive.
- Ask another team member for help when the situation is unclear.
- Escalate scams, doxxing, threats, serious harassment, repeated abuse, and incidents involving staff.

There is no need to argue with someone who is determined to disrupt the community. Make the situation safe, record what matters, and return attention to users who need help.

## Help in Chat

Hermit provides `/say` commands for common redirects. See the [Hermit command reference](https://github.com/openclaw/hermit/blob/main/src/commands/say.ts).

The Server Guide at the top of the channel list contains common questions about the Discord server.
