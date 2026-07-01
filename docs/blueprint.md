# SafeShare Content Bot — Bot specification

**Archetype:** community

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

Telegram bot for secure, moderated media sharing within a closed adult community. Trusted creators upload content marked as available, members request content via buttons/commands, and moderators enforce rules through private chat interactions with audit trails.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- adult community members
- content moderators

## Success criteria

- 95% of content requests delivered within 5 seconds
- zero policy-violating content reaching users
- moderation response time < 2 hours for critical reports

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open main menu with rules acceptance prompt
- **/catalog** (command, actor: user, command: /catalog) — Browse available content categories and items
- **Get Content** (button, actor: user, callback: content:request) — Request specific content item by ID
  - inputs: content_id
  - outputs: media_file
- **Report** (button, actor: user, callback: content:report) — Submit content violation report
  - inputs: content_id, reason
  - outputs: moderation_ticket
- **/upload** (command, actor: creator, command: /upload) — Submit new content for moderation
  - inputs: media_file, title, tags, visibility
  - outputs: content_id

## Flows

### Onboarding
_Trigger:_ /start

1. Display rules
2. Require explicit acceptance
3. Grant access to catalog

_Data touched:_ user

### Content Request
_Trigger:_ content:request

1. Verify access permissions
2. Deliver media via private message
3. Log request in audit trail

_Data touched:_ request, content_item

### Moderation Workflow
_Trigger:_ content:report

1. Create moderation ticket
2. Notify moderators
3. Enable content removal/blocking

_Data touched:_ content_item, moderation_ticket

### Content Upload
_Trigger:_ /upload

1. Validate creator status
2. Store content with metadata
3. Notify moderators for review

_Data touched:_ content_item

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — Community member with access permissions
  - fields: telegram_id, access_level, consent_given
- **Content Item** _(retention: persistent)_ — Moderated media with metadata
  - fields: id, media_file, title, tags, author, visibility, approval_status
- **Request** _(retention: persistent)_ — User content request record
  - fields: user_id, content_id, timestamp
- **Moderator** _(retention: persistent)_ — Trusted content reviewer
  - fields: telegram_id, permissions

## Integrations

- **Telegram** (required) — Bot API messaging
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Configure content delivery mode (private vs group)
- Set content retention policies
- Manage moderator list
- Adjust audit log retention period

## Notifications

- Moderator alerts for new content uploads
- Error reports to admin chat
- Content approval/denial notifications

## Permissions & privacy

- All content delivery via private messages by default
- Content deletion possible at any time
- User consent required for participation
- Audit logs retained 30+ days

## Edge cases

- Content request for non-existent item
- Moderator override of content approval
- User revokes consent after onboarding
- VIP content access by non-VIP user

## Required tests

- End-to-end content request flow with private delivery
- Moderation workflow with report resolution
- Access control validation for VIP content

## Assumptions

- Private message delivery preferred for privacy
- Manual moderator management for small groups
- Content revocation possible at any time
- No payment integration in initial version
