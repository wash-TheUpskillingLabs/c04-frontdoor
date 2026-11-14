# PRD: Participant data across National and Regional OLOS

| | |
|---|---|
| **Status** | Draft, for approval. No design or build work starts until this is approved. |
| **Date** | 2026-11-14 |
| **Pod** | Front Door, Build Cycle C04 |
| **Owner** | _To be named_ |
| **Approvers** | _To be named_ (see [Approval](#16-approval)) |
| **Source research** | [`research/olos-schema`](../research/olos-schema/index.html) (80 tables, 795 columns, c04-olos commit `63dc2df`); [`research/olos-roles`](../research/olos-roles/olos-roles.html) |

---

## 1. Summary

OLOS is The Labs' open-source platform. Today one National instance stores everything about a participant in a single `participants` table with 61 columns. Regions are setting up their own OLOS instances, starting with Greater Washington this Cycle. A person can take part in more than one region.

This PRD asks for approval to do three things:

1. Design a participant data model where **National holds a person's default information** and **each Region owns the activities and participation that happen there**.
2. Give every participant **one central view of what they are doing and have done**, across every region, over time.
3. **Capture, define and consolidate the roles** used across OLOS. There are many of them, stored in many places.

The outcome of the approved work is a proposed schema and a working demo of it. Neither is merged into OLOS as part of this work.

## 2. Background and problem

### 2.1 What the research shows

**The `participants` table carries far more than identity.** Its 61 columns fall into about a dozen groups (full list in [Appendix A](#appendix-a-the-61-columns-today)):

| Group | Columns | Examples |
|---|---|---|
| Identity and login | 7 | `email`, `google_id`, `auth_user_id`, names |
| Contact | 1 | `phone_number` |
| Location and region | 5 | `zip`, `state`, `neighborhood`, `metro_id`, `metro_slug` |
| Demographics | 3 | `gender`, `education_level`, `years_experience` |
| Work and expertise | 5 | `sector`, `sector_other`, `current_title`, `work_situation` |
| AI experience | 2 | `ai_experience_level`, `ai_tool_familiarity` |
| Consent and communications | 5 | `photo_video_consent`, `comms_consent`, `contact_consent`, `email_updates`, `text_updates` |
| Intake answers | 7 | `interest_areas`, `volunteer_interest`, `dcpl_card`, `main_focus` |
| Availability | 3 | `availability_notes`, `availability_snippet`, `commitment_notes` |
| Connected accounts | 4 | `slack_username`, `github_username`, `drive_email`, `linkedin` |
| Public profile | 5 | `handle`, `bio`, `headline`, `profile_image_url`, `public_profile_visible` |
| Agreements, referral, flags, interests, staff notes, system | 14 | `agreement_version`, `referred_by`, `is_staff`, `role_intents`, `notes`, `lab_follow_seeded` |

**Some data is stored twice:**

- `slack_username`, `github_username` and `drive_email` are also in the `connected_accounts` table.
- `agreement_version` and `agreement_accepted_at` overlap with the `agreement_acceptances` table.
- `metro_id` and `metro_slug` both point to the same region.
- `sector` is free text, alongside a separate `sectors` table.

**Some columns are already marked as old:** `dcpl_card`, `dcpl_info`, `main_focus` and `participation_commitment` belong to the intake flow from before July 2026. `dcpl_card` and `dcpl_info` are DC Public Library questions, which only make sense in one region.

**Values get overwritten, not tracked.** Consents, AI experience level and availability each hold one current value, with no history of when they changed.

**Roles are spread out.** The roles research counts 17 stored roles in `participant_roles`. On top of those come account flags (`is_staff`, `is_test`, `testers`), "member intents" (`role_intents`) and roles held in other tables (`project_roles`, `ambassador_applications`). Three older stores (`user_roles`, `moderator_assignments`, `lab_leads`) are still copied into `participant_roles` by triggers. Some roles are checked centrally and others at each call site. See [Appendix B](#appendix-b-roles-today).

**There is no model for more than one instance.** National knows about regions only as rows in `metros`. Nothing describes how a Regional instance relates to National: who owns a person's record, how a region learns about a person, or how activity in a region becomes visible to that person or to National.

### 2.2 The problem

The National `participants` table mixes data with different owners, different sensitivity and different lifespans into one row. As regions stand up their own instances, that structure can't answer basic questions:

- Which instance is the source of truth for a person's details?
- What does a region need to know about someone who joins it?
- How does a person see everything they've done across regions?
- Who is allowed to do what, and where?

## 3. Goals

| # | Goal |
|---|---|
| G1 | A participant's default information is set once, at National, and reused by every region they join. |
| G2 | Each region owns the activities and participation that happen in it. |
| G3 | One person can take part in several regions at the same time. |
| G4 | Every participant can see what they are doing and have done, across all regions, over time. |
| G5 | Participant data is organized into related tables by purpose, so each kind of data can have its own owner, access rules and history. |
| G6 | Every role in OLOS is listed, defined, given one scope, and placed at either National or Regional level. |
| G7 | The model is a pattern the next region can adopt and contribute back to the main OLOS project. |

## 4. Non-goals

- Merging any schema change into National OLOS or a Regional instance as part of this work.
- Migrating real participant data. The demo uses made-up data only, per the Cycle's rules.
- Redesigning other domains in the schema (content, events, ambassadors, surveys) beyond the links they have to participants.
- Building production sync infrastructure, authentication, or the participant-facing interface. The demo shows the idea; it is not a product.
- Settling the privacy policy for what regions can see. That is an open question for approvers ([Q1](#13-open-questions)).

## 5. Stakeholders

| Who | What they need from this |
|---|---|
| **Participants** (upskillers, volunteers, mentors) | Enter their details once. See all their participation in one place. Control what is shared. |
| **Regional lab leads and moderators** | Run cycles, pods, projects and events in their region with the participant details they need, and nothing more. |
| **National admins and staff** | One reliable record per person. A view across regions. Clear rules on who can grant what. |
| **Observers, funders, the Board** | Read-only participation figures across regions, without access to private content. |
| **OLOS maintainers and developers** | A schema with fewer duplicates, clear ownership, and a migration path from today. |
| **Future regions** | A documented pattern they can adopt when they set up their own instance. |

## 6. User needs

| # | As a… | I need to… | So that… |
|---|---|---|---|
| U1 | participant | sign up once and have my details reused when I join another region | I don't fill in the same forms again |
| U2 | participant | see my current and past cycles, pods, projects, roles and events, across every region | I can see what I've done and point others to it |
| U3 | participant | know which regions hold which of my details | I can trust The Labs with my information |
| U4 | participant | update my consent and see that the change applies everywhere | my choices are respected |
| U5 | lab lead | see the people in my region and grant roles within it | I can run my lab |
| U6 | lab lead | ask region-specific intake questions (for example, a library card) | my region's partnerships work |
| U7 | national admin | name a region's lab lead and see participation across regions | I can run the network |
| U8 | developer | know which table owns each piece of participant data | I can change OLOS without breaking another region |

## 7. Proposed direction

These directions were set by the product owner while scoping this PRD. They are the basis of the design work this PRD asks approval for. The details are worked out in the deliverables ([Section 11](#11-deliverables-after-approval)).

| # | Direction | Notes |
|---|---|---|
| D1 | **National holds a person's default information:** identity, login, contact, consent, profile. | The National record is the source of truth for who a person is. |
| D2 | **Each Region owns the activities and participation in it:** cycles, enrollments, pods, projects, events, learning logs, regional roles. | A region does not edit a person's National identity. |
| D3 | **Federated, separate databases.** Each Regional OLOS runs its own database. National is the identity hub. Regions refer to a person by a shared National ID. | Chosen over one shared database with row-level separation by region. |
| D4 | **A person can belong to several regions.** National records which regions a person belongs to. | |
| D5 | **A central participation view.** Regions send National a summary of each person's activity (what and when, not private content), and National shows the person all of it in one place. | What the summary contains is part of [Q1](#13-open-questions) and [Q6](#13-open-questions). |
| D6 | **Roles are consolidated and split by level.** There is one catalogue of role definitions. Every role is either a National or a Regional role and has one scope. Older role stores are retired. Things that aren't roles (memberships, account flags, interests) stop being stored as roles. | |
| D7 | **The 61-column table is broken into related tables** grouped by purpose (see [Section 10](#10-participant-data-direction-by-group)). | |
| D8 | **What a region can see of a person's National profile is not decided.** | [Q1](#13-open-questions) |

## 8. Requirements

"Must" means it is required for approval of the deliverables. "Should" means it is expected unless the design shows a good reason not to.

### 8.1 Functional

| # | Requirement | Priority |
|---|---|---|
| FR1 | Every participant has one National record with a stable ID that is shared with every region they belong to. | Must |
| FR2 | The National `participants` table holds only core identity. Other participant data lives in related tables grouped by purpose. | Must |
| FR3 | Every one of today's 61 columns has a documented destination: kept, moved, merged, moved to a region, or retired with a reason. | Must |
| FR4 | Data stored twice today is stored once. | Must |
| FR5 | A person can be a member of more than one region, with a status per region (for example waitlisted, active, inactive, left). | Must |
| FR6 | Regions own their activity records. National does not create or edit regional activity. | Must |
| FR7 | Regions can ask their own intake questions without changing the National schema. | Must |
| FR8 | National keeps a summary of each person's activity in each region, kept up to date from the regions. | Must |
| FR9 | A participant can see their current and past participation across all regions in one view, ordered by date and filterable by region and type. | Must |
| FR10 | Each item in the participation view links back to its source in the region's OLOS. | Should |
| FR11 | Private regional content (for example, the text of learning logs) is not copied to National. | Must |
| FR12 | Consent changes are kept as dated history and apply in every region. | Must |
| FR13 | AI experience is kept as dated history, so growth over time can be seen. | Should |
| FR14 | Free-text answers that could be pick lists (interests, volunteer interests) are identified, and a path to pick lists is proposed. | Should |
| FR15 | There is a path for a person's deletion request at National to reach every region they belong to. | Must |
| FR16 | Region membership and the participation summary still work if a region is offline for a while. Changes catch up when it reconnects. | Should |

### 8.2 Roles

| # | Requirement | Priority |
|---|---|---|
| RR1 | Every role and role-like value in OLOS today is listed, with where it's stored, its scope, how it's checked and who can grant it. | Must |
| RR2 | Every role is defined: its name, what it allows, its level (National or Regional), its scope (network, region, cycle, pod or project), and who can grant it. | Must |
| RR3 | There is one catalogue of role definitions, shared by National and every region. | Must |
| RR4 | Roles are stored in one place at each level. Older stores are retired. | Must |
| RR5 | Memberships (participant, pod member, project contributor), account flags (staff, tester, test account) and interests (upskiller, volunteer, mentor, events) are kept separately from roles. | Must |
| RR6 | The rule that only an owner can grant owner is kept. | Must |
| RR7 | A role can only be granted at the scope its definition allows. | Should |
| RR8 | Every grant and revocation records who did it and when. | Must |
| RR9 | It is clear how a role granted by National but used in a region (for example, lab lead) gets to that region. | Must |

### 8.3 Non-functional

| # | Requirement | Priority |
|---|---|---|
| NFR1 | Sensitive data (demographics, staff notes, contact details, library card numbers) can have tighter access rules than the rest of the profile. | Must |
| NFR2 | The design works on Supabase/Postgres, which OLOS already uses. | Must |
| NFR3 | Sending activity from a region to National can safely be repeated without creating duplicates. | Must |
| NFR4 | There is a migration path from today's National schema that doesn't lose data, and which can be checked by rebuilding today's 61-column shape from the new tables. | Must |
| NFR5 | Changes are documented well enough that they can be offered back to the main OLOS project as a pull request. | Must |
| NFR6 | The demo uses made-up data only. | Must |

## 9. Roles

### 9.1 What exists today

The roles research describes 17 stored roles. Of these, 13 are named:

`owner`, `admin`, `developer`, `observer`, `staff`, `tester`, `lab_lead`, `poderator` (moderator), `co_lead`, `member`, `dri`, `contributor`, `ambassador`

The other four are not identified in the research ([Q10](#13-open-questions)). Alongside the stored roles, OLOS also uses:

- A "Participant" status worked out from cycle enrollment.
- Account flags: `is_staff`, `is_test` and the `testers` table.
- Four "member intents": upskiller, volunteer, mentor and events. These are used only as directory filters, not as permissions.

Roles are stored across eight places. The full inventory is in [Appendix B](#appendix-b-roles-today).

### 9.2 Direction for approval

Following D6, the design work will:

1. **Catalogue** every role-like value and define each one (RR1, RR2).
2. **Sort** each into one of five kinds:

   | Kind | Meaning | Candidates from today |
   |---|---|---|
   | National role | Grants authority across the network | owner, admin, developer, observer |
   | Regional role | Grants authority within a region, cycle, pod or project | lab lead, moderator, co-lead, DRI, ambassador |
   | Membership | Being part of something, not authority over it | participant (cycle), pod member, project contributor |
   | Account flag | A label with no authority | staff, tester, test account |
   | Interest | What a person wants to do | upskiller, volunteer, mentor, events |

   This sorting is a starting proposal to be confirmed in design, not a decision.
3. **Consolidate** storage into one catalogue and one grant table per level, and retire the older stores (RR3, RR4).

## 10. Participant data: direction by group

This is the starting direction for each group. The deliverables will confirm or change it and give the full column-by-column destination (FR3).

| Group | Where it would live | Owner |
|---|---|---|
| Core identity (names, login email, created, archived) | `participants` | National |
| Login identities (`google_id`) | Related table, one row per login provider | National |
| Contact (email, phone) | Related table, one row per contact method | National |
| Connected accounts (Slack, GitHub, Drive, LinkedIn) | One related table, merging `connected_accounts` | National |
| Location | Related table | National |
| Demographics | Related table with its own access rules | National |
| Work and expertise | Related table, `sector` matched to `sectors` | National |
| AI experience | Dated history | National |
| Consent and communications | Dated history | National |
| Agreements | One table, merging `agreement_acceptances` | National (network agreements); Region (cycle agreements, which `cycle_agreements` already does) |
| Availability | Related table | National ([Q8](#13-open-questions)) |
| Intake answers | Versioned answers; pick lists where they fit | National for network questions; Region for regional questions (for example `dcpl_card`) |
| Public profile | Related table | National ([Q9](#13-open-questions)) |
| Referral | Related table | National |
| Staff notes | Related table with author and date | National or Region ([Q7](#13-open-questions)) |
| Flags and onboarding state | Related tables | National |
| Region | Region memberships, one row per region a person belongs to | National |
| Activities (cycles, pods, projects, events, logs) | Already separate tables | Region |

## 11. Deliverables after approval

The approved work produces the following. None of it is merged into OLOS.

| # | Deliverable | Description |
|---|---|---|
| O1 | **Proposed schema** | National and Regional schemas with tables, keys and constraints, and written reasons for each choice. |
| O2 | **Column map** | Where each of the 61 columns goes, and why (FR3). |
| O3 | **Role catalogue** | Every role defined, sorted by kind, level and scope, with who can grant it (RR1–RR9). |
| O4 | **Federation contract** | What National sends a region when a person joins. What a region sends National about activity. How deletion and updates flow. |
| O5 | **Migration plan** | Steps from today's National schema, including a check that rebuilds today's 61-column shape from the new tables (NFR4). |
| O6 | **Demo: database** | Runnable SQL for the proposed schema, with made-up seed data showing one person active in two regions, the sync, and the guard rails. |
| O7 | **Demo: participation view** | A clickable prototype of a participant's "what I'm doing and have done" view across regions. |
| O8 | **Demo: schema diagram** | Today's model next to the proposed one. |
| O9 | **Demo: role matrix** | An interactive table of every role: kind, level, scope, who grants it and what it allows. |
| O10 | **Pull request draft** | A write-up suitable for offering the pattern back to the main OLOS project. |

All deliverables go on a branch for review. Code that targets the region's OLOS fork follows the Cycle's branch naming (`c04-frontdoor-projectname`).

## 12. Milestones

Dates are to be set by approvers.

| Milestone | Contents | Date |
|---|---|---|
| M0 | PRD approved; open questions that block design answered (at least Q1, Q2, Q3, Q10) | _TBD_ |
| M1 | Role catalogue (O3) and column map (O2) reviewed | _TBD_ |
| M2 | Proposed schema (O1) and federation contract (O4) reviewed | _TBD_ |
| M3 | Demos (O6–O9) shown to at least three real people (a participant, a lab lead and a national admin) | _TBD_ |
| M4 | Migration plan (O5) and pull request draft (O10) ready | _TBD_ |

## 13. Open questions

| # | Question | Why it matters | Owner |
|---|---|---|---|
| Q1 | **What can a region see of a person's National profile?** Options: (a) the participant chooses per region; (b) a fixed standard set; (c) a fixed set plus things the participant opts into. | Sets the contract between National and regions, and the privacy promise to participants. | _TBD_ |
| Q2 | Who grants a **lab lead**: National, the region, or both? If National, does the region confirm it? | Lab lead is the role that sits across the boundary between National and Region. | _TBD_ |
| Q3 | What, if anything, can **National admins** do inside a Region's instance? Read-only? Logged? | Affects trust between regions and National. | _TBD_ |
| Q4 | Should a region keep its own copy of a person's **email and name**, or look them up from National each time? | Copies make offline work possible; lookups keep one source of truth. | _TBD_ |
| Q5 | How does a **deletion request** at National reach every region, and how is completion confirmed? | Needed for FR15, and for any privacy commitments. | _TBD_ |
| Q6 | **How does activity reach National?** Regions push it, National pulls it, or a scheduled job? How fresh must the participation view be? | Shapes the federation contract (O4). | _TBD_ |
| Q7 | Are **staff notes** about a person National, regional, or both? Who can read them? | Notes are sensitive and often written in a regional context. | _TBD_ |
| Q8 | Is **availability** the same across regions, or set per region? | A person may give different time to different regions. | _TBD_ |
| Q9 | Is the **public profile** one profile across the network, or per region? | Affects follows, directories and page profiles. | _TBD_ |
| Q10 | Which **four stored roles** are missing from the research's list of 13? | Needed for a complete catalogue (RR1). Answer is in the `participant_roles` constraint in the c04-olos migrations. | _TBD_ |
| Q11 | Who are the **"coordinators"** who review ambassadors? Is that a role? | The ambassador grant depends on it. | _TBD_ |
| Q12 | Does a region need its **own admin role** (for example a deputy lab lead)? | Keeps the number of roles in check. | _TBD_ |
| Q13 | What are the allowed values of **`ai_experience_level`**? | Not in the research ERDs. | _TBD_ |
| Q14 | How are **event RSVPs** from people without an account (email only) handled in the participation view? | `event_rsvps.participant_id` is optional today. | _TBD_ |
| Q15 | Should **library card numbers** (`dcpl_card`) be kept at all, and if so, under what protection? | It's an ID number belonging to a partner institution. | _TBD_ |
| Q16 | How do we handle **one person with two accounts** (for example, two emails) when regions merge their records? | Federation makes duplicates more likely. | _TBD_ |
| Q17 | Which participation should count toward the central view? Learning logs and RSVPs, or only cycles, teams and roles? | Sets what the person, and observers, see. | _TBD_ |

## 14. Risks

| Risk | Effect | How we'd reduce it |
|---|---|---|
| Separate databases drift apart in schema | Regions can't send compatible activity summaries | A shared, versioned contract (O4) and role catalogue (RR3) |
| The participation summary goes stale or duplicates | Participants see wrong history | Repeat-safe sync (NFR3); each record keeps its region's last-updated time |
| Sensitive data copied to more places than needed | Privacy exposure | Settle Q1 before design; keep private content in regions (FR11) |
| Retiring older role stores breaks existing permission checks | People lose or gain access unexpectedly | Full inventory first (RR1); demo the guard rails; migration plan (O5) |
| Scope grows into a full OLOS rewrite | Nothing ships | Non-goals ([Section 4](#4-non-goals)); nothing merged in this work |

## 15. How we'll know it worked

- Every one of the 61 columns has an approved destination (FR3).
- Every role has an approved definition, kind, level and scope (RR1, RR2).
- In the demo, one made-up person active in two regions sees all their participation in one view (FR9).
- In the demo, today's 61-column shape can be rebuilt from the new tables, and any differences are explained (NFR4).
- At least three real people (a participant, a lab lead and a national admin) have used the demos, and what they did is written down (M3).
- Approvers agree the design is ready to offer to the main OLOS project as a pull request.

## 16. Approval

| Role | Name | Decision | Date |
|---|---|---|---|
| Product owner | | | |
| National OLOS maintainer | | | |
| Greater Washington lab lead | | | |
| Builder in residence | | | |

---

## Appendix A: the 61 columns today

From `participants` in National OLOS (c04-olos commit `63dc2df`). "Notes" records only what the research shows. Destinations are part of deliverable O2.

| # | Column | Type | Group | Notes |
|---|---|---|---|---|
| 1 | `id` | serial, PK | Identity | |
| 2 | `auth_user_id` | uuid, unique | Identity | |
| 3 | `google_id` | varchar(255), unique, not null | Identity | |
| 4 | `email` | varchar(255), unique, not null | Identity | |
| 5 | `first_name` | varchar(255), not null | Identity | |
| 6 | `last_name` | varchar(255), not null | Identity | |
| 7 | `preferred_name` | varchar(255) | Identity | |
| 8 | `gender` | varchar(100) | Demographics | Sensitive |
| 9 | `state` | varchar(10) | Location | |
| 10 | `neighborhood` | varchar(255) | Location | |
| 11 | `dcpl_card` | varchar(20) | Intake | Old intake; DC Public Library; sensitive |
| 12 | `dcpl_info` | boolean | Intake | Old intake; DC Public Library |
| 13 | `work_situation` | varchar(50) | Work | |
| 14 | `main_focus` | varchar(50) | Intake | Old intake |
| 15 | `sector` | varchar(255) | Work | Free text alongside `sectors` table |
| 16 | `current_title` | varchar(255) | Work | |
| 17 | `linkedin` | varchar(500) | Connected accounts | |
| 18 | `ai_tool_familiarity` | smallint | AI experience | Single value, no history |
| 19 | `participation_commitment` | varchar(20) | Intake | Old intake |
| 20 | `primary_expertise` | varchar(500) | Work | |
| 21 | `volunteer_interest` | varchar(500) | Intake | Free text |
| 22 | `text_updates` | boolean | Consent | Single value, no history |
| 23 | `photo_video_consent` | boolean, not null | Consent | Single value, no history |
| 24 | `source` | varchar(255) | Referral | |
| 25 | `slack_username` | varchar(255) | Connected accounts | Also in `connected_accounts` |
| 26 | `github_username` | varchar(255) | Connected accounts | Also in `connected_accounts` |
| 27 | `drive_email` | varchar(255) | Connected accounts | Also in `connected_accounts` |
| 28 | `created_at` | timestamp | System | No time zone |
| 29 | `profile_image_url` | text | Public profile | |
| 30 | `last_pulse_completed_at` | timestamp | System | Could be worked out from `pulse_checks` |
| 31 | `phone_number` | varchar(50) | Contact | Sensitive |
| 32 | `email_updates` | boolean | Consent | Single value, no history |
| 33 | `comms_consent` | boolean, not null | Consent | Single value, no history |
| 34 | `availability_notes` | text | Availability | |
| 35 | `commitment_notes` | text | Availability | |
| 36 | `interest_areas` | text | Intake | Free text |
| 37 | `moderator_experience` | text | Intake | |
| 38 | `notes` | text | Staff | One field, no author or date; sensitive |
| 39 | `contact_consent` | boolean, not null | Consent | Single value, no history |
| 40 | `ai_experience_level` | enum, not null | AI experience | Values not in research (Q13) |
| 41 | `availability_snippet` | text | Availability | |
| 42 | `zip` | varchar(10) | Location | |
| 43 | `metro_slug` | varchar(50) | Region | Duplicates `metro_id` |
| 44 | `role_intents` | text[], not null | Interests | Used as directory filter, not permissions |
| 45 | `referred_by` | varchar(255) | Referral | Free text, not linked to a person |
| 46 | `agreement_version` | varchar(50) | Agreements | Overlaps `agreement_acceptances` |
| 47 | `agreement_accepted_at` | timestamptz | Agreements | Overlaps `agreement_acceptances` |
| 48 | `is_staff` | boolean, not null | Flags | Also a stored role |
| 49 | `is_test` | boolean, not null | Flags | |
| 50 | `handle` | varchar(50) | Public profile | |
| 51 | `bio` | text | Public profile | |
| 52 | `headline` | varchar(200) | Public profile | |
| 53 | `public_profile_visible` | boolean, not null | Public profile | |
| 54 | `metro_id` | integer | Region | One region only |
| 55 | `years_experience` | varchar | Demographics | |
| 56 | `education_level` | varchar | Demographics | |
| 57 | `sector_other` | varchar | Work | |
| 58 | `created_via` | varchar, not null | System | |
| 59 | `lab_follow_seeded` | boolean, not null | System | Onboarding state |
| 60 | `page_follows_seeded` | boolean, not null | System | Onboarding state |
| 61 | `archived_at` | timestamptz | System | |

## Appendix B: roles today

From [`research/olos-roles`](../research/olos-roles/olos-roles.html).

| Name today | Stored in | Scope | How it's checked | Who can grant it |
|---|---|---|---|---|
| `owner` | `participant_roles`, `user_roles` | Global, super admin | `resolveUserRoles` | Owner only (`guard_owner_grant` trigger) |
| `admin` | `participant_roles`, `user_roles` | Global | `resolveUserRoles`, `requireAdmin` | Admin |
| `developer` | `participant_roles` | Global | `resolveUserRoles` | Admin |
| `observer` | `participant_roles`, `participant_permissions` | Global, read-only | `resolveUserRoles` | Admin |
| `staff` | `participant_roles`, `participants.is_staff` | Global flag, no authority | Hidden from rosters | Admin |
| `tester` | `participant_roles`, `testers` | Global flag, no authority | `testing:use` | Admin |
| `lab_lead` | `participant_roles`, `lab_leads` | Lab (metro) | `requireLabAccess` | Admin |
| `poderator` | `participant_roles`, `moderator_assignments` | Pod | `isModeratorForPod` | Admin, or the lab's lead |
| `co_lead` | `participant_roles` | Pod (org workstream runs) | At the call site | Admin, or the lab's lead |
| `member` | `participant_roles` | Pod | At the call site | Admin, the lab's lead, or accepting an invite |
| `dri` | `participant_roles`, `project_roles` | Project | `isProjectDri` | Admin, or a DRI |
| `contributor` | `project_roles` | Project | At the call site | Admin, or a DRI |
| `ambassador` | `ambassador_applications` | Lab | At the call site | Two coordinator reviews, or an approved invite |
| Participant | Worked out from `cycle_enrollments` | Cycle | `isEnrolledParticipant` | Enrolling in a cycle |
| Test account | `participants.is_test` | Global flag | Testing reset | Admin |
| Member intents: upskiller, volunteer, mentor, events | `participants.role_intents` | None | Directory filters only | The person themselves |
| _Four more stored roles_ | `participant_roles` | _Unknown_ | _Unknown_ | _Unknown_ (Q10) |

Where roles are stored today: `participant_roles`, `user_roles`, `moderator_assignments`, `lab_leads`, `participant_permissions`, `project_roles`, `ambassador_applications`, and flag columns on `participants` along with the `testers` table. Admins pass every scope check.

## Appendix C: terms

| Term | Meaning |
|---|---|
| **OLOS** | The Labs' open-source platform. |
| **National OLOS** | The national organization's instance (the Upskilling App). |
| **Regional OLOS** | A region's own instance, for example Greater Washington's. |
| **Region / lab / metro** | One local Labs community. Stored today as a row in `metros`. |
| **Federated** | Each instance has its own database; they share an ID for each person and exchange agreed data. |
| **Participation view** | One place where a person sees everything they are doing and have done, across regions. |
| **Scope** | Where a role applies: network, region, cycle, pod or project. |
| **DRI** | Directly responsible individual for a project. |
