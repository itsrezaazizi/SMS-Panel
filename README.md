# SMS Campaign Builder

**Single-file interactive prototype of an SMS campaign builder for small retail: pick an audience, write the message, see the cost before you send.**

🔗 **[Live demo](https://YOUR-USERNAME.github.io/REPO-NAME/)** · No sign-up, nothing is sent, all data is mock.

![The campaign builder, showing audience selection, message composer and the live cost summary](screenshots/overview.png)

---

## The problem

A shop owner with a loyalty list wants to text their customers about a weekend sale. The tools available to them are built for marketers: audience builders with boolean logic, credit systems that fail silently, and a send button that gives no idea what it will cost until after the fact.

The mistakes that actually hurt are not design mistakes. They are sending to people who never opted in, blowing a month's budget on one campaign, scheduling a message for a time that has already passed, and discovering any of it only afterwards.

So the prototype is built around one idea: **never let someone send blind.**

## The approach

Three decisions, in the order a person actually thinks about them — who receives it, what it says, when it goes out — with a summary panel that stays visible and recalculates as you go.

### Cost is never a surprise

Every change updates the campaign cost immediately. Two separate limits are tracked and explained, because people conflate them: the **campaign budget** is the ceiling you set for this send; the **account balance** is the credit left in your SMS account. Running past either produces a different problem with a different fix.

![The summary panel with cost, budget and balance meters, and a blocking error with a one-click fix](screenshots/summary.png)

### Every error carries a way out

Problems are not just reported. Over budget offers to trim the list to the highest-spending customers who fit. An empty message offers a starting template. A time in the past offers to send now. Blocking errors disable the send button; warnings do not. Any automatic trim is reversible in one click.

### Consent is structural, not a checkbox

Customers who have not opted in are excluded from every count, and the exclusion is stated plainly: *1,024 people are in this group. 324 haven't opted in to SMS and were left out. The message reaches 700 people.* The legally required opt-out line is appended automatically and cannot be edited away.

### The preview shows real substitution

Dynamic tags — first name, store link, store address — render filled in with an actual customer's details, highlighted, in a phone-style bubble. The character counter measures the **filled** length, since that is what gets billed.

![The message composer with dynamic tags and the filled-in preview](screenshots/composer.png)

---

## Try the edge cases

The interesting states are the failure states, so they are reachable directly — via the "Test states" panel in the header, or by URL:

| URL | State |
|---|---|
| [`?test=empty`](https://YOUR-USERNAME.github.io/REPO-NAME/?test=empty) | Filters that match nobody |
| [`?test=budget`](https://YOUR-USERNAME.github.io/REPO-NAME/?test=budget) | Cost exceeds the campaign budget |
| [`?test=credit`](https://YOUR-USERNAME.github.io/REPO-NAME/?test=credit) | Not enough account balance |

---

## Design notes

**Restraint over decoration.** Hierarchy comes from borders, spacing and type, not shadows. Blue means *selected* or *in progress*; the primary action is near-black, so blue never competes with it. Red blocks, amber informs.

**Tight geometry.** Corner radii top out at 10px and there are no capsule shapes — a deliberate move away from the rounded default that makes internal tools read as toys.

**Typography does the work.** Inter for interface text, Inter Tight for headings and figures, tabular numerals throughout so numbers don't jitter as the cost recalculates. Micro-caps for section labels instead of heavier weights.

**Bilingual history.** This started as a Persian, right-to-left product, and was later rebuilt in English for a US retail context — currency, SMS encoding limits (160-character GSM-7 rather than 70-character UCS-2), weekday conventions and opt-out norms all differ between the two. Some of the layout decisions still carry that origin.

---

## Running it

Clone and open `index.html`. That is the whole process.

```
git clone https://github.com/YOUR-USERNAME/REPO-NAME.git
cd REPO-NAME
open index.html
```

No build step, no dependencies, no backend. One HTML file with inline CSS and JavaScript, plus inline Phosphor icon paths. Fonts load from Google Fonts and fall back to the system UI stack offline.

Customer data is generated from a seeded PRNG, so the dataset is identical on every load — the numbers in screenshots stay true.

## Repository

```
index.html      the prototype
HANDOFF.md      technical documentation: architecture, state model, business rules, gotchas
screenshots/
```

Working on the code? Start with [HANDOFF.md](HANDOFF.md).

---

## Status

A design prototype, not production software. No SMS is sent, no data leaves the browser, and there is no backend. Built to explore the interaction design of a high-consequence, irreversible action.

Possible next steps: projected campaign results, send history, per-recipient message segmentation, A/B message split, and a real language toggle rather than two diverged files.
