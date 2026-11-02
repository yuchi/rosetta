# Date & Time Rosetta Stone — Operation Catalog

The full list of date/time operations the first Rosetta stone compares. The goal is
to show developers (and agents) how each operation maps to the **Temporal API**.

Each level-3 item is one comparison entry: one task, shown in every column.

**Columns:** Temporal · `Date` + `Intl` · Moment · Day.js · date-fns · Luxon.
Choices, pinned versions, and per-column rules are in [alternatives.md](alternatives.md).

**Structure:** `## domain` › `### topic` › numbered operation. An operation's ID is
its position, e.g. `8.1.3` = Arithmetic › Calendar units › Add months.
Domains 1–21 list single operations. Domain 22 lists use cases: real tasks that
combine several operations. Each use case names the IDs of the operations it uses.

**Tags** (after an operation):

- `recipe`: Temporal has no single call for this. The entry shows a short helper.
- `Intl`: in the Temporal era the answer is in `Intl`, not `Temporal`.
- `external`: neither Temporal nor `Intl` covers it. Point to a library and show
  how it fits with Temporal values.

A short note after an operation names the trap or detail its entry must show.

---

## 1. Current date and time

### 1.1 Current instant
1. Current timestamp in milliseconds
2. Current timestamp in nanoseconds (BigInt)
3. Current timestamp in Unix seconds `recipe`. Temporal has no `epochSeconds`
4. Current instant as an object

### 1.2 Current wall-clock time
1. Now in the system time zone (date + time + zone)
2. Today's date, no time
3. Current time of day, no date
4. Current date-time without a zone
5. Now in a specific time zone ("what time is it in Tokyo")
6. Today in a specific time zone. "Today" depends on the zone
7. Current year-month / current year

### 1.3 Clock environment
1. System time zone ID
2. Measure elapsed time. Use the monotonic `performance.now()`, not the wall clock

---

## 2. Creating values

### 2.1 From components
1. Date from year, month, day. `Date` months are 0-based
2. Date-time from components
3. Time of day from components, down to nanoseconds
4. Year-month (card expiry, billing month)
5. Month-day (birthday, anniversary)
6. Zoned date-time from components plus a time zone
7. UTC date-time from components. `Date.UTC`
8. From a property bag / plain object
9. From ISO week fields (week-year, week, weekday) `recipe`
10. From an ordinal date (year + day of year) `recipe`
11. Years 0–99. `new Date(99, 0, 1)` gives 1999
12. Negative (BCE) and extended (6-digit) years

### 2.2 From epoch values
1. From epoch milliseconds
2. From epoch seconds (Unix timestamp) `recipe`
3. From epoch microseconds `recipe`
4. From epoch nanoseconds (BigInt)
5. From an epoch value straight to wall-clock time in a zone

### 2.3 Overflow and limits
1. Out-of-range day (Feb 30). `Date` silently rolls over; Temporal `overflow: 'constrain' | 'reject'`
2. Out-of-range fields (month 13, hour 25, minute 60)
3. Values outside the supported range (±10⁸ days from epoch)
4. Leap-second input (`23:59:60`)

### 2.4 Immutability and copying
1. Clone a value. Temporal values are immutable, so there is nothing to clone
2. Defensive copy before mutation. `Date` and Moment mutate in place

---

## 3. Parsing

### 3.1 ISO 8601 / RFC 3339 / RFC 9557
1. Date only (`2024-03-15`). `Date` reads it as UTC midnight, so western zones get the day before
2. Date-time without an offset (`2024-03-15T10:30`). `Date` reads it as local time
3. Date-time with `Z`
4. Date-time with a numeric offset (`+01:00`)
5. Date-time with a bracketed time zone (`…+01:00[Europe/Rome]`). Only Temporal
6. Offset that conflicts with the zone. `offset: 'use' | 'ignore' | 'prefer' | 'reject'`
7. Calendar annotation (`[u-ca=hebrew]`)
8. Time only (`10:30:00.123`)
9. Year-month (`2024-03`)
10. Month-day (`--03-15`, `03-15`)
11. Fractional seconds below milliseconds (µs, ns)
12. Space instead of `T` (SQL style `2024-03-15 10:30:00`)
13. Basic/compact format (`20240315T103000Z`)
14. Extended years (`+275760-09-13`, `-000001-01-01`)
15. ISO week date (`2024-W11-5`) `recipe`
16. ISO ordinal date (`2024-075`) `recipe`
17. Instant from a string with no offset. `Temporal.Instant.from` throws
18. Zone-less string read as wall-clock time in a given zone (Moment `tz(str, zone)`)
19. Keep the string's own offset (Moment `parseZone`). `ZonedDateTime.from` needs a bracketed zone

### 3.2 Other standard formats
1. RFC 2822 / RFC 5322 (email headers) `recipe`
2. HTTP date, RFC 9110 (`Tue, 15 Mar 2024 10:30:00 GMT`) `recipe`
3. Output of `Date.prototype.toString()` `recipe`
4. Numeric timestamp strings (`"1710498600"`)
5. ISO 8601 durations (see 15.1.2)

### 3.3 Custom and localized formats
1. Format pattern (`DD/MM/YYYY`) `recipe`. Temporal has no pattern parser
2. Localized month and weekday names (`15 mars 2024`) `recipe`
3. 12-hour clock with AM/PM `recipe`
4. Two-digit years with a pivot year `recipe`
5. Time zone abbreviations (`EST`, `CET`). Ambiguous; an IANA ID is required
6. Strict vs forgiving parsing
7. Try several formats in order `recipe`
8. Natural language ("next Friday", "in 3 days") `external`

### 3.4 Parse errors
1. Detect a failed parse. `Invalid Date`/`NaN` vs a thrown `RangeError`
2. Safe parse that returns `null` `recipe`
3. Report which field is invalid (Moment `invalidAt()`)

---

## 4. Validation and type checks

### 4.1 Value validity
1. Is this a valid date? A Temporal value cannot be invalid
2. Does this year-month-day exist (Feb 29, 2023)? `overflow: 'reject'`
3. Does this wall-clock time exist in this zone (DST gap)? `disambiguation: 'reject'`
4. Is this string valid ISO 8601?
5. Is this a valid IANA time zone ID?
6. Is this a valid calendar ID?

### 4.2 Type checks
1. Is it a `Date`? `instanceof` fails across realms
2. Is it a Temporal value, and which type?
3. Is it a library object (`moment.isMoment`, `DateTime.isDateTime`, `dayjs.isDayjs`)?
4. Is it a duration?

---

## 5. Reading fields

### 5.1 Date fields
1. Year
2. Month number. `Date` is 0-based, Temporal is 1-based
3. Month code (`M01`, leap month `M05L`)
4. Day of month. `getDate()` vs `getDay()`
5. Day of week. `Date` 0 = Sunday; Temporal 1 = Monday … 7 = Sunday
6. Day of week relative to the locale's first day `recipe`
7. Day of year
8. ISO week number
9. ISO week-numbering year. Differs from the calendar year around January 1
10. Locale week number (e.g. US) `recipe`
11. Week of month `recipe`
12. Quarter `recipe`
13. Era and era year
14. Decade / century `recipe`

### 5.2 Time fields
1. Hour
2. Minute
3. Second
4. Millisecond
5. Microsecond and nanosecond
6. UTC versions of each field (`getUTCHours` …)

### 5.3 Zone and offset fields
1. Time zone ID
2. Offset as a string (`+05:30`)
3. Offset as a number. `getTimezoneOffset()` has the opposite sign
4. Time zone name or abbreviation (`CET`, `Central European Time`) `Intl`

### 5.4 Epoch fields
1. Epoch milliseconds. `getTime()`/`+date` vs `epochMilliseconds`
2. Epoch seconds `recipe`
3. Epoch nanoseconds

### 5.5 Calendar metadata
1. Days in month
2. Days in year
3. Is leap year
4. Months in year (13 in a Hebrew leap year)
5. Days in week
6. Hours in day (23 or 25 on DST days)
7. ISO weeks in year (52 or 53) `recipe`
8. Weeks in month, for calendar grids `recipe`
9. Calendar ID

### 5.6 Bulk and generic access
1. All fields as an object. `{...plainDate}` is `{}`, because fields are getters `recipe`
2. All fields as an array (Moment `toArray`) `recipe`
3. Get a field by unit name (`get('month')`) `recipe`

---

## 6. Changing fields

### 6.1 Date fields
1. Set year. Feb 29 → a non-leap year
2. Set month. Jan 31 → February
3. Set day of month
4. Set day of week in the same week (Monday of this week) `recipe`
5. Set day of year `recipe`
6. Set ISO week / week-year `recipe`
7. Set quarter `recipe`
8. Set several fields at once (`with({ … })`)

### 6.2 Time fields
1. Set hour, minute, second, millisecond
2. Set microsecond and nanosecond
3. Replace the whole time of day (`withPlainTime`)
4. Clear the time to midnight. `setHours(0,0,0,0)` vs `toPlainDate()` / `startOfDay()`
5. Replace the date and keep the time

### 6.3 Mutation semantics
1. Setters mutate vs return a new value. `Date` and Moment mutate
2. Chain several changes
3. Set a field by unit name (`set('month', 3)`) `recipe`

---

## 7. Converting between types

### 7.1 Between Temporal types
1. Instant → ZonedDateTime (`toZonedDateTimeISO`)
2. ZonedDateTime → Instant
3. ZonedDateTime → PlainDateTime / PlainDate / PlainTime
4. PlainDateTime → ZonedDateTime, choosing `disambiguation`
5. PlainDate → ZonedDateTime, at start of day or at a given time
6. PlainDate + PlainTime → PlainDateTime
7. PlainDateTime → PlainDate / PlainTime
8. PlainDate → PlainYearMonth / PlainMonthDay
9. PlainYearMonth + day → PlainDate
10. PlainMonthDay + year → PlainDate. Feb 29 in a non-leap year
11. PlainDateTime → Instant. Impossible without a zone, by design

### 7.2 Legacy `Date` ↔ Temporal
1. `Date` → Instant (`toTemporalInstant()`)
2. `Date` → ZonedDateTime, in the system zone or a given zone
3. `Date` → PlainDate. Local date or UTC date?
4. Instant / ZonedDateTime → `Date`. Precision drops to milliseconds
5. PlainDate / PlainDateTime → `Date`. A time and a zone must be chosen
6. Date-only values stored as UTC midnight → PlainDate. The classic off-by-one-day bug

### 7.3 Library objects ↔ Temporal
1. Moment → Temporal, keeping the zone
2. Luxon `DateTime` → Temporal
3. Day.js → Temporal
4. date-fns `Date` / `TZDate` → Temporal
5. Temporal → each library, for gradual migration
6. Library durations ↔ `Temporal.Duration`

---

## 8. Arithmetic

### 8.1 Calendar units
1. Add/subtract days
2. Add/subtract weeks
3. Add/subtract months. Jan 31 + 1 month
4. Add/subtract quarters `recipe`
5. Add/subtract years. Feb 29 + 1 year
6. Control overflow (`constrain` vs `reject`)
7. Order matters: months then days vs days then months
8. Subtract vs add a negative value. Month-end results differ

### 8.2 Exact time units
1. Add/subtract hours, minutes, seconds
2. Add/subtract milliseconds, microseconds, nanoseconds
3. Add to an Instant. Days and larger units throw
4. Add to a time of day. Wraps around midnight

### 8.3 Across DST
1. Add 1 day vs add 24 hours across a DST change
2. Add hours that land in a gap or overlap
3. Add days that land on a nonexistent wall-clock time

### 8.4 Durations as operands
1. Add a duration object
2. Add mixed units (`{ months: 1, hours: 2 }`). Order in which they apply
3. Add a negative duration

### 8.5 Business days
1. Add N business days `recipe`
2. Subtract N business days `recipe`
3. Business days between two dates `recipe`
4. Next / previous business day `recipe`
5. Skip holidays from a list `recipe`
6. Locale weekends (Friday–Saturday) `Intl`. `Intl.Locale` week info

---

## 9. Differences

### 9.1 Single-unit differences
1. In milliseconds
2. In seconds, minutes, hours
3. In days. Calendar days vs 24-hour periods
4. In weeks
5. In months
6. In years
7. In quarters `recipe`
8. Fractional (`2.5 months`). `total({ unit, relativeTo })`

### 9.2 Calendar vs elapsed differences
1. Calendar-day difference that ignores the time (date-fns `differenceInCalendarDays`)
2. Full elapsed days, truncated
3. Whole months/years: truncated vs rounded
4. Difference across a DST change
5. Difference between values in different zones

### 9.3 Multi-unit differences
1. Balanced difference (years, months, days, hours …) with `until` / `since`
2. Cap the largest unit (`largestUnit: 'hours'` → 50 h instead of 2 d 2 h)
3. Smallest unit and rounding (`smallestUnit`, `roundingIncrement`, `roundingMode`)
4. Signed vs absolute difference. `until` vs `since`, `abs()`
5. Between two times of day across midnight (night shift) `recipe`

---

## 10. Rounding, boundaries, and navigation

### 10.1 Rounding
1. Round to the nearest minute, hour, day
2. Round to an increment (nearest 15 minutes)
3. Floor / ceil / truncate
4. Tie-breaking modes (`halfExpand`, `halfEven`, …)
5. Round a zoned value to a day. Uses the real day length (23/25 h)
6. Round a date. PlainDate has no `round()`

### 10.2 Start of a unit
1. Start of minute / hour
2. Start of day. `startOfDay()`: midnight may not exist
3. Start of week. ISO Monday vs locale first day
4. Start of month
5. Start of quarter `recipe`
6. Start of year
7. Start of decade / century `recipe`
8. Start of ISO week-year `recipe`

### 10.3 End of a unit
1. End of day. `23:59:59.999` vs the exclusive start of the next day
2. End of week
3. End of month / last day of month
4. End of quarter `recipe`
5. End of year
6. Precision of "end" (Moment `.999` ms vs Temporal nanoseconds)

### 10.4 Relative navigation
1. Tomorrow / yesterday
2. Next / previous weekday (next Monday) `recipe`
3. Next / previous month-day (next birthday) `recipe`
4. Nth weekday of a month (third Thursday) `recipe`
5. Last weekday of a month (last Friday) `recipe`
6. First / last day of a month
7. First / last day of a year
8. Same day next month / last month
9. Next time a time of day occurs (next 09:00) `recipe`
10. Closest date to a target, from a list `recipe`

---

## 11. Comparing and querying

### 11.1 Ordering
1. Is before / is after. `<` and `>` throw on Temporal values (`valueOf` throws)
2. Is same or before / same or after
3. Comparator for sorting (`Temporal.X.compare`)
4. Sort ascending / descending
5. Min / max of a list `recipe`
6. Compare different types (PlainDate vs ZonedDateTime). Convert first
7. Compare across zones: by instant or by wall-clock time
8. Sort month-days (birthdays in a year) `recipe`. PlainMonthDay has no `compare`

### 11.2 Equality
1. Exact equality. `equals()` checks zone and calendar; `===` on `Date` checks identity
2. Same instant in different zones or calendars
3. Same wall-clock time in different zones
4. Same unit (same day/week/month/year) `recipe`
5. Time zone aliases (`Asia/Calcutta` vs `Asia/Kolkata`)
6. Use as a `Map`/`Set` key or deduplicate a list `recipe`. Objects match by identity; key by `toString()`

### 11.3 Relative to now
1. Is today / tomorrow / yesterday
2. Is in the past / future
3. Is this week / month / year
4. Is within the last N days

### 11.4 Calendar predicates
1. Is a weekend / weekday, by locale
2. Is a given weekday (`isMonday` …)
3. Is in a leap year
4. Is the first / last day of the month

---

## 12. Ranges and intervals

### 12.1 Representing intervals
1. Interval as a start and an exclusive end `recipe`. Temporal has no Interval type
2. Interval from a start plus a duration `recipe`
3. Interval length as a duration
4. Empty or inverted interval (end before start)

### 12.2 Interval queries
1. Is a value inside, with inclusive/exclusive bounds (`isBetween`)
2. Do two intervals overlap?
3. Length of the overlap (overlapping days)
4. Does one interval contain another?
5. Do two intervals touch (abut)?
6. Clamp a value to an interval `recipe`

### 12.3 Interval set operations
1. Union / merge overlapping intervals `recipe`
2. Intersection `recipe`
3. Difference (busy slots → free slots) `recipe`
4. Sort intervals

### 12.4 Iterating and splitting
1. Every day between two dates `recipe`
2. Every week / month / quarter / year in a range `recipe`
3. Every hour / minute / N-minute slot `recipe`
4. Every weekend in a range `recipe`
5. Split an interval into chunks of a duration `recipe`
6. Divide an interval into N equal parts `recipe`
7. Count occurrences in an interval (how many Mondays) `recipe`

---

## 13. Time zones and offsets

### 13.1 Converting between zones
1. Instant → wall-clock time in a zone
2. Wall-clock time in a zone → instant
3. Move to another zone, keeping the instant (`withTimeZone`)
4. Move to another zone, keeping the wall-clock time `recipe`
5. To UTC / from UTC
6. System zone ↔ UTC
7. Fixed-offset zones (`+05:30`). `Etc/GMT-5` means UTC+5
8. One moment in several zones (world clock)

### 13.2 Time zone information
1. List supported zones (`Intl.supportedValuesOf('timeZone')`) `Intl`
2. Validate a zone ID
3. Normalize case and compare aliases. Temporal fixes the case but keeps aliases as given
4. Display name (long, short, generic) `Intl`
5. A zone's current offset
6. A zone's offset at a given instant
7. All offset changes in a year `recipe`
8. Map an abbreviation (`EST`) to an IANA zone. Not reliably possible

### 13.3 Floating (zone-less) values
1. When to drop the zone (birthdays, all-day events, opening hours)
2. Attach a zone to a floating value
3. Drop the zone from a zoned value

---

## 14. DST and ambiguous times

### 14.1 Gaps (spring forward)
1. Resolve a nonexistent time (02:30 on spring-forward day). `disambiguation`
2. Reject nonexistent times
3. Detect that a time was moved

### 14.2 Overlaps (fall back)
1. Choose the earlier / later of a repeated time
2. Reject ambiguous times
3. Get both possible instants `recipe`
4. Keep the chosen instant through a string round trip (offset + zone in the string)

### 14.3 Transitions
1. Next DST transition (`getTimeZoneTransition('next')`)
2. Previous DST transition
3. Is DST in effect? `recipe`
4. Repeated or missing hours when iterating hourly

### 14.4 Rule changes and history
1. Future events after a zone changes its rules. Store zone + wall-clock time; `offset: 'prefer'`
2. Historical dates with sub-minute offsets (local mean time)
3. Different tzdata versions across runtimes

---

## 15. Durations

### 15.1 Creating
1. From components
2. From an ISO 8601 string (`P1Y2M3DT4H`)
3. From milliseconds or a count of one unit
4. Negative durations. All fields share one sign
5. Mixed signs. Rejected by Temporal
6. From the difference of two values (see 9.3)
7. From a clock string (`01:30:00`) `recipe`

### 15.2 Reading
1. Get one field (the hours part)
2. Total in one unit (`as('hours')` vs `total('hours')`)
3. Total in calendar units. Needs `relativeTo`
4. Sign / is zero / is negative (`sign`, `blank`)
5. Convert units (90 minutes → 1.5 hours; days → weeks)

### 15.3 Arithmetic
1. Add / subtract durations. Years, months, weeks throw
2. Negate / absolute value
3. Multiply / divide by a number `recipe`
4. Sum a list (timesheet) `recipe`
5. Change fields (`with`)

### 15.4 Balancing and rounding
1. Balance (90 min → 1 h 30 min) with `round({ largestUnit })`
2. Flatten to one unit (1 h 30 min → 90 min)
3. Balance calendar units with `relativeTo`. Days → months depends on the month
4. Round to a smallest unit / increment
5. Normalize mixed calendar and time units (Luxon `normalize`, `shiftTo`, `rescale`)

### 15.5 Comparing
1. Compare (`Duration.compare`; `relativeTo` for calendar units)
2. Equality (`PT60M` vs `PT1H`). Duration has no `equals()`
3. Sort durations
4. Min / max `recipe`

### 15.6 Formatting
1. ISO 8601 string
2. Localized ("1 hour, 30 minutes"), long/short/narrow `Intl`. `Intl.DurationFormat`
3. Digital clock style (`1:30:00`) `Intl`
4. Approximate wording ("a few seconds", "2 months"), like Moment `humanize` `recipe`
5. Custom pattern (`hh:mm`) `recipe`

---

## 16. Formatting: machine-readable

### 16.1 ISO 8601 / RFC 9557 output
1. ISO string in UTC (`toISOString`)
2. ISO string with the local offset
3. ISO string with offset and bracketed zone
4. Date-only ISO string. `toISOString().slice(0, 10)` gives the UTC date
5. Time-only ISO string
6. Number of fractional digits / smallest unit
7. Rounding mode when shortening
8. Show or hide offset, zone, calendar (`offset`, `timeZoneName`, `calendarName`)
9. Instant as a string in a given zone (`instant.toString({ timeZone })`)

### 16.2 Other standard formats
1. RFC 2822 (email) `recipe`
2. HTTP date (`toUTCString()`) `recipe`
3. SQL date/time literals `recipe`
4. Unix timestamp (seconds / milliseconds)
5. JSON (`toJSON`)
6. Compact sortable stamp for file names (`20240315T103000Z`) `recipe`

---

## 17. Formatting: human-readable

### 17.1 Localized
1. Default locale format (`toLocaleString()`)
2. Date only / time only
3. `dateStyle` / `timeStyle` presets (short … full)
4. Field options (weekday long, month short, numeric …)
5. A specific locale
6. A specific zone. ZonedDateTime formats in its own zone; Plain types never shift
7. Zone name in the output (`CET`, `GMT+1`, long)
8. 12/24-hour clock (`hourCycle`, `hour12`)
9. Other calendars and numbering systems
10. Day periods ("in the morning", `dayPeriod`)
11. Reuse one `Intl.DateTimeFormat` for many values. Its `format()` rejects ZonedDateTime
12. Format to parts, for custom markup
13. Date ranges ("Mar 15 – 17, 2024", `formatRange`)
14. Ordinal day ("March 15th") `recipe`. `Intl.PluralRules` with `type: 'ordinal'`
15. Name of one date's weekday or month ("Friday", "March")
16. Lists of month and weekday names `recipe`
17. Localized presets (Moment `L`/`LL`, Luxon `DATE_FULL`) → `dateStyle`/`timeStyle`
18. Default locale: global setting vs per call

### 17.2 Relative time
1. "3 days ago" / "in 2 hours" `Intl`. `Intl.RelativeTimeFormat`
2. Pick the best unit automatically ("time ago") `recipe`
3. "Yesterday" / "tomorrow" (`numeric: 'auto'`) `Intl`
4. Calendar style ("Yesterday at 3:00 PM", "Last Monday"), like Moment `calendar()` `recipe`
5. Short / narrow style ("3d ago") `Intl`

### 17.3 Custom patterns
1. Token patterns (`YYYY-MM-DD HH:mm`) `recipe`. Neither Temporal nor `Intl` has one
2. Zero-padding fields
3. Literal text and escaping
4. Moment / Luxon / date-fns / Day.js tokens → `Intl` options (migration table)
5. Fixed, locale-independent format for logs

---

## 18. Calendars and locale rules

### 18.1 Non-ISO calendars
1. Create a date in a calendar (Hebrew, Islamic, Japanese, Chinese, Persian, Buddhist …)
2. Convert between calendars (`withCalendar`)
3. Arithmetic in a non-ISO calendar (add months in the Hebrew calendar)
4. Leap months (`monthCode: 'M05L'`)
5. Eras (Japanese imperial eras, BCE/CE)
6. List supported calendars (`Intl.supportedValuesOf('calendar')`) `Intl`
7. Compute in ISO, display in another calendar
8. Compare across calendars. `compare()` ignores the calendar, `equals()` does not

### 18.2 Weeks
1. A locale's first day of the week `Intl`. `Intl.Locale` week info
2. A locale's weekend days `Intl`
3. Minimal days in the first week (ISO vs US numbering)
4. ISO week vs locale week numbering
5. Days in the week for a calendar

### 18.3 Locale data
1. Custom or overridden locale data (Moment `updateLocale`). `Intl` has no override `recipe`
2. Locale negotiation and fallback `Intl`

---

## 19. Serialization, storage, and interop

### 19.1 JSON
1. Serialize to JSON
2. Revive from JSON with a reviver `recipe`. The type must be known
3. Wire format for each type (instant, zoned, plain)

### 19.2 Storage
1. Store an instant (database timestamp)
2. Store a future local event: wall-clock time + zone, not UTC
3. Store a date without a time
4. Store a time zone in its own column
5. Store durations
6. Sortable string keys. ISO strings sort correctly only with the same offset and year width

### 19.3 HTML and HTTP
1. Read `<input type="date">` into a plain date. `valueAsDate` is UTC midnight
2. Write a plain date into `<input type="date">`
3. `datetime-local` ↔ PlainDateTime (no zone, optional seconds)
4. `time` ↔ PlainTime
5. `month` ↔ PlainYearMonth
6. `week` ↔ ISO week `recipe`
7. `<time datetime>` attribute
8. HTTP headers (`Date`, `Last-Modified`, cookie `Expires`) `recipe`

### 19.4 Crossing boundaries
1. `postMessage` / `structuredClone` / Workers. Send strings
2. URL query parameters
3. Form data
4. APIs that require a `Date` (ORMs, older libraries)
5. Spreadsheet serial dates (Excel) `recipe`

---

## 20. Testing and runtime

### 20.1 Availability
1. Feature detection (`typeof Temporal`)
2. Load a polyfill only when needed
3. TypeScript typings

### 20.2 Controlling time in tests
1. Mock the current time
2. Inject a clock (dependency injection) `recipe`
3. Freeze and advance time
4. Force the system time zone (`TZ`)
5. Force the locale
6. Stable snapshots of formatted output

### 20.3 Limits and precision
1. Supported range (±10⁸ days for Instant, same as `Date`)
2. Precision: milliseconds vs nanoseconds
3. Leap seconds. Not modeled

---

## 21. Modeling

### 21.1 Choosing a type
1. Log entry / `created_at` → Instant
2. Meeting at a place → ZonedDateTime
3. Birthday → PlainMonthDay (or PlainDate)
4. All-day event / holiday → PlainDate
5. Opening hours → PlainTime (+ weekday + zone)
6. Card expiry / billing month → PlainYearMonth
7. Daily alarm at 07:00 → PlainTime + zone
8. Timer length → Duration
9. Flight departure and arrival in two zones → two ZonedDateTimes

---

## 22. Use cases

### 22.1 Calendar widgets and date pickers
1. All dates of a month. Loop from day 1 to `daysInMonth`\
   Uses: 1.2.7, 10.2.4, 5.5.1, 12.4.1
2. Month grid (6 × 7) padded with days from the previous and next month. The locale's first day of week\
   Uses: 10.2.4, 10.3.3, 10.2.3, 18.2.1, 5.1.6, 12.4.1
3. Week-number column. ISO vs locale numbering\
   Uses: 5.1.8, 5.1.9, 5.1.10, 18.2.4
4. Highlight today. "Today" in the user's zone, not UTC\
   Uses: 1.2.2, 1.2.6, 11.3.1, 11.2.1
5. Previous / next month buttons. Step a year-month, not a Jan 31 date\
   Uses: 7.1.8, 8.1.3, 7.1.9
6. Disabled dates (before min, after max, weekends, holidays)\
   Uses: 11.1.1, 12.2.1, 11.4.1, 8.5.5
7. Range selection with hover preview and night count\
   Uses: 11.1.1, 12.2.1, 9.1.3, 17.1.13
8. Year view (12 mini-months) and decade picker\
   Uses: 12.4.2, 10.2.6, 10.2.7, 5.1.14
9. Week view with hourly rows. 23- and 25-hour days\
   Uses: 10.2.3, 12.4.3, 5.5.6, 14.3.4
10. Day view with 30-minute slots\
    Uses: 10.2.2, 12.4.3, 10.1.2, 17.1.2
11. Header labels: weekday names in locale order, a "March 2024" title\
    Uses: 17.1.16, 18.2.1, 17.1.4, 17.1.15
12. Agenda view grouped by day with "Today" / "Tomorrow" headings\
    Uses: 7.1.3, 11.2.6, 17.2.3, 17.2.4

### 22.2 Scheduling and booking
1. Bookable slots for a day from opening hours, slot length and buffer, in the business's zone\
   Uses: 21.1.5, 7.1.5, 12.4.5, 8.2.1
2. Remove booked slots. Interval difference\
   Uses: 12.2.2, 12.3.1, 12.3.3
3. Show availability in the viewer's zone\
   Uses: 1.3.1, 13.1.3, 17.1.6
4. Find a meeting time across zones. Overlapping working hours\
   Uses: 13.1.2, 12.3.2, 12.4.3, 13.1.8
5. Weekly event that stays at 09:00 local through DST\
   Uses: 19.2.2, 8.1.2, 8.3.1, 14.1.1
6. Monthly event "on the 31st". Add N months to the first date, not 1 month to the previous one\
   Uses: 8.1.3, 8.1.6, 2.3.1
7. "Second Tuesday of every month"\
   Uses: 10.4.4, 12.4.2, 5.1.5
8. Expand recurring events inside the visible range `external`. RRULE\
   Uses: 12.2.1, 12.4.2, 19.2.2
9. Hotel stay: nights, check-in 15:00 / check-out 11:00 in the hotel's zone\
   Uses: 21.1.4, 9.1.3, 7.1.5, 17.1.13
10. Flight: departure and arrival zones, duration, "+1 day" marker\
    Uses: 21.1.9, 9.3.1, 9.2.5, 9.2.1
11. Reminders: "15 minutes before", "the day before at 18:00"\
    Uses: 8.2.1, 8.1.1, 6.2.3, 8.3.3

### 22.3 Timestamps in feeds and messages
1. "3 minutes ago", switching to an absolute date after a week\
   Uses: 17.2.1, 17.2.2, 9.3.1, 17.1.3
2. Refreshing relative labels. Work out when the label next changes\
   Uses: 17.2.2, 9.1.2, 10.1.3
3. Day separators in a chat ("Today", "Yesterday", "Monday", "March 3")\
   Uses: 11.2.4, 11.3.1, 17.2.4, 17.1.15
4. Server UTC timestamp shown in the user's zone and locale\
   Uses: 3.1.3, 7.1.1, 1.3.1, 17.1.3
5. Full date-time tooltip on a relative label\
   Uses: 17.1.3, 17.1.7, 19.3.7
6. "Last seen" and "edited" markers\
   Uses: 11.1.1, 9.1.2, 17.2.1

### 22.4 Business and billing
1. Monthly renewal starting Jan 31\
   Uses: 8.1.3, 8.1.6, 9.1.5
2. Trial ending after N days, at end of day in the user's zone\
   Uses: 8.1.1, 10.3.1, 10.2.2, 13.1.2
3. Invoice due dates: net 30, end of month + 30, N business days\
   Uses: 8.1.1, 10.3.3, 8.5.1, 8.5.5
4. SLA deadline that counts only business hours and skips holidays\
   Uses: 8.5.4, 8.5.5, 12.3.2, 15.3.4
5. Proration: share of the billing period used\
   Uses: 9.1.3, 9.1.8, 12.1.3
6. Fiscal years and quarters starting in April or July\
   Uses: 5.1.12, 10.2.5, 8.1.3, 6.1.1
7. Payroll dates: every other Friday, last business day of the month\
   Uses: 8.1.2, 10.4.5, 10.3.3, 8.5.4
8. Card valid through the end of its expiry month\
   Uses: 21.1.6, 3.3.1, 10.3.3, 11.1.1
9. Timesheet totals, overnight shifts, overtime\
   Uses: 9.3.5, 15.3.4, 15.4.1, 15.5.1
10. Interest day-count conventions (actual/365, 30/360) `recipe`\
    Uses: 9.1.3, 5.5.2, 5.1.4

### 22.5 Reporting and analytics
1. Group records by day / week / month in the user's zone\
   Uses: 7.1.1, 7.1.3, 10.2.3, 7.1.8, 11.2.6
2. Fill empty days in a time series\
   Uses: 12.4.1, 11.2.6
3. Fixed-size buckets (5 minutes, UTC)\
   Uses: 10.1.2, 10.1.3, 5.4.1
4. Filter presets: "Last 7 days", "This month", "Last quarter", year to date\
   Uses: 1.2.6, 8.1.1, 10.2.4, 10.2.5, 10.2.6, 12.1.1
5. Same period last year / previous-period comparisons\
   Uses: 8.1.5, 9.1.3, 12.1.2
6. Rolling windows (trailing 7 / 30 days)\
   Uses: 8.1.1, 12.2.1, 11.3.4
7. Cohorts by signup week\
   Uses: 10.2.3, 5.1.8, 5.1.9, 9.1.4
8. Which zone defines "a day" for daily metrics\
   Uses: 1.2.6, 10.2.2, 5.5.6, 13.1.1
9. Chart axis ticks at round dates (month starts, Mondays)\
   Uses: 12.4.2, 10.2.4, 10.2.3, 17.1.4
10. Average and percentile durations (response times)\
    Uses: 9.1.1, 15.3.4, 15.3.3, 15.5.3, 15.6.2

### 22.6 Forms and input validation
1. Date of birth: at least 18 years old\
   Uses: 19.3.1, 9.1.6, 8.1.5, 11.1.1
2. Date range: end after start, maximum span\
   Uses: 11.1.1, 12.1.4, 9.1.3
3. Not in the past / at least 24 hours ahead\
   Uses: 1.1.4, 11.3.2, 8.2.1
4. Card expiry `MM/YY` → year-month\
   Uses: 3.3.1, 3.3.4, 2.1.4
5. Time field with 12- or 24-hour display by locale\
   Uses: 19.3.4, 17.1.8, 3.3.3
6. Normalize dates typed in several formats\
   Uses: 3.3.7, 3.3.6, 3.4.2, 4.1.2
7. Time zone picker: zones with their current offset, sorted and grouped\
   Uses: 13.2.1, 13.2.5, 13.2.4, 5.3.2

### 22.7 Countdowns, timers, and clocks
1. Live countdown to a date ("3d 4h 12m 5s")\
   Uses: 9.3.1, 9.3.2, 15.4.4, 15.6.2
2. Stopwatch / elapsed time. Monotonic clock\
   Uses: 1.3.2, 15.1.3, 15.6.3
3. Session timeout warning\
   Uses: 1.1.4, 8.2.1, 9.1.2
4. World clock for several cities\
   Uses: 13.1.8, 13.2.4, 17.1.7
5. `setTimeout` until an instant. Delays over ~24.8 days overflow and fire immediately\
   Uses: 1.1.4, 9.1.1, 15.2.2
6. Token expiry check (JWT `exp` in Unix seconds)\
   Uses: 2.2.2, 1.1.3, 11.3.2
7. "Try again in 5 minutes" from a rate-limit reset header\
   Uses: 3.2.2, 3.2.4, 9.1.2, 17.2.1

### 22.8 Personal and everyday dates
1. Age in completed years\
   Uses: 1.2.2, 9.1.6, 9.2.3
2. Next birthday and days until it, including Feb 29\
   Uses: 21.1.3, 7.1.10, 10.4.3, 9.1.3
3. Time together ("2 years, 3 months, 5 days")\
   Uses: 9.3.1, 15.6.2
4. Holidays: fixed (Dec 25), floating (4th Thursday of November), computed (Easter) `recipe`\
   Uses: 2.1.5, 7.1.10, 10.4.4, 2.1.1
5. Is a store open now, including hours past midnight\
   Uses: 21.1.5, 1.2.5, 5.1.5, 11.1.1, 8.2.4
6. Greeting by time of day\
   Uses: 1.2.3, 5.2.1, 17.1.10
7. Sprint or course week number from a start date\
   Uses: 9.1.3, 9.1.4, 10.2.3

### 22.9 Systems and infrastructure
1. Log timestamps (UTC ISO, sub-millisecond)\
   Uses: 1.1.4, 16.1.1, 16.1.6, 20.3.2
2. Duration between two log lines\
   Uses: 3.1.3, 9.1.1, 15.6.1
3. HTTP caching: `Expires`, `Last-Modified`, `max-age`\
   Uses: 3.2.2, 16.2.2, 8.2.1
4. Cookie expiry\
   Uses: 8.1.1, 16.2.2, 19.3.8
5. Retry with exponential backoff: next attempt time\
   Uses: 1.1.4, 8.2.1, 15.3.3
6. Date-partitioned paths and keys (`logs/2024/03/15/`)\
   Uses: 7.1.1, 17.3.2, 19.2.6
7. Next run of a cron expression `external`\
   Uses: 10.4.9, 13.1.2, 14.3.4
8. Old `DATETIME` columns without a zone → Temporal types\
   Uses: 3.1.12, 7.1.4, 19.2.1, 19.2.2
