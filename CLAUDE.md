# Bill Rate Calculator

A single-page web tool that prices staffing requisitions. Used by Robert (staffing
sales, Kelly Professional & Industrial) and a teammate.

## Project shape

- One file: `index.html`. Everything is inline — CSS, JavaScript, no build step,
  no framework, no npm, no bundler. Keep it that way.
- The only other file is `kelly-logo.png`, the white Kelly wordmark in the header.
  It is white artwork on transparency, so print rules render it black.
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
loaded cost = pay * (1 + burden/100)   // labelled "Pay + burden" on screen
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
- The only warnings shown are: bill rate below pay plus burden, and target margin >= 100%.
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

GRIP shows 3% and 4% of PGP $, whole dollars, separated by a slash. In GRIP view the
last row reads "Monthly PGP $": total PGP * weeks billed / 12, because GRIP pays out
monthly. Nothing
outside the order table changes.

## Order table toolbar

- **Copy order** puts the table on the clipboard as tab-separated rows (header, lines,
  order total, the monthly or annualized line, plus weeks billed and the OT rule), so it
  pastes into Outlook or Excel. Columns follow the current view. If the browser refuses
  the clipboard, the text appears in an overlay, selected, to copy by hand.
- **Print** (and Ctrl+P) uses the print rules: the rate card, result panel, toolbar and the
  row buttons are hidden, and in GRIP view the GRIP column and Monthly GRIP are hidden too,
  since commission figures should not reach a client sheet.
- The ⧉ button on a row duplicates it directly below; rows stay independent afterwards.
- In GRIP view the last row shows Monthly PGP $ and, beside it, Monthly GRIP: 3% / 4% of
  that monthly figure.

## Colors

Kelly greens: #023618 carries the dark surfaces (bill rate panel, divider,
primary buttons, selected pricing tab) and #00AA34 (`--brand-bright`) the card headings.
The header bar runs a gradient between the two, bright on the left to dark on the right.
`--navy-lift` (#04542A) is the lighter shade for hovers and the rule under the title bar.
The token names still say navy for historical reasons.

## Direct hire order

A separate card below the staffing order, one placement at a time:

    annualized salary = hourly pay * 2080
    DH $ = fee % * (annualized salary, or the salary typed directly)

GRIP on that fee is tiered (`DH_GRIP_BREAK`): 6% on fees under $5,000, 10% from
$5,000 up. The rate is shown in small type beside the amount.

Typing an hourly rate clears the salary field and vice versa, so the field touched
last is the basis (`dhFrom`). Annualized salary is read-only. The fee starts at 20%.
It is internal pricing, so it is left out of Copy order and hidden when printing.
