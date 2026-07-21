# Group Member Removal Bot — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A Telegram bot that lets authorized group admins and moderators remove members via the /remove @user command, silently removing the tagged user without public confirmation.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Telegram group admins
- Telegram group moderators

## Success criteria

- Admins can remove disruptive members in seconds
- Bot silently removes users without spamming the group chat
- Bot correctly rejects /remove commands from non-admins
- Bot correctly handles @username, @mention, and user ID formats

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/remove** (command, actor: user, command: /remove) — Remove a tagged user from the group
- **/start** (command, actor: user, command: /start) — Open the main menu

## Flows

### Remove user
_Trigger:_ /remove @user

1. User types /remove @username in the group
2. Bot checks if remover has admin/moderator permission
3. Bot removes the tagged user silently
4. Bot silently removes the user without sending a public confirmation message

_Data touched:_ User, Group, Permission

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: none)_ — Telegram user (remover or removed)
  - fields: user_id, username, first_name, last_name
- **Group** _(retention: none)_ — Telegram group where removals occur
  - fields: chat_id, title
- **Permission** _(retention: none)_ — Admin/moderator role required to run /remove
  - fields: user_id, chat_id, role

## Integrations

- **Telegram** (required) — Bot API messaging and group management
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Add bot to group as admin
- Grant admin/moderator permission to authorized users
- Revoke admin/moderator permission from unauthorized users

## Permissions & privacy

- Bot requires admin permission in the group to remove members
- Bot does not store any user data permanently
- Bot does not log or audit removals

## Edge cases

- Bot does not have admin permission in the group
- User is not a member of the group
- User is the bot itself
- User is the group creator
- User is already removed
- User is not found by @username
- User is not found by @mention
- User is not found by user ID

## Required tests

- Admin can remove a member via /remove @username
- Admin can remove a member via /remove @mention
- Admin can remove a member via /remove user_id
- Non-admin cannot remove a member via /remove
- Bot does not send a public confirmation message after removal
- Bot does not remove the bot itself
- Bot does not remove the group creator
- Bot does not remove a user who is not a member
- Bot does not remove a user who is already removed
- Bot does not remove a user who is not found by @username
- Bot does not remove a user who is not found by @mention
- Bot does not remove a user who is not found by user ID

## Assumptions

- Silent removal: Bot removes the user without sending a public confirmation message — this is the standard Telegram bot behavior for group management and avoids spamming the group chat.
- Bot does not store any user data permanently
- Bot does not log or audit removals
- Bot does not have an automatic removal based on rule violations
- Bot does not have a user appeal or reinstatement workflow
- Bot does not have a multi-step approval process
