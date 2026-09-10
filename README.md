# My Notes (Mohamed Khaled)

## Running this

```bash
npm install
npm run eval      # or: npx promptfoo eval
npm run view      # or: npx promptfoo view
```

Note: on my machine (Node v22.16.0) `promptfoo` refused to start through the
normal command — it wants Node `^20.20.0` or `>=22.22.0`. If you hit the same
issue, this works around it (same eval, just skips the version check):

```bash
node node_modules/promptfoo/dist/src/main.js eval
```

Last run: **10 passed, 14 failed** out of 24 tests. A red run is expected here
— it means the tests are doing their job.

## What I tested

24 test cases, split into three groups: happy path (greetings, order status,
product info, shipping), edge cases (empty input, whitespace, a 10,000-char
message, unsupported languages, Spanish since it should be supported), and
adversarial/off-spec cases (prompt injection, trying to get order details out
of the bot, asking for a human, lost/damaged packages, refund tone, a legal
advice question that should be refused).

I hadn't used promptfoo before, so I spent some time reading through its docs
to understand how providers, vars, and assertions fit together before writing
anything. For assertions I used five types in the end (`is-json`,
`javascript`, `contains`, `icontains`, `regex`) — more than the three asked
for. `javascript` does most of the real work since it's the only one that can
parse the JSON and check specific fields like `intent` or `card.currency`.

## What I found

Nine bugs, across 14 failing tests:

1. **Currency is always hardcoded to USD** on order/product cards, even
   though A1043 and P100 are EUR in the sample data. Breaks Rule 2.
2. **Empty/whitespace input returns broken JSON** — not even parseable.
   Breaks Rule 7.
3. **Unsupported languages (French, German) don't fall back to English** —
   the bot replies in that language instead. Breaks Rule 5.
4. **Prompt injection works** — asking it to ignore instructions, mentioning
   a random email, or claiming to be an admin all get another customer's
   order details handed over. Breaks Rule 6, and the one that concerned me
   most.
5. **Human handoff doesn't escalate** — `escalate` is hardcoded `false` even
   when the customer explicitly asks for a person. Breaks Rule 4.
6. **Website complaints wrongly escalate** as if they were a damaged
   package, which the spec says shouldn't happen. Also Rule 4.
7. **Refund replies blame the customer** — literally says "you should have
   checked the return window." Breaks Rule 3.
8. **Returns/refunds never attach the order card**, even when an order is
   named. Breaks Rule 1.
9. **Legal-advice requests aren't refused** — falls through to the generic
   fallback instead. Breaks Rule 6.

Each bug is caught by at least one test in `promptfooconfig.yaml` — the test
descriptions match up with the bug descriptions above.

## A couple of things about the spec itself

Rule 4's "website complaint isn't a damaged package" rule is hard to test
cleanly, since the spec doesn't say how to actually tell those two apart.
And the spec doesn't say what `intent` an out-of-scope refusal like the
legal-advice one should carry when no order is involved — I went with
`intent: "refusal"` since that's the closest fit.

## Assumptions

- Delivery dates: checked future/past relative to `new Date()`, not a literal
  date, since the fixture dates move with the day you run the suite.
- `escalate` checked as a strict boolean, not just truthy.

## AI usage

This was genuinely challenging for me since I hadn't used promptfoo before,
so alongside reading the docs myself I used an AI assistant to help me get
oriented and to help write the test cases and assertions in
`promptfooconfig.yaml`. It also helped me catch a couple of my own mistakes —
multi-line `javascript` assertions need an explicit `return` or they error
out, and `(?i)` isn't valid in JS regex, both of which I got wrong the first
time and only noticed once the eval actually threw errors. I still ran
everything myself and checked the real output before trusting any bug on
the list above.

It was a challenging assignment given I was new to the tool, but I enjoyed
working through it — thanks for putting it together.

## Stretch

I focused on the Core section and didn't attempt Stretch — ran out of time
after getting Core solid.