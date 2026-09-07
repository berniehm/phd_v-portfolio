# How to update the tracker

**Everything lives in one file:** `src/data/register.ts`
**You edit it in the browser.** No terminal, no install, no git knowledge.

---

## The loop

1. Go to the repo on github.com
2. Open `src/data/register.ts`
3. Click the **pencil** icon, top right
4. Change what you need
5. Scroll down, click **Commit changes**
6. Netlify rebuilds in about a minute — refresh the site

If you break something, GitHub's History tab lets you revert with one click. You cannot lose anything.

**Ask Bernadette for one thing:** an *Edit* link on each item's panel, pointing at
`https://github.com/{owner}/{repo}/edit/main/src/data/register.ts`
That turns the first three steps into one click. It is a single line of code.

---

## What one paper looks like in the file

```ts
{
  id: '4',
  title: 'Where CCM fails',
  status: 'active',
  court: 'victor',
  courtLabel: 'Victor',
  priority: 2,
  updated: '2026-08-25',

  tasks: [
    { text: 'CCM parameter sensitivity sweep', done: true },
    { text: 'Diagnose AC-dominated embedding', done: true },
    { text: 'Implement coupled fast/slow Lorenz', done: false, needs: 'none' },
    { text: 'Run progressive damping experiment', done: false, needs: 'none' },
    { text: 'Choose NPG or Chaos', done: false, needs: 'none' },
  ],

  actions: [
    { text: 'Run the damping experiment on Moriah',
      owner: 'Victor', leverage: 'medium', cost: '~1 hr' },
  ],
}
```

---

## The five things you will actually do

### 1. Mark something finished

Change `false` to `true`. Then change `updated` to today.

```ts
{ text: 'Run progressive damping experiment', done: true, date: '2026-09-03' },
```

Progress bars and counts recalculate on their own. Never edit a percentage — there isn't one to edit.

### 2. Add something new that came up

Add a line to `tasks`. Mind the comma at the end.

```ts
{ text: 'Rerun with tighter damping increments', done: false, needs: 'none' },
```

### 3. Record that you are waiting on someone

This is the one that makes the site earn its keep. Set `needs` to whoever holds it, and set the court.

```ts
{ text: 'Obtain cubic amplitude derivation', done: false, needs: 'Paldor' },
```

```ts
court: 'collaborator',
courtLabel: 'Victor → Paldor',
status: 'blocked',
```

Anything with a `needs` other than `'none'` drops out of the ready queue automatically. That is how the site answers "what can I start right now" when you are blocked on three other things.

### 4. Change what happens next

Rewrite the `actions` list. Keep it to one to three per paper — this is the first thing you see when you open a paper, so it should be the actual next move, not everything outstanding.

```ts
actions: [
  { text: 'Send Assaf the current state', owner: 'Victor',
    leverage: 'high', cost: 'half a day' },
],
```

### 5. Update the date

```ts
updated: '2026-09-03',
```

**Only when something real happened.** Not every time you open the file.

This is the field the ageing counter reads, and the counter is the point of the whole site. It already got this wrong once: Paper 3 read as untouched for 23 days when weekly meetings had been running throughout, because the seed date recorded when it was last *written down*, not when it last *moved*. If the dates drift, the site will confidently tell you things are stalled when they are not.

---

## Field values

| Field | Allowed |
|---|---|
| `status` | `published` · `in_review` · `active` · `blocked` · `not_started` · `paused` |
| `court` | `victor` · `assaf` · `collaborator` · `external` · `compute` · `none` |
| `priority` | `0` clear this week · `1` critical path · `2` active · `3` held elsewhere · `4` parked |
| `leverage` | `high` · `medium` · `low` |

`courtLabel` is free text — whatever reads well: `Editor`, `Victor ↔ Assaf`, `Victor → Loizou`.

---

## Two rules that will save you

**Commas.** Every entry in a list ends with a comma. A missing one blanks the site. GitHub shows the file with syntax colouring — if a whole block suddenly changes colour, that is the missing comma.

**Apostrophes.** Inside `'single quotes'`, write `\'`:

```ts
text: 'Review Paldor\'s derivation',
```

Or sidestep it entirely by using double quotes: `text: "Review Paldor's derivation",`

---

## A worked example

You run the Lorenz damping experiment on a Thursday afternoon. It works. Paldor sends the derivation the same day.

Open the file. In Paper 4:

```ts
{ text: 'Run progressive damping experiment', done: true, date: '2026-09-03' },
```
```ts
updated: '2026-09-03',
```

In Paper 3:

```ts
{ text: 'Obtain cubic amplitude derivation', done: true, date: '2026-09-03' },
```
```ts
court: 'victor',
courtLabel: 'Victor',
status: 'active',
updated: '2026-09-03',
```
```ts
actions: [
  { text: 'Check the numerics against the analytical threshold',
    owner: 'Victor', leverage: 'high', cost: 'a day' },
],
```

Commit. Ninety seconds later Paper 3 has stopped being blocked, and its next step is sitting in the ready queue.

Total time: about two minutes.

---

## Once a month, not weekly

Two things worth a slower pass:

**Debt entries.** When you make a decision that a later chapter will have to match — a domain, a threshold, a naming convention — record it while you still remember why. Ten minutes, and it is the difference between a hard month of synthesis in 2028 and a brutal one.

**Links.** As repos, DOIs and manuscripts come into existence, add them. The site indexes your work; it does not store it. Every link added now is one you are not hunting for during the write-up.
