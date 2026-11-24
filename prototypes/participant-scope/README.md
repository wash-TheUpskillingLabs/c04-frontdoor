# Participant scoping session

A blank copy of the participants column-scoping form, for one facilitated session with National and Regional stakeholders. The page is at [/prototypes/participant-scope/](https://c04-frontdoor.vercel.app/prototypes/participant-scope/).

## What it's for

Coming to an understanding of which OLOS owns which fields in the `participants` table: National, Regional, Both, a New table, or Unsure.

This is a decision-focused collaborative task, not a discovery probe. In the builder's words, a stable `participants` structure is the pivot point that other data structure changes, analytics, and any API or MCP depend on.

## Which finds it came from

- #8 Record: OLOS schema and data model, partial review
- `research/olos-schema/`, especially `olos-table-usage.md`, which covers where columns are and aren't used in the code, including orphaned fields
- `research/participant-scope/`, which holds the builder's strawman pass through the form and the draft data dictionary
- Leadership discussions that named participant data as the core issue in splitting National and Regional OLOS. *Not yet logged as an issue.*

## Who will see it

At least one National stakeholder and at least one Regional stakeholder in the room together, plus a facilitator. The session takes about an hour. The date is not set yet because of the holidays.

## Prediction

None, by choice. The builder's reasoning: with data structures, going in with a predicted outcome tends to seed poor outcomes. The builder's own solo pass (the strawman in `research/participant-scope/`) was for finding problems with the form, not for predicting the room's decisions. That pass stays on record, so it can be compared with the session's result afterwards.

## What's on the page

- All 61 `participants` columns, each with five choices (National · Regional · Both · New table · Unsure) and a short note. Every decision starts blank.
- A session-notes panel pinned to the right of the form. On a narrow screen it moves below the form.
- A link that opens the [data dictionary](https://c04-frontdoor.vercel.app/research/read/?doc=participant-scope/participants-data-dictionary.md) in a new tab for reference.
- **Download note (.md)**, which saves the decisions and session notes as `participants-scoping.md`. Commit that file over the blank one in this folder.

**Deliberately left out:**
- The other 79 tables.
- Flags on orphaned fields. Some may have a purpose or belong in another table, so the clean-up is a later, separate piece of work.
- The strawman decisions.

## Script

<!-- TODO(human): the builder writes these three lines. -->

1. **Set the scene:**
2. **Hand it over:**
3. **Debrief:**

## What we learned

*Fill this in the same day as the session, while it's fresh.* Cover:
- Which columns were settled, and where.
- Which stayed Unsure, and why.
- Where National and Regional saw a field differently.
- What the data dictionary couldn't answer.
- How the result compares with the strawman.
