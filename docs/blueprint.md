# TripSplit — Bot specification

**Archetype:** finance

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

Telegram bot for splitting trip expenses in group chats. Organizers create trips with currency and participants, users log expenses with custom splits, track live balances, simplify debt settlements, and confirm payments via private messages. Trip data remains private to participants with immutable expense records.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Friends
- Travel groups
- Roommates
- Small teams

## Success criteria

- Trip creation with currency and participants
- Expense logging with payer/participant splits
- Live balance tracking with debt simplification
- Private payment confirmation workflow
- Immutable audit trail of all transactions

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open trip management menu
- **/newtrip** (command, actor: user, command: /newtrip <name>) — Create new trip with optional name parameter
- **Join trip** (button, actor: user, callback: trip:join) — Request to join existing trip
- **Leave trip** (button, actor: user, callback: trip:leave) — Leave current trip
- **/balance** (command, actor: user, command: /balance) — Show current debt balances
- **Log expense** (button, actor: user, callback: expense:create) — Start expense logging flow

## Flows

### Trip creation
_Trigger:_ /newtrip

1. Request currency selection
2. Confirm participant list
3. Generate trip summary

_Data touched:_ Trip

### Expense logging
_Trigger:_ expense:create

1. Capture payer and amount
2. Select splitting method (equal/custom)
3. Confirm participants and amounts
4. Generate confirmation

_Data touched:_ Expense, Balance ledger

### Debt settlement
_Trigger:_ balance:check

1. Calculate minimal payment plan
2. Present settlement options
3. Process /paid confirmation

_Data touched:_ Repayment record, Balance ledger

### Participant management
_Trigger:_ trip:join/leave

1. Verify trip access
2. Update participant list
3. Adjust future splits

_Data touched:_ Participant, Trip

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **Trip** _(retention: persistent)_ — Group trip with currency and participants
  - fields: title, currency, organizer, participants, creation_time
- **Participant** _(retention: persistent)_ — User in trip with share weight
  - fields: telegram_id, display_name, join_time, share_weight
- **Expense** _(retention: persistent)_ — Logged expense with splitting details
  - fields: id, payer, amount, description, split_method, participants
- **Balance ledger** _(retention: persistent)_ — Net balances and transaction history
  - fields: participant_id, net_balance, transaction_history
- **Repayment record** _(retention: persistent)_ — Confirmed payments between users
  - fields: payer, payee, amount, timestamp, status

## Integrations

- **Telegram** (required) — Group chat and DM messaging
- **Telegram** (required) — Inline keyboard buttons for confirmations
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Archive/delete trips
- Add/remove participants
- View full expense ledger
- Set currency defaults

## Notifications

- Private DM for payment confirmations
- Group notification for trip creation
- Optional organizer alerts for joins/leaves

## Permissions & privacy

- Trip data visible only to participants
- Expense amounts hidden from non-participants
- Private confirmation DMs

## Edge cases

- Rounding adjustments for currency units
- Mid-trip joins/leaves handling
- Immutable expense records with correction entries
- Concurrency control for confirmations

## Required tests

- End-to-end trip creation flow
- Expense logging with custom splits
- Debt simplification accuracy
- Private confirmation workflow
- Data privacy in group chats

## Assumptions

- Default currency from group locale
- Equal split as default splitting method
- Deterministic rounding adjustments
- Greedy debt simplification algorithm
