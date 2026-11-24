# OLOS table and column usage

Which of the OLOS database's 80 tables and 795 columns the code actually uses, and how. This isn't a data dictionary. It only records where each table and column is written and read.

- **Source:** `wash-TheUpskillingLabs/c04-olos` at commit [`ca543cc`](https://github.com/wash-TheUpskillingLabs/c04-olos/tree/ca543cc01fb3d7e6ef17d2281832e766b56ec0c3) (2026-11-17). No migration has changed since the ERDs were built (`63dc2df`).
- **Companion file:** `olos-column-usage.csv`, one row per column (795 rows), listing the files and database functions that write or read it.
- **Built:** 2026-11-19. **Corrected:** 2026-11-24 (`change_participant_email()` is called from outside this repo; see Database functions).

## How this was worked out

1. **The schema.** All 113 migrations were run, in order, into an empty Postgres database. That gives the exact tables, columns, defaults, keys, database functions, triggers and row-level security rules as they stand today. There are no views.
2. **The app code.** Every non-test file under `app/` and `lib/` was scanned for database calls: `.from("table")`, then what it does. That covers:
   - Which columns a `select(...)` asks for, including tables joined inside it and column lists kept in constants.
   - Which fields an `insert`, `update` or `upsert` sends, including payloads checked by a validation schema.
   - Which columns are used in filters and sorting.
   - Calls to database functions (`rpc`).
3. **The database itself.** Each function's body was checked for the tables and columns it inserts, updates, deletes or reads. A function only counts if something reaches it: a trigger, a security rule, the app, or another function that is reached.
4. **Kept separate:**
   - The legacy import and ops scripts (`scripts/`) and test data (`supabase/seed.sql`). They don't make a table count as used, but the CSV notes them.
   - The two generic admin tools, the admin data browser and the owner console, which can open any table they list.
5. **Checked by hand.** Every orphan, and samples from each other group, were confirmed by reading the code. The fixes that came out of those checks are built into the results.

**Limits.** This is reading the code, not the data:
- A table can be wired up and still be empty.
- People can enter data directly in the Supabase dashboard or by SQL, and the code can't show that.
- Code outside this repo can also write to the database. At least one such writer exists: an "onboarding admin edge function" with full (service-role) access, named in migration `00085`. Its code isn't in this repo, so what it reads and writes isn't counted here.
- Where code builds a query from variables, a column's use is inferred from its name. The CSV marks these as "Indirect" or "via select *", so you can see where the evidence is weaker.

## Summary

### Tables

| Status | Tables | What it means |
|---|---|---|
| Active | 70 | The app both writes and reads it. |
| Read, never written | 3 | The app reads it, but nothing in the code writes to it. It's filled by migrations or by hand. |
| Written, never read | 2 | The app writes it, but nothing reads it back. |
| Written by the database, never read | 4 | Only a database function or trigger writes it, and nothing reads it. |
| Orphaned | 1 | Nothing creates or reads its rows. Only cleanup functions delete from it. |

### Columns

| Status | Columns | What it means |
|---|---|---|
| Written and read | 481 | Code writes it and code reads it. |
| Read; filled automatically | 102 | Read by code; set by a default (row number, timestamp) or a trigger. |
| Read, never written | 47 | Read by code, but no code writes it. The value comes from migrations, SQL by hand, or the Supabase dashboard. |
| Written, never read | 70 | Code writes it, but nothing reads it back. Often audit fields, which may be on purpose. |
| Indirect (name match only) | 32 | No query names it. Its name does appear in a file that uses the table, probably through a dynamic path. Weaker evidence; verify before acting. |
| Loaded by select * only | 28 | Loaded as part of a whole row (`select("*")`), but no code uses the field. In practice, unused. |
| Filled automatically, never read | 27 | A default sets it (row number, timestamp) and nothing reads it. |
| Orphaned | 8 | Nothing in the app or the database writes or reads it. |

**Most likely unused:** 8 orphaned columns, 28 loaded only by `select *`, and 27 filled automatically but never read. Six tables and 70 columns are written but never read. Another 32 are indirect and need a closer look.

## Tables that aren't plainly active

Ten tables. Six are audit or bookkeeping records that are written on purpose and never shown. One is genuinely orphaned. Two are entered by hand.

### Orphaned

- **`project_subscriptions`** (created in `00060`). `SCHEMA.md` describes it as how individual contributors "join a project via `project_subscriptions` (self-serve follow)". No API route, page or database function creates or reads a row. The only code that touches it is the owner reset functions (`reset_cycle`, `reset_pod`, `reset_project`), which delete from it. The feature was never built.

### Written, never read: audit and bookkeeping records

These are written so there's a record, but nothing in the app shows that record. If someone needs it, they have to query the database directly.

| Table | Written by | Purpose |
|---|---|---|
| `owner_actions` | the owner console and the owner reset functions | Audit trail of owner archive, reset and delete actions (`00078`) |
| `admin_actions` | `admin_add_to_project()` and `admin_move_project()` | Audit log for admin project roster edits (`00103` comment) |
| `participant_erasures` | `delete_participant()` | Record left behind when a person is deleted (`00058`) |
| `handoff_redemptions` | the national-to-regional handoff (`app/api/handoff`) | Single-use guard: the insert fails if a pass was already used, so it never needs reading (`90001`) |
| `auto_follows` | the regional auto-follow trigger | Bookkeeping for the cycle auto-follow, so an unfollow sticks (`90002`) |
| `region_search_demand` | `count_region_search()` | Anonymous daily counts of map searches by area (`90006`). Nothing in the app shows the counts. |

### Read, never written: entered by hand or by migration

| Table | Read by | Where its rows come from |
|---|---|---|
| `sectors` | org pages, the org-cycle code | Seeded by migration `00060` |
| `cycle_events` | the pod page | Seeded by migrations `00086` and `00092`, then changed by ops SQL in `scripts/ops/`. Its `luma_api_id` is never used by the app. |
| `event_transit` | `lib/region/transit.ts` (event cards) | Nothing. The migration says "for now by SQL; an admin form later" (`90005`). |

## Orphaned columns

Each was checked by searching the whole repo for the name. These eight have no reader or writer in the app or the database.

| Column | What was found |
|---|---|
| `project_subscriptions.participant_id`, `.project_id` | The table itself is orphaned (see above). |
| `pod_memberships.preference_rank` | Written only by the legacy import (`scripts/migration/migrate.py`, "Seat 1/2/3" pod preferences). The app never sets or reads it. |
| `cycle_events.luma_api_id` | Only read by an ops SQL script. Meant for a Luma sync that doesn't use it. |
| `cycle_phases.anchor` | No code uses it. |
| `leadership_logs.schema_version` | No code uses it on this table. The name `schema_version` is used for pulse checks and learning logs. |
| `survey_responses.source_url`, `.schema_version` | No code uses either. |

## Loaded but never used (`select *` only)

These columns come along whenever their table's whole row is loaded, but no code ever uses the field. Most are on `participants`, and they line up with the orphaned and legacy fields in the participants data dictionary.

- **`cycle_config`**: `updated_at`, `pulse_agg_default_weeks`
- **`invitations`**: `notes`
- **`metros`**: `updated_at`, `timezone`
- **`page_profiles`**: `updated_at`
- **`participants`**: `gender`, `dcpl_info`, `volunteer_interest`, `text_updates`, `drive_email`, `email_updates`, `comms_consent`, `availability_notes`, `commitment_notes`, `interest_areas`, `moderator_experience`, `public_profile_visible`, `years_experience`, `education_level`, `sector_other`, `created_via`, `lab_follow_seeded`
- **`pods`**: `google_group_email`
- **`projects`**: `slack_channel_id`, `drive_folder_id`, `google_group_email`
- **`workstreams`**: `updated_at`

## Written, never read

Code saves a value here that nothing reads back. Many are "who did it" fields on audit rows, such as `revoked_by`, `granted_by` and `actor_email`, which are normal to keep. The ones worth a look are where members are asked for answers that then go nowhere:

- **`baseline_responses`.** All eight survey answers are saved and never read anywhere in the app: `ai_usage_frequency`, `work_shift_outlook`, `role_change_outlook`, `skills_readiness`, `learning_confidence`, `judgment_confidence`, `autonomy`, `peer_investment`. They may be meant for analysis outside the app.
- **`cycle_agreements.answers`.** The answers given when signing the Open Cycle Agreement are saved but never shown.
- **`email_log`.** Written and read only by the learning-log compliance nudge. That job isn't scheduled in `vercel.json`, sends nothing unless switched on, and dry-runs by default.
- **`projects.forked_from_project_id`.** `SCHEMA.md` calls it a "fork-lineage pointer (no endpoint/UI yet)".

The full list:

- **`admin_actions`**: `actor_participant_id`, `actor_email`, `action`, `participant_id`, `participant_label`, `cycle_id`, `from_project_id`, `to_project_id`, `from_pod_id`, `to_pod_id`, `cap_overridden`, `reason`
- **`agreement_acceptances`**: `source`
- **`ambassador_nominations`**: `nominator_id`, `decided_by`, `decided_at`
- **`announcements`**: `author_participant_id`
- **`auto_follows`**: `participant_id`, `page_type`, `page_id`
- **`baseline_responses`**: `learning_log_id`, `ai_usage_frequency`, `work_shift_outlook`, `role_change_outlook`, `skills_readiness`, `learning_confidence`, `judgment_confidence`, `autonomy`, `peer_investment`
- **`cycle_agreements`**: `answers`
- **`cycle_phases`**: `kind`, `position`
- **`email_log`**: `to_email`, `subject`, `payload`
- **`event_formats`**: `updated_at`
- **`events`**: `updated_at`
- **`feedback`**: `auth_user_id`
- **`feedback_attachments`**: `size_bytes`
- **`follows`**: `id`
- **`handoff_redemptions`**: `jti`, `expires_at`
- **`owner_actions`**: `actor_participant_id`, `actor_email`, `entity_type`, `entity_id`, `entity_label`, `action`, `reason`, `detail`
- **`page_admins`**: `added_by_participant_id`
- **`page_profiles`**: `updated_by`
- **`participant_bans`**: `google_id`, `participant_id`, `banned_by`, `revoked_by`
- **`participant_erasures`**: `erased_by`, `reason`
- **`participant_roles`**: `revoked_by`
- **`profile_updates`**: `learning_log_id`, `posted_by_participant_id`
- **`projects`**: `forked_from_project_id`
- **`region_search_demand`**: `day`, `kind`, `area`, `searches`
- **`simulation_sessions`**: `actor_participant_id`
- **`survey_responses`**: `consent_participation`
- **`testers`**: `granted_by`
- **`update_media`**: `storage_path`

## Read, never written

The app shows or uses these, but no code sets them. They're set by migrations, SQL by hand, or the Supabase dashboard. Some are on purpose, such as content fields edited outside the app. Others are leftovers:

- **`participants.slack_username`, `.github_username`.** Still included in contact exports, but nothing writes them. The current values live in `connected_accounts`, so exports show these as blank.
- **`pods.slack_channel_id`, `.github_repo_url`, `.drive_folder_id`.** Read by the app, but there's no form to set them.

The full list:

- **`ambassador_applications`**: `invite_id`
- **`announcements`**: `published_at`
- **`custom_tasks`**: `archived_at`
- **`cycle_config`**: `ai_summary_prompt`
- **`cycle_events`**: `cycle_id`, `label`, `occurs_at`
- **`cycles`**: `description`, `what_you_build`, `start_at`
- **`event_transit`**: `station`, `walk_minutes`, `note`
- **`events`**: `host`, `gallery`, `sponsors`, `stats`
- **`follows`**: `followee_participant_id`
- **`leadership_logs`**: `pod_id`
- **`metros`**: `partner`, `members`, `blurb`, `archived_at`
- **`nominations`**: `participant_id`
- **`page_profiles`**: `cover_url`, `mark_url`
- **`participants`**: `participation_commitment`, `slack_username`, `github_username`, `notes`
- **`pods`**: `slack_channel_id`, `github_repo_url`, `drive_folder_id`
- **`projects`**: `sector_id`
- **`resources`**: `img`, `grad`, `tags`, `from_line`, `body`
- **`sectors`**: `name`, `slug`, `description`
- **`solution_proposals`**: `proposal_text`
- **`spotlights`**: `slug`, `published_at`, `image_url`
- **`user_roles`**: `revoked_at`

## Indirect: verify before acting

No query names these columns, but the name appears in a file that works with the table. Most are `id` and `created_at` fields, which are almost certainly in use. The rest are worth a check:

- **`access_revocations`**: `id`
- **`agreement_acceptances`**: `id`
- **`baseline_responses`**: `id`, `schema_version`, `created_at`
- **`cycle_config`**: `id`
- **`cycle_enrollments`**: `tier`
- **`cycle_events`**: `id`, `key`
- **`cycle_phases`**: `id`, `duration`
- **`email_log`**: `id`
- **`feedback_attachments`**: `created_at`
- **`field_surveys`**: `created_at`
- **`follows`**: `created_at`
- **`metro_waitlist_signups`**: `created_at`
- **`metros`**: `waiting_baseline`
- **`owner_actions`**: `id`
- **`participant_permissions`**: `id`
- **`profile_update_likes`**: `id`, `created_at`
- **`project_votes`**: `created_at`
- **`pulse_checks`**: `created_at`
- **`resources`**: `updated_at`
- **`simulation_sessions`**: `started_at`
- **`survey_response_answers`**: `id`, `created_at`
- **`testers`**: `created_at`
- **`update_media`**: `id`
- **`update_reposts`**: `id`
- **`user_roles`**: `id`
- **`weekly_messages`**: `id`

## Code that writes data but that nothing calls

These API routes exist, but nothing in the app calls them. Any data written only through them isn't reachable from the UI. Some are called from outside the app, and the list says which.

| Route | Notes |
|---|---|
| [`app/api/cron/learning-log-compliance-nudge/route.ts`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/app/api/cron/learning-log-compliance-nudge/route.ts) | A job not scheduled in `vercel.json`; run by hand, dry-run by default. |
| [`app/api/cron/revocation-check/route.ts`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/app/api/cron/revocation-check/route.ts) | A job not scheduled in `vercel.json`. |
| [`app/api/cycles/[cycle_id]/interest/route.ts`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/app/api/cycles/%5Bcycle_id%5D/interest/route.ts) | Writes `participants.state`, `work_situation`, `main_focus`, `sector`, `linkedin` and participant options. No UI calls it. |
| [`app/api/cycles/[cycle_id]/my-solution-proposal/route.ts`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/app/api/cycles/%5Bcycle_id%5D/my-solution-proposal/route.ts) | No caller found in app code. |
| [`app/api/dashboard/[cycle_id]/pods/[pod_id]/route.ts`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/app/api/dashboard/%5Bcycle_id%5D/pods/%5Bpod_id%5D/route.ts) | No caller found in app code. |
| [`app/api/dashboard/[cycle_id]/route.ts`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/app/api/dashboard/%5Bcycle_id%5D/route.ts) | No caller found in app code. |
| [`app/api/handoff/route.ts`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/app/api/handoff/route.ts) | Called from national's site (the handoff from the national dashboard). In use. |
| [`app/api/nominations/route.ts`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/app/api/nominations/route.ts) | No caller found in app code. |
| [`app/api/options/route.ts`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/app/api/options/route.ts) | No caller found in app code. |
| [`app/api/pods/[pod_id]/members/[participant_id]/route.ts`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/app/api/pods/%5Bpod_id%5D/members/%5Bparticipant_id%5D/route.ts) | The pod leader's member edit (`preferred_name`, `phone_number`, `availability_snippet`). No UI calls it. |
| [`app/api/pods/[pod_id]/members/route.ts`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/app/api/pods/%5Bpod_id%5D/members/route.ts) | No caller found in app code. |
| [`app/api/pods/[pod_id]/name/route.ts`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/app/api/pods/%5Bpod_id%5D/name/route.ts) | No caller found in app code. |
| [`app/api/projects/[project_id]/name/route.ts`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/app/api/projects/%5Bproject_id%5D/name/route.ts) | No caller found in app code. |
| [`app/api/pulse-checks/[cycle_id]/route.ts`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/app/api/pulse-checks/%5Bcycle_id%5D/route.ts) | No caller found in app code. |
| [`app/api/pulse-checks/enforcement/route.ts`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/app/api/pulse-checks/enforcement/route.ts) | No caller found in app code. |
| [`app/api/pulse-checks/me/route.ts`](https://github.com/wash-TheUpskillingLabs/c04-olos/blob/ca543cc/app/api/pulse-checks/me/route.ts) | No caller found in app code. |

API routes may also be called by scripts or tools outside this repo. None were found here.

## Database functions

There are 34 functions, not counting the built-in encryption extension's own. Only one is never reached, and a second is called only from outside this repo:

- **`is_banned_email()`.** Never reached. The ban check is done in app code instead (`lib/auth/bans.ts`).
- **`change_participant_email()`.** Changes a person's email in both `participants` and the login. Nothing in this repo calls it, but its migration (`00085`) says the onboarding admin edge function, which lives outside this repo, does. Treat it as in use. (Corrected 2026-11-24. The first version of this report said nothing calls it.)

**Triggers.** Some fill in data on their own: `updated_at` timestamps, profile handles, role syncing and the regional auto-follows. Others only enforce rules, such as the pod membership fence and the project membership cap:

- `announcements`: `trg_announcements_updated_at→set_updated_at()`
- `connected_accounts`: `trg_connected_accounts_updated_at→set_updated_at()`
- `cycle_config`: `trg_cycle_config_updated_at→set_updated_at()`
- `cycle_enrollments`: `trg_region_follow_on_enroll→region_follow_on_enroll()`
- `cycles`: `trg_cycles_updated_at→set_updated_at()`
- `event_formats`: `trg_event_formats_updated_at→set_updated_at()`
- `event_transit`: `trg_event_transit_updated_at→set_updated_at()`
- `events`: `trg_events_updated_at→set_updated_at()`
- `field_surveys`: `trg_field_surveys_updated_at→set_updated_at()`
- `lab_leads`: `trg_sync_lab_lead→sync_lab_lead_to_proles()`
- `metros`: `trg_metros_updated_at→set_updated_at()`
- `moderator_assignments`: `trg_sync_modassign→sync_modassign_to_proles()`
- `moderator_ui_state`: `trg_moderator_ui_state_updated_at→set_updated_at()`
- `page_profiles`: `trg_page_profiles_updated_at→set_updated_at()`
- `participant_permissions`: `trg_sync_perm_admin→sync_perm_admin_to_proles()`
- `participant_roles`: `trg_guard_owner_grant→guard_owner_grant()`
- `participants`: `trg_set_participant_handle→set_participant_handle()`, `trg_guard_email→guard_email_change()`
- `pod_memberships`: `enforce_local_pod_membership_trg→enforce_local_pod_membership()`
- `pods`: `trg_pods_updated_at→set_updated_at()`, `trg_region_follow_new_pod→region_follow_new_pod()`
- `profile_update_comments`: `trg_profile_update_comments_updated_at→set_updated_at()`
- `project_memberships`: `project_membership_cap→enforce_project_membership_cap()`
- `projects`: `trg_projects_updated_at→set_updated_at()`, `trg_region_follow_new_project→region_follow_new_project()`
- `resources`: `trg_resources_updated_at→set_updated_at()`
- `spotlights`: `trg_spotlights_updated_at→set_updated_at()`
- `survey_questions`: `trg_survey_questions_updated_at→set_updated_at()`
- `user_roles`: `trg_sync_user_role→sync_user_role_to_proles()`
- `workstreams`: `trg_workstreams_updated_at→set_updated_at()`

## Corrections to the participants data dictionary

This scan found two things the dictionary got wrong. Both were corrected in the dictionary's 2026-11-24 revision, along with points from an independent Archify check.

1. **The pod leader member edit isn't reachable.** The dictionary said pod leaders can edit `preferred_name`, `phone_number` and `availability_snippet`. The route exists, but nothing in this repo calls it. Pod leaders' screens still read and show those fields. Any `availability_snippet` values come from outside this repo's code, such as direct edits in Supabase or the onboarding admin edge function.
2. **A second writer for five legacy fields.** `app/api/cycles/[cycle_id]/interest/route.ts` writes `state`, `work_situation`, `main_focus`, `sector` and `linkedin`. Nothing in this repo calls it either.

## Every table

The column counts use these short labels:

| Label | Means |
|---|---|
| W+R | Written and read |
| Auto+R | Read; filled automatically |
| R only | Read, never written |
| W only | Written, never read |
| Indirect | Indirect (name match only) |
| Select * | Loaded by select * only |
| Auto only | Filled automatically, never read |
| Orphan | Orphaned |

App R and App W are the number of app files that read or write the table. "Regional" tables come from the Greater Washington fork's own migrations (`90001` onward).

| Table | Status | Created in | Columns | W+R | Auto+R | R only | W only | Indirect | Select * | Auto only | Orphan | App R | App W | DB functions and triggers |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `access_revocations` | Active | `00001_initial_schema` | 7 | 5 | 1 |  |  | 1 |  |  |  | 4 | 3 |  |
| `admin_actions` | Written by the database, never read | `00103_admin_project_membership` | 14 |  |  |  | 12 |  |  | 2 |  | 0 | 0 | `admin_add_to_project`, `admin_move_project` |
| `agreement_acceptances` | Active | `00055_agreement_acceptances` | 6 | 3 | 1 |  | 1 | 1 |  |  |  | 1 | 1 |  |
| `ambassador_applications` | Active | `00104_ambassadors` | 23 | 20 | 2 | 1 |  |  |  |  |  | 7 | 4 |  |
| `ambassador_invites` | Active | `00104_ambassadors` | 17 | 14 | 3 |  |  |  |  |  |  | 4 | 3 |  |
| `ambassador_nominations` | Active | `00104_ambassadors` | 11 | 6 | 2 |  | 3 |  |  |  |  | 3 | 3 |  |
| `ambassador_reviews` | Active | `00105_ambassador_reviews` | 8 | 6 | 2 |  |  |  |  |  |  | 1 | 1 |  |
| `ambassador_step_progress` | Active | `00104_ambassadors` | 3 | 2 |  |  |  |  |  | 1 |  | 3 | 1 |  |
| `announcements` | Active | `00070_announcements` | 10 | 5 | 2 | 1 | 1 |  |  | 1 |  | 4 | 2 | 1 trigger |
| `auto_follows` | Written by the database, never read | `90002_cycle_auto_follows (regional)` | 4 |  |  |  | 3 |  |  | 1 |  | 0 | 0 | `region_auto_follow` |
| `baseline_responses` | Active | `00087_baseline_learning_log_weekly_messages` | 14 | 2 |  |  | 9 | 3 |  |  |  | 1 | 1 |  |
| `connected_accounts` | Active | `90001_region_handoff_and_connected_accounts (regional)` | 5 | 4 |  |  |  |  |  | 1 |  | 2 | 1 | 1 trigger |
| `custom_tasks` | Active | `00097_custom_tasks` | 14 | 11 | 2 | 1 |  |  |  |  |  | 3 | 2 |  |
| `cycle_agreements` | Active | `00032_cycle_agreements` | 7 | 4 | 2 |  | 1 |  |  |  |  | 6 | 1 |  |
| `cycle_config` | Active | `00001_initial_schema` | 40 | 33 | 3 | 1 |  | 1 | 2 |  |  | 46 | 5 | `enforce_project_membership_cap`, `reset_cycle`; 1 trigger |
| `cycle_enrollments` | Active | `00001_initial_schema` | 9 | 6 | 2 |  |  | 1 |  |  |  | 35 | 8 | `region_follow_new_pod`, `region_follow_new_project`; 1 trigger |
| `cycle_events` | Read, never written | `00086_cycle_phases_events` | 6 |  |  | 3 |  | 2 |  |  | 1 | 1 | 0 |  |
| `cycle_phases` | Active | `00086_cycle_phases_events` | 9 | 4 |  |  | 2 | 2 |  |  | 1 | 5 | 1 |  |
| `cycles` | Active | `00001_initial_schema` | 13 | 8 | 2 | 3 |  |  |  |  |  | 76 | 4 | `enforce_local_pod_membership`, `reset_cycle`; 1 trigger |
| `email_log` | Active | `00057_email_log_and_consent` | 7 | 3 |  |  | 3 | 1 |  |  |  | 1 | 1 |  |
| `event_formats` | Active | `90007_event_formats (regional)` | 4 | 3 |  |  | 1 |  |  |  |  | 3 | 1 | 1 trigger |
| `event_rsvps` | Active | `00033_public_content` | 6 | 4 | 2 |  |  |  |  |  |  | 5 | 2 |  |
| `event_transit` | Read, never written | `90005_event_transit (regional)` | 5 |  | 1 | 3 |  |  |  | 1 |  | 1 | 0 | 1 trigger |
| `events` | Active | `00033_public_content` | 29 | 20 | 4 | 4 | 1 |  |  |  |  | 8 | 2 | 1 trigger |
| `feedback` | Active | `00029_feedback` | 8 | 5 | 2 |  | 1 |  |  |  |  | 4 | 2 | `delete_participant` |
| `feedback_attachments` | Active | `00029_feedback` | 6 | 3 | 1 |  | 1 | 1 |  |  |  | 1 | 1 |  |
| `field_surveys` | Active | `00053_field_survey_intake` | 12 | 10 |  |  |  | 1 |  | 1 |  | 5 | 2 | 1 trigger |
| `follows` | Active | `00074_follows` | 6 | 3 |  | 1 | 1 | 1 |  |  |  | 7 | 2 | `region_auto_follow` |
| `handoff_redemptions` | Written, never read | `90001_region_handoff_and_connected_accounts (regional)` | 3 |  |  |  | 2 |  |  | 1 |  | 0 | 1 |  |
| `invitations` | Active | `00009_permissions_model` | 15 | 10 | 4 |  |  |  | 1 |  |  | 6 | 4 |  |
| `lab_leads` | Active | `00062_local_labs` | 6 | 4 | 2 |  |  |  |  |  |  | 7 | 2 | `sync_lab_lead_to_proles`; 1 trigger |
| `leadership_logs` | Active | `00069_leadership_logs` | 15 | 11 | 2 | 1 |  |  |  |  | 1 | 4 | 1 |  |
| `learning_logs` | Active | `00040_learning_logs` | 27 | 25 | 2 |  |  |  |  |  |  | 17 | 1 |  |
| `metro_waitlist_signups` | Active | `00033_public_content` | 4 | 2 | 1 |  |  | 1 |  |  |  | 4 | 4 |  |
| `metros` | Active | `00033_public_content` | 15 | 4 | 4 | 4 |  | 1 | 2 |  |  | 34 | 2 | 1 trigger |
| `moderator_assignments` | Active | `00001_initial_schema` | 6 | 4 | 2 |  |  |  |  |  |  | 17 | 7 | `sync_modassign_to_proles`; 1 trigger |
| `moderator_ui_state` | Active | `00024_moderator_ui_state` | 7 | 6 |  |  |  |  |  | 1 |  | 2 | 1 | 1 trigger |
| `nominations` | Active | `00010_pulse_check_v2` | 10 | 7 | 2 | 1 |  |  |  |  |  | 1 | 1 |  |
| `nudge_dismissals` | Active | `00023_nudge_dismissals` | 5 | 3 |  |  |  |  |  | 2 |  | 1 | 1 |  |
| `option_lists` | Active | `00001_initial_schema` | 5 | 3 | 2 |  |  |  |  |  |  | 6 | 1 |  |
| `owner_actions` | Written, never read | `00078_owner_actions_audit` | 10 |  |  |  | 8 | 1 |  | 1 |  | 0 | 1 | `reset_cycle`, `reset_participant`, `reset_pod`, `reset_project` |
| `page_admins` | Active | `00076_page_authored_updates` | 7 | 4 | 2 |  | 1 |  |  |  |  | 3 | 2 |  |
| `page_profiles` | Active | `90003_page_profiles (regional)` | 16 | 12 |  | 2 | 1 |  | 1 |  |  | 1 | 2 | 1 trigger |
| `participant_bans` | Active | `00102_participant_bans` | 9 | 3 | 2 |  | 4 |  |  |  |  | 3 | 1 |  |
| `participant_erasures` | Written by the database, never read | `00058_admin_roles_and_erasure` | 4 |  |  |  | 2 |  |  | 2 |  | 0 | 0 | `delete_participant` |
| `participant_options` | Active | `00001_initial_schema` | 2 | 2 |  |  |  |  |  |  |  | 3 | 3 |  |
| `participant_permissions` | Active | `00009_permissions_model` | 6 | 5 |  |  |  | 1 |  |  |  | 4 | 4 | `has_permission`, `sync_perm_admin_to_proles`; 1 trigger |
| `participant_roles` | Active | `00054_participant_roles` | 12 | 10 | 1 |  | 1 |  |  |  |  | 15 | 3 | `is_admin`, `is_lab_lead`, `is_owner`, `sync_lab_lead_to_proles` +3; 1 trigger |
| `participants` | Active | `00001_initial_schema` | 61 | 37 | 3 | 4 |  |  | 17 |  |  | 103 | 15 | `admin_add_to_project`, `admin_move_project`, `current_participant_id`, `delete_participant` +10; 2 triggers |
| `pod_memberships` | Active | `00001_initial_schema` | 6 | 3 | 2 |  |  |  |  |  | 1 | 46 | 8 | `_place_on_project`, `admin_move_project`; 1 trigger |
| `pods` | Active | `00001_initial_schema` | 13 | 6 | 3 | 3 |  |  | 1 |  |  | 70 | 8 | `enforce_local_pod_membership`, `region_follow_on_enroll`, `reset_cycle`, `reset_pod`; 2 triggers |
| `problem_statements` | Active | `00001_initial_schema` | 7 | 5 | 2 |  |  |  |  |  |  | 13 | 1 | `delete_participant` |
| `profile_update_comments` | Active | `00073_profile_update_social` | 6 | 3 | 2 |  |  |  |  | 1 |  | 1 | 2 | 1 trigger |
| `profile_update_likes` | Active | `00073_profile_update_social` | 4 | 2 |  |  |  | 2 |  |  |  | 1 | 1 |  |
| `profile_updates` | Active | `00040_learning_logs` | 9 | 5 | 2 |  | 2 |  |  |  |  | 7 | 4 |  |
| `project_memberships` | Active | `00001_initial_schema` | 6 | 5 | 1 |  |  |  |  |  |  | 10 | 2 | `_place_on_project`, `admin_add_to_project`, `admin_move_project`, `enforce_project_membership_cap`; 1 trigger |
| `project_roles` | Active | `00060_org_cycles_and_workstreams` | 7 | 5 | 2 |  |  |  |  |  |  | 6 | 3 |  |
| `project_subscriptions` | Orphaned | `00060_org_cycles_and_workstreams` | 4 |  |  |  |  |  |  | 2 | 2 | 0 | 0 |  |
| `project_votes` | Active | `00001_initial_schema` | 7 | 5 | 1 |  |  | 1 |  |  |  | 5 | 1 |  |
| `projects` | Active | `00001_initial_schema` | 15 | 7 | 3 | 1 | 1 |  | 3 |  |  | 23 | 5 | `_place_on_project`, `admin_add_to_project`, `admin_move_project`, `enforce_project_membership_cap` +4; 2 triggers |
| `pulse_checks` | Active | `00001_initial_schema` | 7 | 5 | 1 |  |  | 1 |  |  |  | 16 | 1 |  |
| `region_search_demand` | Written by the database, never read | `90006_region_search_demand (regional)` | 4 |  |  |  | 4 |  |  |  |  | 0 | 0 | `count_region_search` |
| `resources` | Active | `00033_public_content` | 17 | 8 | 3 | 5 |  | 1 |  |  |  | 3 | 2 | 1 trigger |
| `saved_items` | Active | `00050_saved_items` | 5 | 3 | 1 |  |  |  |  | 1 |  | 2 | 1 |  |
| `sectors` | Read, never written | `00049_sector_model_phase_a` | 7 |  | 2 | 3 |  |  |  | 2 |  | 3 | 0 |  |
| `simulation_sessions` | Active | `00098_simulation_sessions` | 5 | 2 | 1 |  | 1 | 1 |  |  |  | 1 | 2 |  |
| `solution_proposals` | Active | `00001_initial_schema` | 9 | 6 | 2 | 1 |  |  |  |  |  | 13 | 1 | `delete_participant` |
| `spotlights` | Active | `00051_spotlights` | 17 | 11 | 2 | 3 |  |  |  | 1 |  | 4 | 2 | 1 trigger |
| `survey_questions` | Active | `00061_survey_question_builder` | 15 | 13 |  |  |  |  |  | 2 |  | 4 | 3 | 1 trigger |
| `survey_response_answers` | Active | `00061_survey_question_builder` | 5 | 3 |  |  |  | 2 |  |  |  | 2 | 1 |  |
| `survey_responses` | Active | `00053_field_survey_intake` | 19 | 13 | 3 |  | 1 |  |  |  | 2 | 4 | 1 | `delete_participant` |
| `task_dismissals` | Active | `00096_task_dismissals` | 4 | 2 |  |  |  |  |  | 2 |  | 2 | 1 |  |
| `testers` | Active | `00042_testers` | 3 | 1 |  |  | 1 | 1 |  |  |  | 1 | 1 | `delete_participant` |
| `update_findings` | Active | `90004_multimedia_posts (regional)` | 4 | 4 |  |  |  |  |  |  |  | 2 | 1 |  |
| `update_media` | Active | `90004_multimedia_posts (regional)` | 13 | 10 |  |  | 1 | 1 |  | 1 |  | 1 | 1 |  |
| `update_reposts` | Active | `90004_multimedia_posts (regional)` | 4 | 2 | 1 |  |  | 1 |  |  |  | 3 | 1 |  |
| `user_roles` | Active | `00001_initial_schema` | 6 | 3 | 1 | 1 |  | 1 |  |  |  | 4 | 1 | `sync_perm_admin_to_proles`, `sync_user_role_to_proles`; 1 trigger |
| `votes` | Active | `00001_initial_schema` | 6 | 4 | 2 |  |  |  |  |  |  | 3 | 1 |  |
| `weekly_messages` | Active | `00088_weekly_messages_global` | 4 | 3 |  |  |  | 1 |  |  |  | 4 | 1 |  |
| `workstreams` | Active | `00060_org_cycles_and_workstreams` | 9 | 6 | 2 |  |  |  | 1 |  |  | 10 | 2 | 1 trigger |

## The CSV

`olos-column-usage.csv` has one row per column:

| Field | Contains |
|---|---|
| `table`, `column` | The column |
| `table_status` | The table's status from this report |
| `type`, `nullable`, `default`, `key` | From the database: data type, whether it can be empty, default value, primary key or the table a foreign key points to |
| `added_in_migration` | The migration that created the column |
| `column_status` | One of the eight statuses above |
| `app_writes_in` | Files that write it. "(payload rules in …)" means the fields come from a validation schema; "(row built in same file)" means the row is assembled before the call. |
| `app_reads_in` | Files that read it, filter on it or sort by it. "(via select *)" means the whole row was loaded and the field is used by name. |
| `db_functions` | Live database functions that write or read it |
| `admin_tools` | The admin data browser or owner console, if they list it |
| `name_only_in` | Files that use the table and mention the column's name, without a query that names it |
| `notes` | Database comment, legacy-import-only, set by a default, or written only by an uncalled route |

