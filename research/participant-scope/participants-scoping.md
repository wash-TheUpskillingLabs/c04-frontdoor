# participants column scoping

Where each column of the OLOS `participants` table should live: in the national schema, in the regional (Greater Washington) copy, in both, or moved to a new table.

This file is the record. To change it, open the [scoping page](./) (on the site: `/research/participant-scope/`), make your changes, select **Download note**, and commit the downloaded file over this one. You can also edit the table below by hand on GitHub.

Decisions: National · Regional · Both · New table · Unsure. A blank Decision means not decided yet.

Last updated: 2026-11-19

## Decisions

| Column | Type | Flags | Decision | Note |
|---|---|---|---|---|
| `id` | serial | PK | National |  |
| `auth_user_id` | uuid | UK | National |  |
| `google_id` | varchar(255) not null | UK | National |  |
| `email` | varchar(255) not null | UK | National |  |
| `first_name` | varchar(255) not null |  | National |  |
| `last_name` | varchar(255) not null |  | National |  |
| `preferred_name` | varchar(255) |  | National |  |
| `gender` | varchar(100) |  | National |  |
| `state` | varchar(10) |  |  |  |
| `neighborhood` | varchar(255) |  |  |  |
| `dcpl_card` | varchar(20) | legacy intake | Regional |  |
| `dcpl_info` | boolean | legacy intake | Regional |  |
| `work_situation` | varchar(50) |  | New table |  |
| `main_focus` | varchar(50) | legacy intake | Regional |  |
| `sector` | varchar(255) |  | New table |  |
| `current_title` | varchar(255) |  | New table |  |
| `linkedin` | varchar(500) |  | National |  |
| `ai_tool_familiarity` | smallint |  | New table |  |
| `participation_commitment` | varchar(20) | legacy intake | New table |  |
| `primary_expertise` | varchar(500) |  | New table |  |
| `volunteer_interest` | varchar(500) |  | Regional |  |
| `text_updates` | boolean |  | Unsure |  |
| `photo_video_consent` | boolean not null |  | Regional |  |
| `source` | varchar(255) |  | Unsure |  |
| `slack_username` | varchar(255) |  | National |  |
| `github_username` | varchar(255) |  | National |  |
| `drive_email` | varchar(255) |  | Unsure |  |
| `created_at` | timestamp |  | Both |  |
| `profile_image_url` | text |  | National |  |
| `last_pulse_completed_at` | timestamp |  | Unsure |  |
| `phone_number` | varchar(50) |  | Both |  |
| `email_updates` | boolean |  | Both |  |
| `comms_consent` | boolean not null |  | Both |  |
| `availability_notes` | text |  | Regional |  |
| `commitment_notes` | text |  | Regional |  |
| `interest_areas` | text |  | New table |  |
| `moderator_experience` | text |  | Regional |  |
| `notes` | text |  | Unsure |  |
| `contact_consent` | boolean not null |  |  |  |
| `ai_experience_level` | ai_experience_level not null |  |  |  |
| `availability_snippet` | text |  |  |  |
| `zip` | varchar(10) |  | Regional |  |
| `metro_slug` | varchar(50) |  | Regional |  |
| `role_intents` | text[] not null |  | Regional |  |
| `referred_by` | varchar(255) |  | New table |  |
| `agreement_version` | varchar(50) |  | New table |  |
| `agreement_accepted_at` | timestamptz |  | New table |  |
| `is_staff` | boolean not null |  | New table |  |
| `is_test` | boolean not null |  | New table |  |
| `handle` | varchar(50) |  | Unsure |  |
| `bio` | text |  | Regional |  |
| `headline` | varchar(200) |  | Regional |  |
| `public_profile_visible` | boolean not null |  | Both |  |
| `metro_id` | integer |  | Unsure |  |
| `years_experience` | varchar |  | New table |  |
| `education_level` | varchar |  | New table |  |
| `sector_other` | varchar |  | Unsure |  |
| `created_via` | varchar not null |  | Both |  |
| `lab_follow_seeded` | boolean not null |  | Unsure |  |
| `page_follows_seeded` | boolean not null |  | Unsure |  |
| `archived_at` | timestamptz |  | Unsure |  |

## Annotations

Discussion, open questions and context for the decisions above. Add dated entries.
