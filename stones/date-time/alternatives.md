# Date & Time Rosetta Stone — Alternatives

Which APIs appear as columns next to Temporal, and the rules each column follows.
The operations themselves are in [operations.md](operations.md).

## Columns

Temporal comes first: it is the destination. Moment and Day.js sit side by side
because Day.js copies Moment's API. Each column shows a different way of
working with dates.

| Column | Packages (pinned) | Why it is here |
|---|---|---|
| **Temporal** | native `Temporal`; polyfill `temporal-polyfill@1.0.6` | The target. Immutable, zone- and calendar-aware, nanosecond precision |
| **Date + Intl** | built-in | The "Vanilla" baseline that every runtime has. Shows why Temporal exists |
| **Moment** | `moment@2.31.0`, `moment-timezone@0.6.6` | The most legacy code to migrate. Mutable, in maintenance mode since 2020 |
| **Day.js** | `dayjs@1.11.23` + official plugins | Moment's API, but immutable and split into plugins. Common Moment replacement |
| **date-fns** | `date-fns@4.4.0`, `@date-fns/tz@1.5.0` | Most downloaded. Plain functions over native `Date`; time zones via `TZDate` |
| **Luxon** | `luxon@3.7.2` | Closest to Temporal in design: zones, `Duration`, `Interval`, `Intl`-based |

Weekly npm downloads for the week before 2026-11-02, as a rough sign of how much
code uses each one (downloads also count CI runs and indirect installs):

| Package | Weekly downloads |
|---|---:|
| `date-fns` | 119 M |
| `dayjs` | 107 M |
| `luxon` | 57 M |
| `moment` | 52 M |
| `@date-fns/tz` | 40 M |
| `moment-timezone` | 25 M |

## Column rules

Every entry follows these rules so columns stay comparable.

### All columns
- Show the shortest idiomatic code a careful developer would write, not a
  one-liner that hides a bug.
- When an API cannot do something, say so plainly. Don't write a fake equivalent.
- Show each import or plugin the snippet needs.
- Snippets use the pinned versions above.

### Temporal
- Uses the global `Temporal` and `Intl`, nothing else.
- Code must run the same natively and with the polyfill.
- `recipe` entries define a small named function next to the snippet.

### Date + Intl
- Built-ins only: `Date`, `Intl.*`, `Math`, string methods.
- Shows the real workaround even when it's ugly. That contrast is the point.

### Moment
- `moment` for everything; `moment-timezone` only in entries about named zones.
- Shows `.clone()` wherever mutation would change the caller's value.

### Day.js
- Core plus official plugins from `dayjs/plugin/*` only.
- Every entry lists its `dayjs.extend(…)` calls (`utc`, `timezone`,
  `customParseFormat`, `isoWeek`, `duration`, `relativeTime`, …).

### date-fns
- v4 named imports (`import { addMonths } from 'date-fns'`).
- Time zones through `@date-fns/tz` (`TZDate`, the `in: tz(…)` option), not
  the older `date-fns-tz`.
- Locales from `date-fns/locale`.

### Luxon
- `DateTime`, `Duration`, `Interval`, `Info`, `Settings` from `luxon`.

## Entry states

Each cell in an entry has one of these states:

| State | Meaning |
|---|---|
| built-in | One call in the core API |
| plugin | Needs an official plugin or companion package (Day.js plugin, `moment-timezone`, `@date-fns/tz`) |
| recipe | A few lines on top of the API |
| unsupported | Not possible, or only with a different library |

## Left out

| Candidate | Weekly downloads | Why not a column |
|---|---:|---|
| `date-fns-tz` | 15 M | Replaced by `@date-fns/tz` for date-fns v4; last release 2024-09. Mention it in date-fns migration notes |
| `@js-joda/core` | 7.4 M | Almost all transitive: the `tedious` SQL Server driver (7.1 M) depends on it. Same design family as Temporal; could be a later appendix |
| `@js-temporal/polyfill` | 6.3 M | A Temporal implementation, not an alternative. Last release 2025-03 |
| `rrule` | 4.3 M | Recurrence rules only. Referenced from `external` entries |
| `chrono-node` | 3.1 M | Natural-language parsing only. Referenced from `external` entries |
| `date-and-time`, `spacetime`, `timeago.js`, `@formkit/tempo` | < 0.6 M each | Too little use to justify a column |
