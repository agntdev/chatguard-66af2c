# ChatGuard Group Moderation Bot — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

Telegram bot for automated moderation of group chats with welcome challenges, spam detection, admin commands, and audit logs. Blocks obvious spam with configurable thresholds, provides moderation tools for admins, and maintains action logs with explanations.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Telegram group owners
- Group administrators

## Success criteria

- Automatically blocks spam messages with configurable thresholds
- Provides admin commands for moderation actions
- Maintains audit logs with explanations for all automated and admin actions
- Implements welcome challenge for new members

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open main admin menu for group moderators
- **/warn** (command, actor: admin, command: /warn @user [reason]) — Issue warning to user with optional reason
- **/mute** (command, actor: admin, command: /mute @user [duration]) — Temporarily mute user for specified duration
- **/kick** (command, actor: admin, command: /kick @user [reason]) — Remove user from group with optional reason
- **/ban** (command, actor: admin, command: /ban @user [reason]) — Permanently ban user with optional reason
- **/trust** (command, actor: admin, command: /trust @user) — Mark user as trusted to exempt from moderation rules
- **/log** (command, actor: admin, command: /log [period]) — View moderation log for specified time period
- **Я человек** (button, actor: user, callback: welcome:confirm) — Complete welcome challenge to begin chatting

## Flows

### New user onboarding
_Trigger:_ User joins group

1. Send welcome message with challenge button
2. Enforce 3-minute timer for challenge completion
3. Auto-remove user if challenge not completed
4. Mark user as verified upon challenge completion

_Data touched:_ user, welcome_challenge

### Spam detection
_Trigger:_ Message posted

1. Check message against spam rules (links from new accounts, repeated messages, rapid posting)
2. Apply configured actions (delete, warn, mute, kick) based on thresholds
3. Log automated action with explanation

_Data touched:_ user, rule_set, admin_action_record

### Admin moderation
_Trigger:_ /moderation command

1. Validate admin privileges
2. Execute requested action (warn, mute, kick, ban)
3. Update user status and log action

_Data touched:_ user, admin_action_record

### Audit reporting
_Trigger:_ /log command

1. Filter log entries by time period
2. Display summary metrics (joins, checks, deletions)
3. Show detailed log entries with evidence

_Data touched:_ admin_action_record

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **Group (chat)** _(retention: persistent)_ — Telegram group where bot is active
  - fields: chat_id, welcome_message, rules, trusted_users, log_channel
- **User** _(retention: persistent)_ — Telegram user profile with moderation status
  - fields: user_id, status, join_time, last_action
- **Welcome Challenge** _(retention: session)_ — Verification challenge for new users
  - fields: user_id, challenge_time, completed
- **Rule Set** _(retention: persistent)_ — Configurable spam detection rules and thresholds
  - fields: rule_type, threshold, action
- **Admin Action Record** _(retention: persistent)_ — Log of moderation actions
  - fields: timestamp, actor, target_user, reason, action, evidence

## Integrations

- **Telegram** (required) — Bot API messaging and moderation actions
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Configure welcome message content
- Set spam detection thresholds
- Define automated action rules
- Specify admin notification channel
- View and export moderation logs

## Notifications

- Automated action explanations in group chat
- Admin log entries in configured channel
- Welcome challenge timeout notifications

## Permissions & privacy

- User IDs and moderation status stored per group
- Message content analyzed only for spam detection
- Log entries contain only necessary moderation data

## Edge cases

- Users who never complete welcome challenge
- Admins who need to override automated actions
- Message patterns that match multiple spam rules

## Required tests

- Verify welcome challenge timeout behavior
- Test spam detection with various message patterns
- Validate admin command permissions
- Confirm log entry formatting and retention

## Assumptions

- Default 3-minute welcome challenge timeout balances user experience and security
- Automated spam rules focus on common bot behaviors (links from new accounts, message repetition)
- Admins will configure rules based on group needs
