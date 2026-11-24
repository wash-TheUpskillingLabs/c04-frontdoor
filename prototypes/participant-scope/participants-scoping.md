# participants column scoping

Where each column of the OLOS `participants` table should live: in the national schema, in the regional (Greater Washington) copy, in both, or moved to a new table.

This file is the record. To change it, open the [scoping page](./) (on the site: `/prototypes/participant-scope/`), make your changes, select **Download note**, and commit the downloaded file over this one. You can also edit the table below by hand on GitHub.

Decisions: National · Regional · Both · New table · Unsure. A blank Decision means not decided yet.

Last updated: 2026-11-24

## Decisions

| Column | Type | Flags | Decision | Note |
|---|---|---|---|---|
| `id` | serial | PK |  |  |
| `auth_user_id` | uuid | UK |  |  |
| `google_id` | varchar(255) not null | UK |  |  |
| `email` | varchar(255) not null | UK |  |  |
| `first_name` | varchar(255) not null |  |  |  |
| `last_name` | varchar(255) not null |  |  |  |
| `preferred_name` | varchar(255) |  |  |  |
| `gender` | varchar(100) |  |  |  |
| `state` | varchar(10) |  |  |  |
| `neighborhood` | varchar(255) |  |  |  |
| `dcpl_card` | varchar(20) | legacy intake |  |  |
| `dcpl_info` | boolean | legacy intake |  |  |
| `work_situation` | varchar(50) |  |  |  |
| `main_focus` | varchar(50) | legacy intake |  |  |
| `sector` | varchar(255) |  |  |  |
| `current_title` | varchar(255) |  |  |  |
| `linkedin` | varchar(500) |  |  |  |
| `ai_tool_familiarity` | smallint |  |  |  |
| `participation_commitment` | varchar(20) | legacy intake |  |  |
| `primary_expertise` | varchar(500) |  |  |  |
| `volunteer_interest` | varchar(500) |  |  |  |
| `text_updates` | boolean |  |  |  |
| `photo_video_consent` | boolean not null |  |  |  |
| `source` | varchar(255) |  |  |  |
| `slack_username` | varchar(255) |  |  |  |
| `github_username` | varchar(255) |  |  |  |
| `drive_email` | varchar(255) |  |  |  |
| `created_at` | timestamp |  |  |  |
| `profile_image_url` | text |  |  |  |
| `last_pulse_completed_at` | timestamp |  |  |  |
| `phone_number` | varchar(50) |  |  |  |
| `email_updates` | boolean |  |  |  |
| `comms_consent` | boolean not null |  |  |  |
| `availability_notes` | text |  |  |  |
| `commitment_notes` | text |  |  |  |
| `interest_areas` | text |  |  |  |
| `moderator_experience` | text |  |  |  |
| `notes` | text |  |  |  |
| `contact_consent` | boolean not null |  |  |  |
| `ai_experience_level` | ai_experience_level not null |  |  |  |
| `availability_snippet` | text |  |  |  |
| `zip` | varchar(10) |  |  |  |
| `metro_slug` | varchar(50) |  |  |  |
| `role_intents` | text[] not null |  |  |  |
| `referred_by` | varchar(255) |  |  |  |
| `agreement_version` | varchar(50) |  |  |  |
| `agreement_accepted_at` | timestamptz |  |  |  |
| `is_staff` | boolean not null |  |  |  |
| `is_test` | boolean not null |  |  |  |
| `handle` | varchar(50) |  |  |  |
| `bio` | text |  |  |  |
| `headline` | varchar(200) |  |  |  |
| `public_profile_visible` | boolean not null |  |  |  |
| `metro_id` | integer |  |  |  |
| `years_experience` | varchar |  |  |  |
| `education_level` | varchar |  |  |  |
| `sector_other` | varchar |  |  |  |
| `created_via` | varchar not null |  |  |  |
| `lab_follow_seeded` | boolean not null |  |  |  |
| `page_follows_seeded` | boolean not null |  |  |  |
| `archived_at` | timestamptz |  |  |  |

## Annotations

Discussion, open questions and context for the decisions above. Add dated entries.
