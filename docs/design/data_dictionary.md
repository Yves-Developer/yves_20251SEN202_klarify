# Phase II – Data Dictionary

Logical design of the Klarify database: 11 tables in Third Normal Form. Oracle data types and sizes are added in Phase IV. The normalization steps are in [normalization.md](normalization.md) and the diagram is in [erd.png](erd.png).

**Legend:** PK = primary key, FK = foreign key. **Required** = a value must always be given. **Unique** = no two rows may hold the same value.

## Tables at a Glance
| # | Table | Purpose | Source |
|---|---|---|---|
| 1 | `DATASET_SOURCES` | Datasets imported into Klarify | Raw data (dataset page) |
| 2 | `SPEAKERS` | Anonymised speakers heard in the audio | Raw data (README) |
| 3 | `RECORDING_SESSIONS` | Recording sessions (`TTS 1` to `TTS 20`) | Raw data (clip codes) |
| 4 | `CLIPS` | One audio clip and its workflow status | Raw data (clip codes) |
| 5 | `TRANSCRIPTS` | The original text of each clip | Raw data (text) |
| 6 | `TRANSCRIPT_VERSIONS` | Every correction of a transcript | Phase I process |
| 7 | `USERS` | Contributors, reviewers and admins | Phase I process |
| 8 | `REVIEWS` | Reviewer decisions (junction: users and versions) | Phase I process |
| 9 | `ERROR_TYPES` | Categories of errors | Phase I process |
| 10 | `FLAGS` | Reports of poor audio or text (junction: users and clips) | Phase I process |
| 11 | `AUDIT_LOG` | Record of all changes and denied actions | Phase I process |

## 1. DATASET_SOURCES
A dataset imported into Klarify.

| Column | Meaning | Key | Required | Unique |
|---|---|---|---|---|
| `source_id` | Identifier of the dataset | PK | Yes | Yes |
| `source_name` | Name of the dataset | | Yes | Yes |
| `source_url` | Link to the original dataset | | Yes | Yes |
| `licence` | Licence under which the data is used | | Yes | No |
| `language_code` | Language of the recordings (`rw`) | | Yes | No |
| `creator` | Organisation that created the data | | No | No |
| `recording_note` | How the audio was recorded | | No | No |
| `storage_base_url` | Address of the folder that holds the audio files | | No | No |
| `source_updated_on` | Date the publisher last updated the dataset | | No | No |
| `collected_on` | Date the dataset was downloaded | | Yes | No |

## 2. SPEAKERS
A person whose voice is heard in the clips. Speakers are anonymised.

| Column | Meaning | Key | Required | Unique |
|---|---|---|---|---|
| `speaker_id` | Identifier of the speaker | PK | Yes | Yes |
| `speaker_code` | Anonymous label, e.g. `SPK-01` | | Yes | Yes |
| `gender` | Gender of the speaker, if known | | No | No |
| `description` | Short description, e.g. "voice actress" | | No | No |

## 3. RECORDING_SESSIONS
One recording session of a dataset, identified by the number in the clip code (`TTS_10_…` = session 10).

| Column | Meaning | Key | Required | Unique |
|---|---|---|---|---|
| `session_id` | Identifier of the session | PK | Yes | Yes |
| `source_id` | Dataset the session belongs to | FK → `DATASET_SOURCES` | Yes | No |
| `speaker_id` | Speaker recorded in the session | FK → `SPEAKERS` | Yes | No |
| `session_no` | Session number from the clip code | | Yes | Unique together with `source_id` |

## 4. CLIPS
One recorded sentence (audio file) and its position in the review workflow.

| Column | Meaning | Key | Required | Unique |
|---|---|---|---|---|
| `clip_id` | Identifier of the clip | PK | Yes | Yes |
| `session_id` | Session the clip was recorded in | FK → `RECORDING_SESSIONS` | Yes | No |
| `sentence_no` | Sentence number within the session | | Yes | Unique together with `session_id` |
| `clip_code` | Code from the raw file, e.g. `TTS_10_175` | | Yes | Yes |
| `file_name` | Audio file name, e.g. `TTS 10_175.wav` | | Yes | Yes |
| `status` | Workflow status: `PENDING`, `FLAGGED`, `IN_REVIEW`, `NEEDS_REWORK`, `APPROVED` or `ARCHIVED` | | Yes | No |
| `assigned_to` | Contributor currently working on the clip | FK → `USERS` | No | No |
| `imported_at` | Date and time the clip was loaded | | Yes | No |

## 5. TRANSCRIPTS
The text of a clip as it was imported. It never changes; corrections are stored as versions.

| Column | Meaning | Key | Required | Unique |
|---|---|---|---|---|
| `transcript_id` | Identifier of the transcript | PK | Yes | Yes |
| `clip_id` | Clip the transcript belongs to | FK → `CLIPS` | Yes | Yes (one transcript per clip) |
| `original_text` | Sentence exactly as in the raw file | | Yes | No (73 sentences appear in two clips) |
| `created_at` | Date and time the transcript was loaded | | Yes | No |

## 6. TRANSCRIPT_VERSIONS
Each correction submitted by a contributor. The latest version is the current text of the clip.

| Column | Meaning | Key | Required | Unique |
|---|---|---|---|---|
| `version_id` | Identifier of the version | PK | Yes | Yes |
| `transcript_id` | Transcript that was corrected | FK → `TRANSCRIPTS` | Yes | No |
| `version_no` | Order of the version (1, 2, 3 …) | | Yes | Unique together with `transcript_id` |
| `corrected_text` | Text after the correction | | Yes | No |
| `edited_by` | Contributor who made the correction | FK → `USERS` | Yes | No |
| `error_type_id` | Main type of error that was corrected | FK → `ERROR_TYPES` | No | No |
| `change_note` | Short explanation of the change | | No | No |
| `created_at` | Date and time the version was saved | | Yes | No |

## 7. USERS
A person who uses Klarify.

| Column | Meaning | Key | Required | Unique |
|---|---|---|---|---|
| `user_id` | Identifier of the user | PK | Yes | Yes |
| `username` | Login name | | Yes | Yes |
| `full_name` | Full name of the user | | Yes | No |
| `email` | Email address | | Yes | Yes |
| `role` | `CONTRIBUTOR`, `REVIEWER` or `ADMIN` | | Yes | No |
| `is_active` | Whether the account can be used | | Yes | No |
| `created_at` | Date and time the account was created | | Yes | No |

## 8. REVIEWS
A reviewer's decision on one transcript version. This junction table resolves the many-to-many relationship between `USERS` and `TRANSCRIPT_VERSIONS`.

| Column | Meaning | Key | Required | Unique |
|---|---|---|---|---|
| `review_id` | Identifier of the review | PK | Yes | Yes |
| `version_id` | Version that was reviewed | FK → `TRANSCRIPT_VERSIONS` | Yes | Unique together with `reviewer_id` |
| `reviewer_id` | Reviewer who made the decision | FK → `USERS` | Yes | Unique together with `version_id` |
| `decision` | `APPROVED` or `REJECTED` | | Yes | No |
| `comment_text` | Reason for the decision | | Required when `REJECTED` | No |
| `error_type_id` | Type of error found, when rejected | FK → `ERROR_TYPES` | No | No |
| `reviewed_at` | Date and time of the decision | | Yes | No |

## 9. ERROR_TYPES
A category of error used when flagging, correcting or rejecting.

| Column | Meaning | Key | Required | Unique |
|---|---|---|---|---|
| `error_type_id` | Identifier of the error type | PK | Yes | Yes |
| `code` | Short code, e.g. `SPELLING`, `PUNCTUATION`, `MISMATCH`, `NOISE` | | Yes | Yes |
| `name` | Name shown to users | | Yes | Yes |
| `description` | Explanation of the error type | | No | No |

## 10. FLAGS
A report that a clip's audio or text cannot be corrected normally. This junction table resolves the many-to-many relationship between `USERS` and `CLIPS`.

| Column | Meaning | Key | Required | Unique |
|---|---|---|---|---|
| `flag_id` | Identifier of the flag | PK | Yes | Yes |
| `clip_id` | Clip that was flagged | FK → `CLIPS` | Yes | No |
| `flagged_by` | Contributor who raised the flag | FK → `USERS` | Yes | No |
| `error_type_id` | Type of problem | FK → `ERROR_TYPES` | Yes | No |
| `reason` | Description of the problem | | No | No |
| `status` | `OPEN` or `RESOLVED` | | Yes | No |
| `resolution` | `RETURNED` (back to the queue) or `ARCHIVED` | | Required when `RESOLVED` | No |
| `resolved_by` | Admin who resolved the flag | FK → `USERS` | Required when `RESOLVED` | No |
| `created_at` | Date and time the flag was raised | | Yes | No |
| `resolved_at` | Date and time the flag was resolved | | Required when `RESOLVED` | No |

## 11. AUDIT_LOG
One recorded change or denied action. Rows are written by triggers and never edited.

| Column | Meaning | Key | Required | Unique |
|---|---|---|---|---|
| `audit_id` | Identifier of the log entry | PK | Yes | Yes |
| `user_id` | User who performed the action, if known | FK → `USERS` | No | No |
| `table_name` | Table that was affected | | Yes | No |
| `record_id` | Identifier of the affected row | | Yes | No |
| `action` | `INSERT`, `UPDATE`, `DELETE` or `DENIED` | | Yes | No |
| `old_value` | Values before the change | | No | No |
| `new_value` | Values after the change | | No | No |
| `action_time` | Date and time of the action | | Yes | No |

## Relationships
Cardinality is read from the first table to the second. Optionality shows whether a row must take part in the relationship.

| Relationship | Cardinality | Optionality |
|---|---|---|
| `DATASET_SOURCES` → `RECORDING_SESSIONS` | One to many | A source has one or more sessions; a session belongs to exactly one source |
| `SPEAKERS` → `RECORDING_SESSIONS` | One to many | A speaker has zero or more sessions; a session has exactly one speaker |
| `RECORDING_SESSIONS` → `CLIPS` | One to many | A session has one or more clips; a clip belongs to exactly one session |
| `CLIPS` → `TRANSCRIPTS` | One to one | Each clip has exactly one transcript, and each transcript belongs to exactly one clip |
| `TRANSCRIPTS` → `TRANSCRIPT_VERSIONS` | One to many | A transcript has zero or more versions; a version belongs to exactly one transcript |
| `USERS` → `TRANSCRIPT_VERSIONS` | One to many | A user edits zero or more versions; a version has exactly one editor |
| `USERS` ↔ `TRANSCRIPT_VERSIONS` through `REVIEWS` | Many to many | A user reviews zero or more versions; a version receives zero or more reviews |
| `USERS` ↔ `CLIPS` through `FLAGS` | Many to many | A user raises zero or more flags; a clip receives zero or more flags |
| `USERS` → `FLAGS` (resolved by) | One to many | A flag is resolved by zero or one admin |
| `USERS` → `CLIPS` (assigned to) | One to many | A clip is assigned to zero or one contributor |
| `ERROR_TYPES` → `FLAGS` | One to many | A flag has exactly one error type |
| `ERROR_TYPES` → `TRANSCRIPT_VERSIONS` | One to many | A version has zero or one error type |
| `ERROR_TYPES` → `REVIEWS` | One to many | A review has zero or one error type |
| `USERS` → `AUDIT_LOG` | One to many | A log entry has zero or one user |

## Business Rules
1. A clip is approved when its latest transcript version has **two `APPROVED` reviews from two different reviewers**.
2. A reviewer cannot review a version they edited themselves (self-review).
3. An approved clip cannot be edited.
4. `TRANSCRIPTS.original_text` is never changed; the current text of a clip is its latest version, or the original text if no version exists.
5. Archiving a clip changes its status to `ARCHIVED`; the clip and its history are not deleted.
6. Every insert, update and denied action is written to `AUDIT_LOG`.
