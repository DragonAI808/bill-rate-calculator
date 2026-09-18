# Bill Rate Calculator

A single-page web tool that prices staffing requisitions. Used by Robert (staffing
sales, Kelly Professional & Industrial) and a teammate.

## Project shape

- One file: `index.html`. Everything is inline — CSS, JavaScript, no build step,
  no framework, no npm, no bundler. Keep it that way.
- The only external request is the Google Fonts stylesheet for IBM Plex Sans and
  IBM Plex Mono. Don't add other external scripts or stylesheets.
- No storage. Values reset on reload by design. Don't add localStorage or a backend.

## Deploy

GitHub `DragonAI808/bill-rate-calculator` → Cloudflare Pages, auto-deploys on push
to `main` → https://bill-rate-calculator.pages.dev

Build command and output directory are intentionally empty. There is nothing to compile.

To ship a change: `git add index.html`, `git commit -m "..."`, `git push`. Cloudflare
builds within a minute. Hard refresh (Ctrl+Shift+R) if the old page persists.

## The math (don't change without being asked)

Burden is a percentage of wages, applied flat to both straight time and overtime.

```
loaded cost = pay * (1 + burden/100)
```

Bill rate comes from one of three modes:

```
markup mode:  bill = pay * (1 + markup/100)
margin mode:  bill = cost / (1 - margin/100)      // undefined at margin >= 100
bill mode:    bill = the number the client quoted
```

Then:

```
gross profit $ = bill - cost
gross profit % = gross profit $ / bill
markup on pay  = bill / pay - 1
```

Overtime:

```
OT wage = pay * 1.5
OT cost = OT wage * (1 + burden/100)
OT bill = bill * 1.5                       // "1.5x bill rate" mode
OT bill = bill + 0.5 * pay * (1+burden/100) // "pass premium at cost" mode
```

Markup is measured against pay, margin against bill. They are never the same number.

## Burden rates by job scope (approved, do not round)

| Scope            | Burden   |
|------------------|----------|
| Light industrial | 24.716%  |
| Office           | 19.716%  |
| Light assembly   | 21.061%  |

They live in `BURDEN_BY_SCOPE` in the script. Typing in the burden field by hand
switches the scope dropdown to "Custom burden".

## Display rules

- Gross profit percentages show two decimals. Markup shows one.
- Gross profit % turns red under 10%, in both the result panel and the order table.
- The only warnings shown are: bill rate below loaded cost, and target margin >= 100%.
  A previous "check your approved floor" warning was removed on purpose. Don't re-add it.

## Order table

Rows are built once and updated in place. Do NOT re-render the whole tbody on input —
that steals focus and makes the fields untypeable. This was a real bug; keep the
in-place update pattern.

Pricing is separate from the rate card. Each line carries its own markup column
(new lines start at `DEFAULT_MARKUP`, 45%) and its own job scope, whose burden comes
from `BURDEN_BY_SCOPE`. Bill rate is pay * (1 + that line's markup). The rate card's
markup, margin, bill rate, scope, burden and overtime rule do not affect the table;
it has its own "OT billing" setting.
Editing a line's pay or markup reprices its bill rate in place. Typing a bill rate
instead reads back as that line's markup, so the two always agree.

The GRIP button in the order header swaps the three output columns for Weekly GP,
PGP $ and GRIP, and reads "Normal" while on. PGP $ uses its own rounded burdens in
`GRIP_BURDEN` (24 / 19 / 21%), not `BURDEN_BY_SCOPE`:

    PGP $ = (bill - pay * (1 + grip burden/100)) * (ST hrs + OT hrs) * heads

GRIP shows 3% and 4% of PGP $, whole dollars, separated by a slash. Nothing else on
the page changes, including Annualized GP.
