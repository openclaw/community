# Discord Moderation

OpenClaw uses Barnacle (723205682911903774) to manage its moderation needs. It’s a whitelabel instance of [Sapphire](https://sapph.xyz).

## Moderation Team Structure

See [Moderation Organization](moderation-organization.md) for team layout, leads, and membership rules.

Here are some common commands you would use to moderate the server.

- `/ban` - This permanently (or temporarily) removes a user from the server and prevents them from joining back. They will not be able to speak or read messages. You must specify a user and a reason, and a duration is optional (the user will be unbanned after the time ends), otherwise it will default to permanent.
- `/mute` - Times out a user. They will not be able to speak for the duration of the timeout but will still be able to read messages. You must specify a user and reason, and it’s recommended to specify a duration, otherwise it will default to an hour.
- `/warn` - This does not impair the users ability to talk in the server. It’s simply a mark on their record. More of a “stop doing that” before escalating to a mute or ban. Requires a user and reason.
- `/caselist` - This lets you see a user's history of warns/mutes/bans from different moderators.

When you're moderating, please keep in mind that the user can see everything you put in except for the mod notes section.

- Reason = the main reason they see and that's logged in previews like `/caselist`
- Verified Proof = the message that was reported
- Proof = anything you type in as evidence

## Github Moderation

Some users here have access to moderate users on Github, maintainers or community staff can ping them for situations on Github in #github-moderation, and full guidelines are there.

## Appeals

Users can appeal their mutes and bans by going to https://appeal.gg/clawd.

Community Staff can review these either in the #appeal channel or by going to https://dashboard.appeal.gg/1456350064065904867/submissions

They can also report you to admins on that same form.

Appeals should be handled by someone other than the originating mod. If you issued the action, do not review that appeal, ask another mod or your lead/admin to handle it.

When you accept or deny the appeal, the user will see the reason you put in, in the format like `Your appeal was accepted. ${reason}`

## Working in Chat

Barnacle has a set of `/say` commands to help with redirecting users in chat.
You can find all of those here: [https://github.com/openclaw/barnacle/blob/main/src/commands/say.ts](https://github.com/openclaw/barnacle/blob/main/src/commands/say.ts)

Additionally, the Server Guide has a large amount of common questions about the Discord server itself. This can be found at the top of the channel list.

## Activity Expectations

Stay reasonably active and engaged with the community. This can include sending messages, being present in voice, answering support questions, or participating in mod discussions.

### Inactivity / Leave of Absence (LOA)

If you will be away for multiple days or unable to complete your duties, post an inactivity log in your team channel and ping your lead, so leads can reassign coverage. There is no hard limit to LOA length, but please avoid radio silence.

Format:
- Dates: 12th June 2024 - 1st July 2024
- Reason: Optional. If you don’t want to share publicly, DM a lead or the Admin.

Consider muting server notifications while you are away.

## Biggest Rules

If you didn't read anything else, read this:

- When in doubt, mute for 1 hour with the phrase "pending moderation". Do not ban users instantly.
- If you are unsure, ping online moderators in #vc-mods or #moderators and ask them for help. (VC Mods, you can ping the regular mods too)
- If you're doing a voice moderation, you must (if at all possible) use the /report-vc too so that we have a log for if/when they appeal or complain in my DMs
- DO NOT use the native moderation tools. Always use the Barnacle commands, because otherwise we have no transparency and they don't get any information about why they were moderated.
