# Upskilling Labs Logic Model Webpage: Product Requirements Document

Nov 14, 2026 · @Julie

## Overview

The Upskilling Labs needs one public web page that shows how its tools, content, volunteering and learning opportunities will improve and expand over time. The page presents a logic model (inputs, activities, outputs, and short-, medium- and long-term outcomes) so funders, partners, volunteers and the core team share one picture of the work.

A first design concept is already built on a canvas (see the linked artboard in this project). This document sets the requirements to confirm before that concept is finalized and built.

**Product:** a responsive, accessible logic model page on the Upskilling Labs website.

**Status:** draft for team review. Items in [brackets] are placeholders or open decisions.

## Problem and opportunity

The Upskilling Labs is growing quickly, but its theory of change lives in conversations and slide decks rather than in one place anyone can read.

- Funders and partners see events and projects but not how they connect to lasting outcomes, which makes the case for support harder to make.
- Volunteers and members can't easily see where their contribution fits or what the community is working toward.
- The core team has no shared reference for deciding which tools, content and learning opportunities to expand first, or what to measure.

A single, public logic model addresses all three. It also gives the team a durable framework for evaluation as the Labs adds new build cycles and programs.

## Goals and non-goals

### Goals

1. Explain in under two minutes how Upskilling Labs inputs and activities lead to short-, medium- and long-term outcomes.
2. Give funders and partners a credible, shareable reference for grant applications and conversations.
3. Give the team and volunteers a working framework to prioritize tools, content, volunteering and learning work.
4. Name indicators for each outcome horizon so evaluation can start from a shared baseline.

### Non-goals

- Reporting live results or a metrics dashboard (a later phase).
- Member sign-up, event registration or donation flows; the page links to existing ones.
- A full program catalog or content library.
- A content management system; edits go through the team's existing process at first.

## Audiences and user needs

The page serves both external readers and the internal team. Funders and partners set the tone and depth; the team uses the same page as a planning reference.

| Audience | Priority | What they need from the page |
|---|---|---|
| Funders and grant reviewers | Primary | A clear chain from investment to outcomes, plus how success is measured |
| Partners (libraries, universities, employers) | Primary | How their space, time or expertise fits into the model |
| Core team and program leads | Primary | A shared framework to plan, prioritize and evaluate work |
| Volunteers and mentors | Secondary | Where their role contributes and what growth it supports |
| Members and prospective members | Secondary | What they can expect to gain, in plain language |
| Press and the general public | Secondary | A short, quotable summary of the mission and approach |

## Functional requirements

Priority uses MoSCoW: Must (launch blocker), Should (expected at launch), Could (later).

| ID | Requirement | Priority |
|---|---|---|
| F1 | Hero section with a plain-language summary of the model and a "how to read it" note | Must |
| F2 | Six-column logic model: Inputs, Activities, Outputs, then short-, medium- and long-term outcomes, readable left to right | Must |
| F3 | Each item tagged by workstream (Tools, Content, Volunteering, Learning) | Must |
| F4 | Workstream filter that highlights related items and dims the rest | Should |
| F5 | Indicators section with sample measures for each outcome horizon | Must |
| F6 | Assumptions and external factors section | Should |
| F7 | Links to existing join, donate, event and volunteer pages | Must |
| F8 | Stacked, scrollable layout on phones and tablets | Must |
| F9 | Print and PDF-friendly version for grant packets | Should |
| F10 | Shareable link to a filtered view (for example, volunteering only) | Could |
| F11 | Last-updated date and a contact for feedback in the footer | Should |

## Content requirements

Every statement on the page must be one the organization can support. The concept draft contains placeholders and sample indicators that the team has to confirm or replace before launch.

- **Structure:** each logic model column holds 4 to 6 items, each one sentence, in plain language with no jargon.
- **Time horizons:** short term is within 12 months, medium term is 1 to 3 years, long term is 3 or more years. [Confirm these ranges.]
- **Sourcing:** inputs, activities and outputs come from current program records. Outcomes come from the theory of change workshop below. Anything not yet true is written as an intended result, never a claim.
- **Numbers:** all figures such as [#] participants or [X]% targets are filled in only from real data or agreed targets. No invented statistics.
- **People and stories:** no personal details about members without their written consent.
- **Approval:** one named owner signs off on content, and one reviewer checks it for accuracy against records.
- **Updates:** review the model each quarter, aligned with the build cycle and Summit calendar, and show the last-updated date on the page.

To gather content, hold one working session with the core team, volunteer leads and one or two partners to agree on outcomes and indicators before copy is finalized.

## Design, accessibility and technical requirements

### Design

- Match the Upskilling Labs brand where one exists; the concept uses a neutral off-white ground, one accent color and a clean grotesque type pairing. [Confirm brand colors, fonts and logo with the team.]
- The model must read left to right on desktop at 1280 px and wider, and stack in the same order on phones.
- Outcome columns increase in visual weight from short to long term so the progression is visible at a glance.

### Accessibility

- Meet WCAG 2.2 AA: 4.5:1 text contrast, visible keyboard focus, and touch targets of at least 44 px.
- Filter controls are real buttons with a pressed state, and the whole page works by keyboard.
- Dimming a filtered-out item must never be the only cue; items stay readable and in their original order.
- Semantic headings and landmarks so screen readers can jump between sections.

### Technical

- Fast, static-first page with no required login and no third-party tracking beyond the team's existing analytics. [Confirm which analytics the site uses.]
- Content kept in one structured source (for example a JSON or spreadsheet file) so updates don't require redesign.
- Open Graph title and image so the page previews well when shared.
- Works in current versions of Chrome, Safari, Firefox and Edge, and on iOS and Android browsers.

## Success metrics

The page succeeds if people understand the model and use it. Targets are left blank until the team sets a baseline in the first 30 days after launch.

| Metric | How measured | Target |
|---|---|---|
| Comprehension | Short feedback prompt: "I understand how the Labs creates change" (agree / disagree) | [X]% agree |
| Engagement | Median time on page and scroll depth to the outcomes columns | [X] min / [X]% |
| Use in funder and partner work | Count of grant applications, decks or partner meetings that cite or link the page | [X] per quarter |
| Internal adoption | Team and volunteer leads who use the model in planning, by survey | [X]% of leads |
| Click-through | Visits to join, volunteer, donate and event pages from the model page | [X]% of visits |
| Accuracy | Quarterly review completed and last-updated date current | 4 reviews per year |

## Timeline, milestones and roles

The work runs in five phases, each ending at a gate that must pass before the next begins. Dates are set at kickoff.

Launch is the milestone that matters most; the three gates before it keep unsupported claims and accessibility problems off the public page.

| Role | Responsibility | Owner |
|---|---|---|
| Product owner | Decides scope, resolves open questions, approves launch | [Name] |
| Content lead | Gathers program data, writes and updates copy | [Name] |
| Accuracy reviewer | Checks claims and figures against records | [Name] |
| Design lead | Owns look, brand fit and accessibility | [Name] |
| Developer | Builds, tests and deploys the page | [Name] |
| Funder and partner advisors | Review draft for clarity and usefulness | [Names] |

## Risks, assumptions and open questions

| Risk | Likelihood | Mitigation |
|---|---|---|
| Outcomes read as promises the Labs can't yet evidence | Medium | Word long-term outcomes as intended results; tie each to an indicator |
| Content goes stale as programs change | High | Quarterly review with a named owner and a visible last-updated date |
| Too many items make the model hard to scan | Medium | Cap each column at 6 items; move detail to linked pages |
| Team disagrees on outcomes or time horizons | Medium | Run the working session before copy is final |
| Volunteer capacity limits who can maintain the page | Medium | Keep content in one simple file with written update steps |

### Assumptions

- The page lives on the existing Upskilling Labs site and can link to current join, volunteer, donate and event pages.
- The core team can provide program records for outputs.

### Open questions

- [ ] Who owns the page and signs off on content?
- [ ] Which brand assets (logo, colors, fonts) should the page use?
- [ ] Where will the page be hosted, and who deploys changes?
- [ ] Are the time horizons (12 months, 1 to 3 years, 3+ years) the right ones for funders?
- [ ] Which outcomes and indicators does the team already track?
- [ ] Is a PDF version needed for the next grant cycle?
