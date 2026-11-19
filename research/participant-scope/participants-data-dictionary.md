# `participants` data dictionary

What each column of the OLOS `participants` table holds, based only on evidence in the source code. Nothing here is guessed. Where the code doesn't settle a column's meaning, the entry says so and lists what to ask.

- **Source:** [`wash-TheUpskillingLabs/c04-olos`](https://github.com/wash-TheUpskillingLabs/c04-olos) at commit [`ca543cc`](https://github.com/wash-TheUpskillingLabs/c04-olos/tree/ca543cc01fb3d7e6ef17d2281832e766b56ec0c3) (2026-11-17). That's newer than the commit the ERDs were built from (`63dc2df`). No migration between the two commits changes `participants`.
- **Evidence used:** the 113 migrations in `supabase/migrations/`; the comments written into the database (`COMMENT ON COLUMN`); the sign-up form and its API; the profile editor and its validation rules; the legacy spreadsheet import (`scripts/migration/`), whose `column_mapping.csv` ties each original registration-form question to a column; and the code that reads each column.
- **Not used:** Slack, meeting notes and production data. Nothing here shows how many rows actually hold a value. The [value-count query](#query-to-check-real-data) at the end answers that for someone with database access.
- **Built:** 2026-11-19.

## How to read an entry

**Confidence**
- **Confirmed:** the code states what the column holds, through the question wording, a comment in the database or code, or allowed values that make it plain.
- **Partial:** what it holds is clear, but something material isn't. For example, the name doesn't match what's stored, or values from different eras mean different things.
- **Unknown:** the code doesn't say what it means.

**Status** says whether anything still fills the column in:
- **Active:** current code writes it.
- **Legacy:** only the old registration form wrote it, through the one-time spreadsheet import. Some of these are still editable in the profile.
- **Orphaned:** nothing in the code writes it.

"Legacy form" below means the original Google Form, *Upskiller Registrations*. Its rows were loaded by `scripts/migration/migrate.py`, following `scripts/migration/column_mapping.csv`. Questions are quoted as they appear in that file.

"Sign-up form" means the current registration funnel: `app/(auth)/register/funnel.tsx`, which posts to `app/api/registrations/funnel/route.ts`.

## Summary

| Column | Status | Confidence | In one line |
|---|---|---|---|
| `id` | Active | Confirmed | Row number, the internal participant key |
| `auth_user_id` | Active | Confirmed | Link to the person's Supabase login |
| `google_id` | Active | **Partial** | Named "Google ID", but every writer stores the email address |
| `email` | Active | Confirmed | Login and identity key, unique ignoring case |
| `first_name` | Active | Confirmed | First name |
| `last_name` | Active | Confirmed | Last name |
| `preferred_name` | Active | Confirmed | Name to use instead of the first name |
| `gender` | Legacy | **Partial** | Free-text answer to "Gender"; nothing sets or shows it now |
| `state` | Legacy, editable | Confirmed | Where based: MD, DC, VA or Other |
| `neighborhood` | Legacy, editable | Confirmed | DMV neighborhood, free text |
| `dcpl_card` | Legacy, editable | Confirmed | Has a DC Public Library card: yes, no or not sure |
| `dcpl_info` | Legacy | Confirmed | Wanted information on getting a library card |
| `work_situation` | Active | Confirmed | Employment situation, 7 fixed values |
| `main_focus` | Legacy, editable | Confirmed | Goal for the next few months, for people in transition |
| `sector` | Legacy, editable | Confirmed | Self-described sector or field, free text; not the `sectors` table |
| `current_title` | Legacy, editable | Confirmed | Current or most recent job title (old rows also include the organization) |
| `linkedin` | Legacy, editable | Confirmed | LinkedIn profile |
| `ai_tool_familiarity` | Legacy, editable | Confirmed | 1–5: how often they use AI tools |
| `participation_commitment` | Legacy | Confirmed | Ready to commit (`yes`) or availability uncertain |
| `primary_expertise` | Legacy, editable | Confirmed | Primary domain or area of expertise |
| `volunteer_interest` | **Orphaned** | **Partial** | Meant for "Would you consider helping steward the community?", but the import never wrote it |
| `text_updates` | Legacy | Confirmed | Opted in to text updates. Marked LEGACY in the database |
| `photo_video_consent` | Active | **Partial** | Photo and video consent; new sign-ups get `true` without being asked |
| `source` | Active | **Partial** | How they heard about The Labs; three different vocabularies |
| `slack_username` | **Orphaned** | **Partial** | Slack name; nothing writes it. Replaced by `connected_accounts` |
| `github_username` | **Orphaned** | **Partial** | GitHub username; nothing writes it. Replaced by `connected_accounts` |
| `drive_email` | **Orphaned** | **Partial** | Google Drive sharing email; nothing writes or reads it |
| `created_at` | Active | Confirmed | When the row was created |
| `profile_image_url` | Active | Confirmed | Avatar image URL |
| `last_pulse_completed_at` | Active | Confirmed | When their last weekly pulse check was completed |
| `phone_number` | Legacy and pod leaders | Confirmed | Phone for day-of event updates |
| `email_updates` | Legacy | Confirmed | Opted in to email updates. Marked LEGACY in the database |
| `comms_consent` | Legacy | Confirmed | Accepted "consent to receive communications". Marked LEGACY in the database |
| `availability_notes` | Legacy | Confirmed | "Anything we should know about your availability?" |
| `commitment_notes` | Legacy | Confirmed | What would make participation workable, if uncertain |
| `interest_areas` | Legacy | Confirmed | Areas they're most interested in exploring |
| `moderator_experience` | Legacy | Confirmed | Prior experience facilitating or moderating groups |
| `notes` | Legacy | Confirmed | Merged from two legacy free-text answers, or an import audit note |
| `contact_consent` | Active | Confirmed | The one current marketing opt-in |
| `ai_experience_level` | **Orphaned** | **Partial** | new, consumer, builder or shipper; defined, but nothing sets it (everyone is `new`) |
| `availability_snippet` | Active (pod leaders) | Confirmed | Short availability note pod leaders see, e.g. "weekends only" |
| `zip` | Active | Confirmed | ZIP code; decides the suggested local lab |
| `metro_slug` | Active | **Partial** | Local lab as a text slug; can drift from `metro_id` |
| `role_intents` | Active | Confirmed | Ways they want to take part: cycle, events, volunteer, mentor |
| `referred_by` | Active | Confirmed | Who referred them, free text |
| `agreement_version` | Active | Confirmed | Which Participant Agreement version they accepted |
| `agreement_accepted_at` | Active | Confirmed | When they accepted it |
| `is_staff` | Active | Confirmed | Staff account; hidden from rosters (a display filter, not a permission) |
| `is_test` | Active | Confirmed | Test account; hidden from rosters and pod health figures |
| `handle` | Active | Confirmed | Profile URL name (`/u/[handle]`) |
| `bio` | Active | Confirmed | Profile "About" text |
| `headline` | Active | Confirmed | Profile tagline |
| `public_profile_visible` | **Orphaned** | Confirmed | Opt-in for a future public profile; not used yet |
| `metro_id` | Active | Confirmed | Local lab (`metros.id`); only set for active labs |
| `years_experience` | **Orphaned** | **Partial** | Allowed values defined; no form asks it |
| `education_level` | **Orphaned** | **Partial** | Allowed values defined; no form asks it |
| `sector_other` | **Orphaned** | **Unknown** | No code, comment or form defines it |
| `created_via` | **Orphaned** | **Partial** | Which route created the row; current sign-ups are left as `unknown` |
| `lab_follow_seeded` | **Orphaned** | Confirmed | Flag for a one-time "follow your lab" step; superseded by `page_follows_seeded` |
| `page_follows_seeded` | Active | Confirmed | Flag: the one-time "follow your own groups" step has run |
| `archived_at` | Active | Confirmed | When an owner archived (deactivated) the person; empty means active |

## The columns in detail

Links point to the source at `ca543cc`.

### Identity and login

#### `id`
- **Type:** `serial`, primary key.
- **Meaning:** the internal participant number. Every table that refers to a person uses it.
- **Evidence:** [`00001_initial_schema.sql`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/supabase/migrations/00001_initial_schema.sql).
- **Confidence:** Confirmed.

#### `auth_user_id`
- **Type:** `uuid`, unique, nullable.
- **Meaning:** the Supabase Auth user ID. Comment in `00001`: "Supabase Auth user ID (linked after OAuth)".
- **Written by:**
  - The sign-up form, from the signed-in session.
  - The sign-in callback, if the row exists but has no link yet ([`app/api/auth/callback/route.ts`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/app/api/auth/callback/route.ts)).
  - The national-to-regional handoff ([`app/api/handoff/route.ts`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/app/api/handoff/route.ts)).
- **Can be empty:** yes. Rows loaded from the legacy import stay empty until the person first signs in.
- **Confidence:** Confirmed.

#### `google_id`
- **Type:** `varchar(255)`, required, unique.
- **What the name suggests:** a Google account ID. It came from the original spec ([`TUL_MVP_Spec.md`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/TUL_MVP_Spec.md)).
- **What's actually stored:** an email address, from every writer.
  - The sign-up form sends `google_id: email` ([`funnel.tsx` L224](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/app/%28auth%29/register/funnel.tsx#L224)).
  - The legacy import fills it with the email too ([`migrate.py` L635–637, L703–707](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/scripts/migration/migrate.py#L635-L637)). It says this is the "prod convention" and that "the auth flow overwrites with the real Google sub on first sign-in".
  - No code does that overwrite. The sign-in callback only sets `auth_user_id`.
- **Read by:** the admin data browser ([`lib/entity-explorer/registry.ts`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/lib/entity-explorer/registry.ts)) and the ban record ([`lib/owner/ban.ts`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/lib/owner/ban.ts)).
- **Confidence:** Partial. In practice it duplicates the email as it was at sign-up. Real Google account IDs are never stored. The login link is `auth_user_id`.

#### `email`
- **Type:** `varchar(255)`, required, unique. Migration `00016` adds a second unique index on `lower(email)`.
- **Meaning:** the person's email. It's the identity key:
  - Sign-in matches on it, ignoring case.
  - The legacy import upserts on it.
  - Bans and tester grants are keyed on it.
- **Written by:** the sign-up form (the Google account email) and the legacy import ("Email Address", lowercased and trimmed).
- **Confidence:** Confirmed.

#### `first_name` and `last_name`
- **Type:** `varchar(255)`, required.
- **Meaning:** first and last name.
- **Written by:**
  - The sign-up form ("Tell us who you are").
  - The legacy import ("First Name" and "Last Name").
  - The profile editor and the admin name editor.
- **Watch for:** the legacy import wrote the literal value `Unknown` when a name was missing ([`migrate.py` L708–715](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/scripts/migration/migrate.py#L708-L715)). The profile editor rejects `Unknown`, and sign-in treats it as a placeholder to replace.
- **Confidence:** Confirmed.

#### `preferred_name`
- **Type:** `varchar(255)`, nullable.
- **Meaning:** the name to use instead of the first name. Profile URLs (`handle`) are built from it when present.
- **Written by:**
  - The legacy import ("Preferred Name").
  - The profile editor.
  - The admin name editor.
  - Pod leaders, through their limited member edit ([`lib/validations/pod-member-update.ts`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/lib/validations/pod-member-update.ts)).
- **Confidence:** Confirmed.

#### `gender`
- **Type:** `varchar(100)`, nullable, no allowed-values list.
- **Meaning:** the free-text answer to the legacy question "Gender".
- **Written by:** the legacy import only. The sign-up form, the profile editor and the admin tools don't ask for or set it.
- **Read by:** nothing in the app. The admin data browser deliberately leaves it out as personal data.
- **Confidence:** Partial. The source is clear, but whether the legacy form was free text or multiple choice isn't in the code. The import copies it as-is, so the values can't be assumed to be consistent.
- **Open question:** what answer options did the legacy form offer?

### Location and library

#### `state`
- **Type:** `varchar(10)`. Allowed values: `MD`, `DC`, `VA`, `Other`. Required until `00016`, nullable since.
- **Meaning:** the answer to the legacy question "Where are you based?".
- **Written by:**
  - The legacy import. Blank or unrecognized answers became `Other`, so `Other` mixes "outside the DMV" with "no answer".
  - The profile editor. Entering a ZIP code fills it in automatically through `stateFromZip`, and the person can still change it.
- **Not asked by:** the current sign-up form.
- **Confidence:** Confirmed.

#### `neighborhood`
- **Type:** `varchar(255)`, nullable since `00016`.
- **Meaning:** the answer to the legacy question "If you are based in the DMV, what neighborhood do you live in? (e.g. Capitol Hill, Pentagon City, Laurel, etc)".
- **Written by:** the legacy import (missing answers became an empty string `''`) and the profile editor.
- **Not asked by:** the current sign-up form.
- **Confidence:** Confirmed.

#### `dcpl_card`
- **Type:** `varchar(20)`. Allowed values: `yes`, `no`, `not sure`.
- **Database comment:** `LEGACY intake (pre-2026-07 flow)` (`00057`).
- **Meaning:** the answer to "Do you currently have a DC Public Library Card?".
- **Written by:** the legacy import (blank answers became `not sure`) and the profile editor, where it's labeled "DC Public Library card".
- **Confidence:** Confirmed. Old `not sure` values mix real "not sure" answers with blanks.

#### `dcpl_info`
- **Type:** `boolean`, nullable.
- **Database comment:** `LEGACY intake (pre-2026-07 flow)`.
- **Meaning:** the answer to "If you do not have a DC Public Library Card or are not sure whether you do, would you like information on how to get one?".
  - Empty means not answered ([`TUL_MVP_Spec.md`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/TUL_MVP_Spec.md)).
  - `true` means yes. `false` means no.
- **Written by:** the legacy import only. Nothing reads it.
- **Confidence:** Confirmed.

### Work and background

#### `work_situation`
- **Type:** `varchar(50)`, nullable since `00016`.
- **Allowed values:**
  - `employed full time`
  - `employed part-time`
  - `self-employed`
  - `unemployed and jobseeking`
  - `in a career transition`
  - `student`
  - `prefer not to say`
- **Meaning:** current work situation. The question has been asked three ways:
  - Legacy form: "What best describes your current work situation?"
  - Sign-up form: "What best describes you right now?"
  - Profile editor: labeled "Work situation".
- **Written by:**
  - The sign-up form. It's required there, and its options match the database list exactly ([`lib/validations/funnel-registration.ts`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/lib/validations/funnel-registration.ts)).
  - The legacy import. Blank answers became `prefer not to say`.
  - The profile editor.
- **Confidence:** Confirmed.

#### `main_focus`
- **Type:** `varchar(50)`.
- **Allowed values:**
  - `finding a new role`
  - `building a portfolio`
  - `upskilling in current field`
  - `exploring new directions`
  - `starting something new`
  - `other`
  - `n/a`
- **Database comment:** `LEGACY intake (pre-2026-07 flow)`.
- **Meaning:** the answer to "If unemployed, underemployed, or in transition: What's your main focus over the next few months?".
- **Written by:** the legacy import (blank answers became `n/a`) and the profile editor ("Main focus").
- **Confidence:** Confirmed. `n/a` mixes "doesn't apply to me" with "no answer".

#### `sector`
- **Type:** `varchar(255)`, free text.
- **Meaning:** the answer to "What sector or field do you primarily work in?". It's labeled "Sector" in the profile.
- **Written by:** the legacy import and the profile editor.
- **Not the same as:** the `sectors` table or `cycles.sector_id`. Those are The Labs' own organizing units. Following a sector page is driven by the cycles someone is enrolled in, not by this column ([`lib/follows/seed.ts`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/lib/follows/seed.ts)).
- **Confidence:** Confirmed.

#### `current_title`
- **Type:** `varchar(255)`, free text.
- **Meaning:** the answer to "Current or most recent job title and organization", so legacy values can include the employer. It's labeled "Current title" in the profile editor.
- **Written by:** the legacy import and the profile editor.
- **Confidence:** Confirmed.

#### `linkedin`
- **Type:** `varchar(500)`.
- **Meaning:** a LinkedIn profile URL, from the legacy question "LinkedIn" and the profile editor.
- **Read by:** the profile page and contact exports ([`lib/export/contacts.ts`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/lib/export/contacts.ts)).
- **Confidence:** Confirmed.

#### `ai_tool_familiarity`
- **Type:** `smallint`, 1 to 5.
- **Meaning:** the answer to "How familiar are you with generative AI tools (like ChatGPT and Claude)?". The scale is recorded in `column_mapping.csv`:

  | Value | Answer |
  |---|---|
  | 1 | Never used |
  | 2 | Hardly use |
  | 3 | Use occasionally |
  | 4 | Use often |
  | 5 | Daily user |

- **Written by:**
  - The legacy import. Missing answers became **1**, so 1 mixes "never used" with "no answer" ([`migrate.py` L732–733](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/scripts/migration/migrate.py#L732-L733)).
  - The profile editor ("How familiar are you with AI tools?").
- **Not the same as:** `ai_experience_level`.
- **Confidence:** Confirmed.

#### `primary_expertise`
- **Type:** `varchar(500)`.
- **Meaning:** the answer to "What is your primary domain or area of expertise?". It's labeled "Primary expertise" in the profile.
- **Written by:** the legacy import and the profile editor.
- **Read by:** the profile and the member directory ([`lib/directory/data.ts`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/lib/directory/data.ts)).
- **Confidence:** Confirmed.

#### `ai_experience_level`
- **Type:** `ai_experience_level`, an enum. Required, default `new`.
- **Allowed values and labels:**

  | Value | Label in the moderator spec | Label in the app |
  |---|---|---|
  | `new` | New to building | New to AI |
  | `consumer` | Consumer of AI tools | AI consumer |
  | `builder` | Has built with AI | AI builder |
  | `shipper` | Ships AI projects | AI shipper |

  The spec labels are in [`docs/PRD-moderator-dashboard.md` §7.3.1](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/docs/PRD-moderator-dashboard.md). The app labels are in [`roster-table.tsx`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/app/%28dashboard%29/moderator/pods/%5Bpod_id%5D/roster-table.tsx).
- **Meaning:** how much someone has built with AI, used in the pod leader's roster, nudges and pulse review. The migration says it's "captured at registration" and that "the registration form is updated separately" ([`00025`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/supabase/migrations/00025_participants_ai_experience.sql)).
- **Written by:** nothing. No form or API sets it, so every row holds the default, `new`.
- **Read by:** the pod leader roster (with a filter and sort), the pulse review panel, and [`lib/moderator/pod-detail.ts`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/lib/moderator/pod-detail.ts).
- **Confidence:** Partial. The intended meaning is documented, but the value has no source. A `new` tells you nothing about the person.
- **Open question:** was the registration question ever built? If not, should this come from `ai_tool_familiarity`?

### Taking part in a cycle (legacy intake)

#### `participation_commitment`
- **Type:** `varchar(20)`. Allowed values: `yes`, `uncertain`.
- **Database comment:** `LEGACY intake (pre-2026-07 flow)`.
- **Meaning:** the answer to "Participation expectations. Please confirm one:".
  - `yes` = "I'm ready to participate consistently and follow through on commitments".
  - `uncertain` = "I'm interested, but my availability is uncertain right now".
- **Written by:** the legacy import only.
- **Read by:** the participant API ([`app/api/participants/[participant_id]/route.ts`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/app/api/participants/%5Bparticipant_id%5D/route.ts)).
- **Confidence:** Confirmed.

#### `commitment_notes`
- **Type:** `text`.
- **Meaning:** the answer to "If your commitment is uncertain, what would make participation workable for you?".
- **Written by:** the legacy import only (added in `00011`). Nothing reads it.
- **Confidence:** Confirmed.

#### `availability_notes`
- **Type:** `text`.
- **Meaning:** the answer to "Anything we should know about your availability?".
- **Written by:** the legacy import only (`00011`). Nothing reads it.
- **Not the same as:** `availability_snippet`.
- **Confidence:** Confirmed.

#### `interest_areas`
- **Type:** `text`.
- **Meaning:** the answer to "What areas are you most interested in exploring?".
- **Written by:** the legacy import only (`00011`). Nothing reads it.
- **Confidence:** Confirmed.

#### `moderator_experience`
- **Type:** `text`.
- **Meaning:** the answer to "If you are interested in moderating: Have you facilitated or moderated groups before?".
- **Written by:** the legacy import only (`00011`). Nothing reads it.
- **Confidence:** Confirmed.

#### `volunteer_interest`
- **Type:** `varchar(500)`.
- **Meaning:** `column_mapping.csv` maps "Would you consider helping steward the community?" to this column.
- **Written by:** nothing, apart from made-up test data in `supabase/seed.sql`. The import's insert statement leaves this column out ([`migrate.py` L638–663](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/scripts/migration/migrate.py#L638-L663)), so the legacy answers never landed here. Nothing reads it.
- **Confidence:** Partial. The intended meaning is clear, but the column is probably empty.
- **Open question:** were the stewardship answers loaded some other way before the import script existed?

#### `notes`
- **Type:** `text`.
- **Meaning:** a holding place for legacy free text. It can contain:
  - Two legacy answers merged with labels ([`migrate.py` L601–614](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/scripts/migration/migrate.py#L601-L614)):
    - `Stewardship interest: …`: the answer to "What interests you about taking on a stewardship or leadership role?"
    - `Other: …`: the answer to "Anything else you'd like us to know?"
  - An audit note, `stub from <sheet>, W1-004`, on placeholder rows. The import created these for people who appeared in other legacy sheets (for example, problem statement submissions) but never registered.
- **Written by:** the legacy import only.
- **Read by:** nothing. The directory and admin data browser deliberately leave it out as personal data.
- **Confidence:** Confirmed.

### Consent and communication

These have changed over time. Migration [`00057`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/supabase/migrations/00057_email_log_and_consent.sql) made `contact_consent` "the single marketing permission". It marked `text_updates`, `email_updates` and `comms_consent` as `LEGACY — superseded by contact_consent (2026-07)`, and kept `photo_video_consent` because it has a different purpose.

The same migration left a one-time step commented out: copying old opt-ins into `contact_consent`, marked "POLICY DECISION, needs legal sign-off". As far as the code shows, it never ran. Legacy opt-ins are **not** reflected in `contact_consent`.

#### `contact_consent`
- **Type:** `boolean`, required, default `false` ("a genuine opt-in"). Added in `00016`.
- **Meaning:** consent to receive updates from The Upskilling Labs.
- **Written by:** the sign-up form's one required checkbox: "I agree to the Participant Agreement — including the Terms of Service, Privacy Policy, and Code of Conduct — and consent to receive updates from The Upskilling Labs."
  - Everyone who signs up through the current form has `true`. They can't join without ticking it.
  - Rows from the legacy import have `false` (see above).
- **Read by:** contact exports.
- **Confidence:** Confirmed. One checkbox covers both agreeing to the Participant Agreement and consenting to updates.

#### `comms_consent`
- **Type:** `boolean`, required, default `true`. Marked LEGACY in the database.
- **Meaning:** "By submitting this form, you consent to receive communications from The Upskilling Labs." Accepting this was required to submit the legacy form, which is why the default is `true` ([`00011`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/supabase/migrations/00011_extend_participants_legacy_fields.sql)).
- **Watch for:** sign-ups through the current form also get `true`, from the default, without being asked.
- **Written by:** the legacy import. Nothing reads it.
- **Confidence:** Confirmed.

#### `email_updates`
- **Type:** `boolean`, nullable. Marked LEGACY in the database.
- **Meaning:** the answer to "Would you like to receive email updates about upcoming Cycles, events, and opportunities?". Empty is a deliberate "no answer", because the question's wording changed partway through a cycle (`00011`).
- **Written by:** the legacy import. Nothing reads it.
- **Confidence:** Confirmed.

#### `text_updates`
- **Type:** `boolean`. Required until `00016`. Marked LEGACY in the database.
- **Meaning:** the answer to "Would you like to receive occasional text updates about in-person events?".
- **Written by:** the legacy import. Missing answers became `false`. Nothing reads it.
- **Confidence:** Confirmed.

#### `photo_video_consent`
- **Type:** `boolean`, required, default `true`.
- **Meaning:** consent to being photographed or filmed at events. The legacy wording was "We record our events through photography and videography in order to share all the great work upskillers like you are doing. Is that okay with you?".
- **Written by:**
  - The legacy import. Missing answers became `false`, "conservative consent posture".
  - The sign-up form, which sets **`true` without asking** ([`route.ts` L177](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/app/api/registrations/funnel/route.ts#L177)). The agreement text says "Sessions may be photographed — you can opt out of photography consent when registering for the event."
- **Read by:** contact exports.
- **Confidence:** Partial. For current sign-ups, `true` isn't an answer the person gave.
- **Open question:** is consent now meant to be per event? If so, where is that recorded?

#### `phone_number`
- **Type:** `varchar(50)`.
- **Meaning:** the answer to "Phone number (Used for day-of updates about in-person events.)".
- **Written by:** the legacy import and pod leaders (limited member edit).
- **Not asked by:** the sign-up form or the profile editor.
- **Read by:** contact exports and pod leader tools.
- **Confidence:** Confirmed.

### How they found The Labs

#### `source`
- **Type:** `varchar(255)`, free text in the database.
- **Meaning:** how the person heard about The Labs. Three different vocabularies are mixed in this column:

  | Who wrote it | Values |
  |---|---|
  | Sign-up form, "How did you hear about The Labs?" | `referral` (a friend or colleague referred me), `invited` (someone at The Labs invited me), `event` (a workshop, summit, or event), `other` (somewhere else) |
  | Legacy import, "How did you hear about The Upskilling Labs?" | Whatever the person typed, as-is |
  | Legacy import, rows with no answer and placeholder rows | `legacy_migration` |

  The `source` values in the ambassador code (`ambassador_flow`, `invite`) are written to other tables (`agreement_acceptances` and the ambassador tables), not to `participants`.
- **Confidence:** Partial. The meaning is clear, but the values aren't comparable across eras, and `legacy_migration` means "unknown", not a channel.

#### `referred_by`
- **Type:** `varchar(255)`, free text.
- **Meaning:** the follow-up question "Who referred you? (optional — so we can thank them)". The form shows it when `source` is `referral`, `invited` or `other`.
- **Written by:** the sign-up form. Added in [`00031`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/supabase/migrations/00031_funnel_registration_fields.sql).
- **Confidence:** Confirmed.

#### `created_via`
- **Type:** `varchar`, required, default `unknown`.
- **Allowed values:** `onboarding_app`, `invitation`, `admin`, `import`, `unknown`.
- **Meaning:** "which door they came through" ([`00056`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/supabase/migrations/00056_intake_and_interest.sql)). That migration set every row that existed at the time to `import`.
- **Written by:** nothing. The sign-up form doesn't set it, so everyone who signed up after `00056` has `unknown`, not `onboarding_app`.
- **Read by:** nothing.
- **Confidence:** Partial. `import` reliably means "existed before `00056`". `unknown` actually means "created afterwards", almost certainly through the sign-up form, since that is the only code that creates rows.

#### `role_intents`
- **Type:** `text[]`, required, default empty. Values must come from `cycle`, `events`, `volunteer`, `mentor`.
- **Meaning:** the ways the person wants to take part, picked at sign-up (at least one is required):

  | Value | Option shown |
  |---|---|
  | `cycle` | Build Cycles: "Join a pod, take on a real problem, and ship something you're proud of." |
  | `events` | Attend events & workshops |
  | `volunteer` | Volunteer: "Help run events, or help behind the scenes." |
  | `mentor` | Mentor: "Share what you know — office hours, workshops, and the occasional rescue." |

- **Written by:** the sign-up form and the profile editor.
- **Read by:** the profile and the ambassador application. A 2026-09 audit notes the values "drive no flow" yet ([`docs/roadmap/2026-09-audit.md` §4.7](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/docs/roadmap/2026-09-audit.md)).
- **Confidence:** Confirmed.

### Participant Agreement

#### `agreement_version`
- **Type:** `varchar(50)`.
- **Meaning:** the version of the Participant Agreement accepted at sign-up. The current version is `participant-2026-07-v2`; acceptances of v1 "remain valid; no backfill" ([`funnel-registration.ts`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/lib/validations/funnel-registration.ts)).
- **Written by:** the sign-up form.
- **Not stored here:** the Open Cycle Agreement, which goes in `cycle_agreements`, and the Ambassador agreement, which goes in `agreement_acceptances` and `ambassador_applications`.
- **Can be empty:** yes, for legacy rows.
- **Confidence:** Confirmed.

#### `agreement_accepted_at`
- **Type:** `timestamptz`.
- **Meaning:** when that agreement was accepted. The sign-up form sets it to the moment the row is created.
- **Confidence:** Confirmed.

### Local lab

#### `zip`
- **Type:** `varchar(10)`.
- **Meaning:** ZIP code, required at sign-up as 5 digits or ZIP+4. Its first 3 digits are used to suggest the nearest local lab.
- **Written by:** the sign-up form and the profile editor. The profile editor's help text says it "Sets your local lab and fills in your state."
- **Confidence:** Confirmed.

#### `metro_id`
- **Type:** `integer`, a reference to `metros(id)`. Added in [`00044`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/supabase/migrations/00044_directory_columns.sql).
- **Meaning:** the person's local lab. The sign-up form sets it **only when the person joined an active lab**. People on a waitlist and people with no lab stay empty, and a waitlist place is recorded in `metro_waitlist_signups` instead ([`docs/LOCAL_LABS.md`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/docs/LOCAL_LABS.md)).
- **Why it matters:** it gates registering for a cycle and which pods the person can join.
- **Written by:** the sign-up form, and joining a lab later ([`lib/labs/membership.ts`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/lib/labs/membership.ts)).
- **Confidence:** Confirmed.

#### `metro_slug`
- **Type:** `varchar(50)`. References `metros(slug)` since `00038`; that rule isn't checked against older rows.
- **Meaning:** the local lab as a text slug. It came before `metro_id`; `00044` describes `metro_id` as normalizing it.
- **Written by:**
  - The sign-up form and joining a lab, together with `metro_id`.
  - The profile editor, when the ZIP changes: it recalculates `metro_slug` from the ZIP **but leaves `metro_id` as it was** ([`app/api/participants/[participant_id]/route.ts` L97–101](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/app/api/participants/%5Bparticipant_id%5D/route.ts#L97-L101)). So the two can disagree.
- **Read by:** contact exports.
- **Confidence:** Partial. Treat `metro_id` as the authoritative lab.

### Profile and directory

#### `handle`
- **Type:** `varchar(50)`. Unique ignoring case; lowercase letters, numbers and dashes.
- **Database comment:** "URL-safe vanity handle for /u/[handle]; auto-generated (trigger) + member-editable."
- **How it's set:**
  - Built automatically when the row is created, from preferred or first name plus last name.
  - If that's taken, the database adds the row number.
  - Migration `00083` later removed number suffixes where the plain handle was free.
- **Editable in:** the profile editor ("Profile URL").
- **Confidence:** Confirmed.

#### `headline`
- **Type:** `varchar(200)`.
- **Meaning:** the profile tagline, labeled "Headline". It's shown in the directory, on profiles and in follow suggestions.
- **Confidence:** Confirmed.

#### `bio`
- **Type:** `text`, limited to 2,000 characters by the profile editor.
- **Meaning:** labeled "About": "A short intro shown on your profile."
- **Confidence:** Confirmed.

#### `profile_image_url`
- **Type:** `text`.
- **Meaning:** the avatar image URL.
  - The sign-up form fills it with the Google account photo.
  - The avatar upload replaces or clears it ([`app/api/participants/[participant_id]/avatar/route.ts`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/app/api/participants/%5Bparticipant_id%5D/avatar/route.ts)).
- **Watch for:** the `00045` database comment ("Null for funnel signups today") is out of date. The sign-up form now does fill it.
- **Confidence:** Confirmed.

#### `public_profile_visible`
- **Type:** `boolean`, required, default `false`.
- **Database comment:** "Opt-in flag for a future PUBLIC (non-authed) profile tier. Default false; the members-only directory ignores it."
- **Written or read by:** nothing yet.
- **Confidence:** Confirmed. It's a placeholder for a feature that isn't built.

#### `availability_snippet`
- **Type:** `text`. Pod leaders' edits are limited to 200 characters.
- **Meaning:** a short, free-form availability note for pod leaders, e.g. "weekends only" ([PRD §7.3.1](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/docs/PRD-moderator-dashboard.md)).
- **Written by:** pod leaders only, through the member edit ([`app/api/pods/[pod_id]/members/[participant_id]/route.ts`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/app/api/pods/%5Bpod_id%5D/members/%5Bparticipant_id%5D/route.ts)). The PRD says it's "captured at registration", but no registration form asks for it.
- **Read by:** the pod leader roster (searchable) and the pulse review panel.
- **Confidence:** Confirmed.

### Fields added for a redesign but never asked

[`00056`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/supabase/migrations/00056_intake_and_interest.sql) added these as "New intake columns (all nullable — events-only signups skip every one of them)". It says they were generated from `docs/DB_CHANGES_ONBOARDING.md`, which isn't in the repo. No current form asks for them and no code reads them. An audit lists them as "Professional context for 'find your people,' unsurfaced" ([`docs/audit/SOCIAL_LAYER_ANALYSIS.md`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/docs/audit/SOCIAL_LAYER_ANALYSIS.md)).

#### `years_experience`
- **Type:** `varchar`.
- **Allowed values:** `just starting out`, `1-4`, `5-9`, `10-19`, `20+`, `prefer not to answer`.
- **Meaning:** years of experience, but the code never says experience in what (work in general, the field in `sector`, or AI).
- **Confidence:** Partial. Expect it to be empty.

#### `education_level`
- **Type:** `varchar`.
- **Allowed values:**
  - `high school or GED`
  - `some college`
  - `associate degree`
  - `bachelors degree`
  - `graduate degree`
  - `trade or technical certification`
  - `prefer not to answer`
- **Meaning:** highest education. The values make that much clear, but the question wording was never built.
- **Confidence:** Partial. Expect it to be empty.

#### `sector_other`
- **Type:** `varchar`.
- **Meaning:** not defined anywhere. The name suggests the free text for an "Other" choice on a fixed sector list. But `participants.sector` is already free text, and no fixed list of sector choices for members exists in the code.
- **Confidence:** **Unknown.**

### Admin and system flags

#### `is_staff`
- **Type:** `boolean`, required, default `false`.
- **Meaning:** a staff account. In [`00041`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/supabase/migrations/00041_participants_staff_test_flags.sql)'s words, staff and test accounts are "hidden from participant-facing rosters by default"; the flags "are display filters, never permissions".
- **Written by:** the admin staff toggle ([`app/api/admin/staff-flag/route.ts`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/app/api/admin/staff-flag/route.ts)).
- **Confidence:** Confirmed.

#### `is_test`
- **Type:** `boolean`, required, default `false`.
- **Meaning:** a test account. It's hidden like staff accounts and also left out of pod health figures.
- **Written by:**
  - Admin tester controls ([`app/api/admin/testers/route.ts`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/app/api/admin/testers/route.ts)).
  - The sign-up form, automatically, when the email is in `testers` ([`00042`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/supabase/migrations/00042_testers.sql)).
- **Confidence:** Confirmed.

#### `archived_at`
- **Type:** `timestamptz`.
- **Meaning:** when an owner archived (deactivated) the person. Empty means active ([`00079`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/supabase/migrations/00079_participant_lifecycle.sql)).
- **What archiving does:** it also revokes their roles, enrollments and memberships ([`lib/owner/archive.ts`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/lib/owner/archive.ts)).
- **Undoing it:** clear the field. Lifting a ban also clears it.
- **Not a ban:** archived people can still sign in, with no roles. Bans live in `participant_bans`.
- **Confidence:** Confirmed.

#### `created_at`
- **Type:** `timestamp` (without time zone), default now.
- **Meaning:** when the row was created.
  - For legacy rows, it's the import date, not the original form submission date. The import doesn't set it.
  - It's also the starting point for pulse-check deadlines when no pulse check has been completed.
- **Confidence:** Confirmed.

#### `last_pulse_completed_at`
- **Type:** `timestamp`.
- **Meaning:** when the person last completed a weekly pulse check ([`00010`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/supabase/migrations/00010_pulse_check_v2.sql), "7-day enforcement tracking").
- **Written by:** the pulse check submission.
- **Read by:** the enforcement check, which counts 7 days from this value (or from `created_at`) and flags warnings at 3 days and 1 day before the deadline, then overdue ([`app/api/pulse-checks/enforcement/route.ts`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/app/api/pulse-checks/enforcement/route.ts)).
- **Confidence:** Confirmed.

#### `page_follows_seeded`
- **Type:** `boolean`, required, default `false`.
- **Meaning:** a marker that the one-time "auto-follow your own groups" step has run ([`00076`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/supabase/migrations/00076_page_authored_updates.sql)). That step follows the person's lab, active pods, workstreams, projects and sector. The marker means it never re-runs, so a later unfollow sticks.
- **Written and read by:** the dashboard on load ([`lib/follows/seed.ts`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/lib/follows/seed.ts)).
- **Confidence:** Confirmed.

#### `lab_follow_seeded`
- **Type:** `boolean`, required, default `false`.
- **Meaning:** the same kind of marker, for lab pages only ([`00075`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/supabase/migrations/00075_participant_lab_follow_seed.sql)).
- **Written or read by:** nothing now. The lab is covered by `page_follows_seeded`.
- **Confidence:** Confirmed. Orphaned.

### Platform accounts (superseded)

The original spec called these "Platform accounts (populated post-onboarding)". The regional fork now keeps them in a separate table, `connected_accounts`, with `github_username`, `slack_name` and `files_email` ([`90001`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/supabase/migrations/90001_region_handoff_and_connected_accounts.sql), edited at `/profile/accounts`). That table's description: "the accounts a member wants The Labs to invite to things".

#### `slack_username`
- **Type:** `varchar(255)`.
- **Meaning:** the person's Slack name.
- **Written by:** nothing.
- **Read by:** contact exports.
- **Confidence:** Partial. The current value lives in `connected_accounts.slack_name`.

#### `github_username`
- **Type:** `varchar(255)`.
- **Meaning:** the person's GitHub username.
- **Written by:** nothing on `participants`.
- **Read by:** contact exports.
- **Confidence:** Partial. The current value lives in `connected_accounts.github_username`.

#### `drive_email`
- **Type:** `varchar(255)`.
- **Meaning:** the Google account for shared files.
- **Written or read by:** nothing.
- **Confidence:** Partial. The current value lives in `connected_accounts.files_email`.

## Things to watch for when scoping

1. **Default values that look like answers.** The legacy import filled required fields with placeholders. Don't read these as real answers:
   - `state = 'Other'`
   - `neighborhood = ''`
   - `dcpl_card = 'not sure'`
   - `work_situation = 'prefer not to say'`
   - `main_focus = 'n/a'`
   - `ai_tool_familiarity = 1`
   - `text_updates = false` and `photo_video_consent = false`
   - names of `Unknown`
   - `source = 'legacy_migration'`

   Current sign-ups get some too:
   - `ai_experience_level = 'new'`
   - `created_via = 'unknown'`
   - `comms_consent = true`
   - `photo_video_consent = true`
2. **Three consent fields are retired.** Only `contact_consent` is current. Legacy opt-ins were never copied into it.
3. **Two lab fields can disagree.** `metro_id` is authoritative; `metro_slug` can drift from it.
4. **Columns nothing writes:**
   - `volunteer_interest`
   - `slack_username`, `github_username`, `drive_email`
   - `ai_experience_level`
   - `years_experience`, `education_level`, `sector_other`
   - `created_via`
   - `public_profile_visible`
   - `lab_follow_seeded`
5. **`google_id` doesn't hold Google account IDs.** It holds an email address.

## Questions for the OLOS developers

These are only for what the code can't answer:

1. **`sector_other`:** what was it meant to hold? Was a fixed sector list planned for members? `docs/DB_CHANGES_ONBOARDING.md`, which the migration says it was generated from, isn't in the repo. Where is it?
2. **`years_experience` and `education_level`:** what was the question wording, and was the "experience" in years of experience meant to be work, field or AI?
3. **`ai_experience_level`:** was the registration question ever built? Should it be filled from `ai_tool_familiarity`, or dropped?
4. **`volunteer_interest`:** were legacy stewardship answers loaded anywhere other than `notes`?
5. **`gender`:** what answer options did the legacy form offer?
6. **`google_id`:** is it still needed, given that `auth_user_id` is the login link and every value is an email?
7. **`photo_video_consent`:** new sign-ups get `true` without being asked. Is consent now collected per event, and where?
8. **`created_via`:** should the sign-up form set `onboarding_app`? Rows from after `00056` show `unknown`.
9. **`metro_slug`:** should the profile editor update `metro_id` as well, or should `metro_slug` be retired?

## Query to check real data

Someone with access to the regional Supabase database can run this to see how many rows hold each value. It only reads data and returns counts, with no names or emails. Free-text columns are counted as filled or empty.

```sql
-- Coded columns: count each distinct value.
SELECT 'state' AS col, state::text AS value, count(*) FROM participants GROUP BY 2
UNION ALL SELECT 'dcpl_card', dcpl_card, count(*) FROM participants GROUP BY 2
UNION ALL SELECT 'dcpl_info', dcpl_info::text, count(*) FROM participants GROUP BY 2
UNION ALL SELECT 'work_situation', work_situation, count(*) FROM participants GROUP BY 2
UNION ALL SELECT 'main_focus', main_focus, count(*) FROM participants GROUP BY 2
UNION ALL SELECT 'ai_tool_familiarity', ai_tool_familiarity::text, count(*) FROM participants GROUP BY 2
UNION ALL SELECT 'ai_experience_level', ai_experience_level::text, count(*) FROM participants GROUP BY 2
UNION ALL SELECT 'participation_commitment', participation_commitment, count(*) FROM participants GROUP BY 2
UNION ALL SELECT 'source', source, count(*) FROM participants GROUP BY 2
UNION ALL SELECT 'created_via', created_via, count(*) FROM participants GROUP BY 2
UNION ALL SELECT 'years_experience', years_experience, count(*) FROM participants GROUP BY 2
UNION ALL SELECT 'education_level', education_level, count(*) FROM participants GROUP BY 2
UNION ALL SELECT 'agreement_version', agreement_version, count(*) FROM participants GROUP BY 2
UNION ALL SELECT 'role_intents', array_to_string(role_intents, ','), count(*) FROM participants GROUP BY 2
UNION ALL SELECT 'text_updates', text_updates::text, count(*) FROM participants GROUP BY 2
UNION ALL SELECT 'email_updates', email_updates::text, count(*) FROM participants GROUP BY 2
UNION ALL SELECT 'comms_consent', comms_consent::text, count(*) FROM participants GROUP BY 2
UNION ALL SELECT 'contact_consent', contact_consent::text, count(*) FROM participants GROUP BY 2
UNION ALL SELECT 'photo_video_consent', photo_video_consent::text, count(*) FROM participants GROUP BY 2
UNION ALL SELECT 'is_staff', is_staff::text, count(*) FROM participants GROUP BY 2
UNION ALL SELECT 'is_test', is_test::text, count(*) FROM participants GROUP BY 2
UNION ALL SELECT 'public_profile_visible', public_profile_visible::text, count(*) FROM participants GROUP BY 2
UNION ALL SELECT 'lab_follow_seeded', lab_follow_seeded::text, count(*) FROM participants GROUP BY 2
ORDER BY 1, 3 DESC;

-- Free-text and identifier columns: filled vs empty, plus two consistency checks.
SELECT
  count(*)                                                   AS total_rows,
  count(*) FILTER (WHERE google_id = email)                  AS google_id_equals_email,
  count(auth_user_id)                                        AS has_auth_user_id,
  count(NULLIF(gender, ''))                                  AS gender,
  count(NULLIF(neighborhood, ''))                            AS neighborhood,
  count(NULLIF(sector, ''))                                  AS sector,
  count(NULLIF(sector_other, ''))                            AS sector_other,
  count(NULLIF(current_title, ''))                           AS current_title,
  count(NULLIF(primary_expertise, ''))                       AS primary_expertise,
  count(NULLIF(volunteer_interest, ''))                      AS volunteer_interest,
  count(NULLIF(availability_notes, ''))                      AS availability_notes,
  count(NULLIF(commitment_notes, ''))                        AS commitment_notes,
  count(NULLIF(interest_areas, ''))                          AS interest_areas,
  count(NULLIF(moderator_experience, ''))                    AS moderator_experience,
  count(NULLIF(notes, ''))                                   AS notes,
  count(NULLIF(availability_snippet, ''))                    AS availability_snippet,
  count(NULLIF(phone_number, ''))                            AS phone_number,
  count(NULLIF(slack_username, ''))                          AS slack_username,
  count(NULLIF(github_username, ''))                         AS github_username,
  count(NULLIF(drive_email, ''))                             AS drive_email,
  count(NULLIF(referred_by, ''))                             AS referred_by,
  count(NULLIF(zip, ''))                                     AS zip,
  count(metro_id)                                            AS metro_id,
  count(*) FILTER (WHERE metro_slug IS DISTINCT FROM
        (SELECT m.slug FROM metros m WHERE m.id = participants.metro_id)) AS metro_slug_mismatch,
  count(archived_at)                                         AS archived
FROM participants;
```
