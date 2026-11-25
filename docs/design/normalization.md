# Phase II – Normalization (UNF → 3NF)

This document turns the raw mbazaNLP data into relational tables in Third Normal Form (3NF). Every example row is real data from [`data/raw/tts-dataset.csv`](../../data/raw/tts-dataset.csv). The dataset is described in the [dataset card](../../data/dataset_card.md).

## 1. Starting Point
The raw file has **3,992 lines** and no header. Each line holds a clip code and a sentence:

```
TTS_1_3 "iyo nzu ni nto ariko ni nini bihagije kuri twe"
TTS_10_175 "uko binjiye baririmba ni nako basohotse"
```

Some facts apply to every clip but are given only once, on the dataset page and in its README: the publisher, link, licence, language, speaker and recording setting. A complete record of one clip therefore contains both the line from the file and these dataset-level facts.

## 2. Unnormalized Form (UNF)
All facts about a clip are written in one wide row. The dataset-level facts repeat on every one of the 3,992 rows.

| clip_code | text | source_name | source_url | licence | language | recording_setting | speaker | speaker_gender |
|---|---|---|---|---|---|---|---|---|
| TTS_1_3 | iyo nzu ni nto ariko ni nini bihagije kuri twe | mbazaNLP Kinyarwanda TTS | huggingface.co/datasets/mbazaNLP/kinyarwanda-tts-dataset | CC BY 4.0 | rw | semi-professional studio | SPK-01 | female |
| TTS_1_56 | bose bari bicaye ku meza bakina amakarita | mbazaNLP Kinyarwanda TTS | huggingface.co/datasets/mbazaNLP/kinyarwanda-tts-dataset | CC BY 4.0 | rw | semi-professional studio | SPK-01 | female |
| TTS_10_3 | iyi alubumu yabo bahisemo kuyimaraho igihe kirekire ngo bayitunganye | mbazaNLP Kinyarwanda TTS | huggingface.co/datasets/mbazaNLP/kinyarwanda-tts-dataset | CC BY 4.0 | rw | semi-professional studio | SPK-01 | female |
| TTS_10_175 | uko binjiye baririmba ni nako basohotse | mbazaNLP Kinyarwanda TTS | huggingface.co/datasets/mbazaNLP/kinyarwanda-tts-dataset | CC BY 4.0 | rw | semi-professional studio | SPK-01 | female |
| TTS_20_207 | kugeza ubu akamanzi ni umwe mu bayobozi bayo | mbazaNLP Kinyarwanda TTS | huggingface.co/datasets/mbazaNLP/kinyarwanda-tts-dataset | CC BY 4.0 | rw | semi-professional studio | SPK-01 | female |

**Problems:** `clip_code` holds two facts (session and sentence) in one value, and the dataset-level facts are repeated 3,992 times.

## 3. First Normal Form (1NF)
**Change:** split the non-atomic `clip_code` into `session_no` and `sentence_no`, so every column holds a single value. **Key:** (`session_no`, `sentence_no`).

| session_no | sentence_no | text | source_name | source_url | licence | language | recording_setting | speaker | speaker_gender |
|---|---|---|---|---|---|---|---|---|---|
| 1 | 3 | iyo nzu ni nto ariko ni nini bihagije kuri twe | mbazaNLP Kinyarwanda TTS | huggingface.co/… | CC BY 4.0 | rw | semi-professional studio | SPK-01 | female |
| 1 | 56 | bose bari bicaye ku meza bakina amakarita | mbazaNLP Kinyarwanda TTS | huggingface.co/… | CC BY 4.0 | rw | semi-professional studio | SPK-01 | female |
| 10 | 3 | iyi alubumu yabo bahisemo kuyimaraho igihe kirekire ngo bayitunganye | mbazaNLP Kinyarwanda TTS | huggingface.co/… | CC BY 4.0 | rw | semi-professional studio | SPK-01 | female |
| 10 | 175 | uko binjiye baririmba ni nako basohotse | mbazaNLP Kinyarwanda TTS | huggingface.co/… | CC BY 4.0 | rw | semi-professional studio | SPK-01 | female |
| 20 | 207 | kugeza ubu akamanzi ni umwe mu bayobozi bayo | mbazaNLP Kinyarwanda TTS | huggingface.co/… | CC BY 4.0 | rw | semi-professional studio | SPK-01 | female |

**Dependency removed:** none yet; this step removes the **non-atomic value**. The raw data has no repeating group, because each line already describes one clip.

The text cannot be the key: **73 sentences** appear in two different clips (for example `TTS_4_26` and `TTS_11_88`).

## 4. Second Normal Form (2NF)
**Partial dependency:** the source, licence, language, recording setting and speaker depend only on `session_no`, not on the full key (`session_no`, `sentence_no`). Each recording session belongs to one dataset and one speaker.

**Split:** these columns move to `RECORDING_SESSIONS`; `text` stays with the clip because it needs both parts of the key.

**RECORDING_SESSIONS** (key: `session_no`)
| session_no | source_name | source_url | licence | language | recording_setting | speaker | speaker_gender |
|---|---|---|---|---|---|---|---|
| 1 | mbazaNLP Kinyarwanda TTS | huggingface.co/… | CC BY 4.0 | rw | semi-professional studio | SPK-01 | female |
| 10 | mbazaNLP Kinyarwanda TTS | huggingface.co/… | CC BY 4.0 | rw | semi-professional studio | SPK-01 | female |
| 20 | mbazaNLP Kinyarwanda TTS | huggingface.co/… | CC BY 4.0 | rw | semi-professional studio | SPK-01 | female |

**CLIPS** (key: `session_no`, `sentence_no`)
| session_no | sentence_no | text |
|---|---|---|
| 1 | 3 | iyo nzu ni nto ariko ni nini bihagije kuri twe |
| 1 | 56 | bose bari bicaye ku meza bakina amakarita |
| 10 | 3 | iyi alubumu yabo bahisemo kuyimaraho igihe kirekire ngo bayitunganye |
| 10 | 175 | uko binjiye baririmba ni nako basohotse |
| 20 | 207 | kugeza ubu akamanzi ni umwe mu bayobozi bayo |

The repetition drops from 3,992 rows to 20 rows (one per session).

## 5. Third Normal Form (3NF)
**Transitive dependency 1:** in `RECORDING_SESSIONS`, `source_url`, `licence`, `language` and `recording_setting` depend on `source_name`, which depends on `session_no`. **Split:** they move to `DATASET_SOURCES`.

**Transitive dependency 2:** `speaker_gender` depends on `speaker`, which depends on `session_no`. **Split:** speaker details move to `SPEAKERS`.

**DATASET_SOURCES**
| source_name | source_url | licence | language | recording_setting |
|---|---|---|---|---|
| mbazaNLP Kinyarwanda TTS | huggingface.co/datasets/mbazaNLP/kinyarwanda-tts-dataset | CC BY 4.0 | rw | semi-professional studio |

**SPEAKERS**
| speaker_code | gender |
|---|---|
| SPK-01 | female |

**RECORDING_SESSIONS**
| session_no | source_name | speaker_code |
|---|---|---|
| 1 | mbazaNLP Kinyarwanda TTS | SPK-01 |
| 10 | mbazaNLP Kinyarwanda TTS | SPK-01 |
| 20 | mbazaNLP Kinyarwanda TTS | SPK-01 |

**CLIPS**
| session_no | sentence_no | text |
|---|---|---|
| 1 | 3 | iyo nzu ni nto ariko ni nini bihagije kuri twe |
| 1 | 56 | bose bari bicaye ku meza bakina amakarita |
| 10 | 3 | iyi alubumu yabo bahisemo kuyimaraho igihe kirekire ngo bayitunganye |
| 10 | 175 | uko binjiye baririmba ni nako basohotse |
| 20 | 207 | kugeza ubu akamanzi ni umwe mu bayobozi bayo |

Every non-key column now depends on the key, the whole key and nothing but the key. Today the dataset has one source and one speaker, but the design can accept more datasets and speakers (for example, mbazaNLP's multi-speaker data) without repeating their details.

## 6. From 3NF to the Logical Design
The 3NF tables above cover the **raw data**. Three design decisions complete them, and the Phase I process adds the **workflow tables**.

**Design decisions on the raw-data tables**
- **Surrogate keys** (`source_id`, `speaker_id`, `session_id`, `clip_id`). Sentence numbers have **86 gaps**, and a single-column key is simpler for the many tables that refer to a clip. (`session_id`, `sentence_no`) stays unique.
- **The text moves to `TRANSCRIPTS`** (one transcript per clip). The audio never changes, but the text is corrected, versioned and reviewed. Keeping it separate means corrections attach to the transcript, while `original_text` stays exactly as imported.
- **`clip_code` and `file_name` are kept** in `CLIPS` as unique references to the raw file and the audio file, so every row can be traced back to its source.

**Workflow tables from the Phase I process**
| Table | Comes from | Rule applied |
|---|---|---|
| `USERS` | Contributors, reviewers and admins in the process model | Repeated user details become a separate table |
| `TRANSCRIPT_VERSIONS` | "Save new version" (`submit_correction`) | A transcript has a **list** of versions, so it becomes a child table |
| `ERROR_TYPES` | Error categories chosen when flagging, correcting or rejecting | Repeated values (spelling, punctuation, mismatch, noise) become a lookup table |
| `REVIEWS` | "Save approval" / "Reject with comment" | **Many-to-many** between users and transcript versions, resolved as a junction table |
| `FLAGS` | "Flag clip" | **Many-to-many** between users and clips, resolved as a junction table |
| `AUDIT_LOG` | Audit trigger on every change and denied action | A list of events, so it becomes a child table |

The final design has **11 related tables**: the 8 tables planned in the proposal, plus `DATASET_SOURCES`, `SPEAKERS` and `RECORDING_SESSIONS`, which the normalization revealed. See the [ERD](erd.png) and the [data dictionary](data_dictionary.md).

## 7. Field Mapping
Every raw field is mapped to its target table and column, or the reason it was dropped is given. The **conversion** column is the cleaning checklist for Phase IV.

**From `tts-dataset.csv`**
| Raw field | Example | Target table.column | Conversion needed (Phase IV) |
|---|---|---|---|
| Column 1: clip code | `TTS_10_175` | `CLIPS.clip_code` | Text; no change |
| Column 1: session part | `10` | `RECORDING_SESSIONS.session_no` | Split on `_`; text → NUMBER |
| Column 1: sentence part | `175` | `CLIPS.sentence_no` | Split on `_`; text → NUMBER |
| Column 1: audio file name | `TTS 10_175.wav` | `CLIPS.file_name` | Build `TTS <session>_<sentence>.wav`; the raw code never matches the file name directly |
| Column 2: text | `uko binjiye baririmba ni nako basohotse` | `TRANSCRIPTS.original_text` | Remove the surrounding quotes; store unchanged. The 28 double spaces, missing capitals and missing punctuation are corrected through transcript versions, not on import |

**From the dataset page, README and LICENSE**
| Raw field | Example | Target table.column | Conversion needed (Phase IV) |
|---|---|---|---|
| Dataset name | Kinyarwanda TTS dataset | `DATASET_SOURCES.source_name` | None |
| Dataset link | huggingface.co/datasets/mbazaNLP/… | `DATASET_SOURCES.source_url` | None |
| Licence | CC BY 4.0 / CC0 1.0 | `DATASET_SOURCES.licence` | Resolve the conflict: store CC BY 4.0 |
| Language | `rw` | `DATASET_SOURCES.language_code` | None |
| Language creators | Digital Umuganda | `DATASET_SOURCES.creator` | None |
| Recording setting | "studio, semi-professional, some reverb" | `DATASET_SOURCES.recording_note` | None |
| Last modified | 2023-06-27 | `DATASET_SOURCES.source_updated_on` | Text → DATE |
| Date downloaded | 2026-11-25 | `DATASET_SOURCES.collected_on` | Text → DATE |
| Speaker ("a voice actress") | voice actress | `SPEAKERS.speaker_code`, `SPEAKERS.gender` | Anonymise as `SPK-01`, gender `female` |

**Dropped fields**
| Raw field | Reason |
|---|---|
| Size category (`3K<n<4K`) | Can be calculated by counting `CLIPS` |
| Duration (`~6hours`) | Approximate; can be calculated from the audio later |
| Text creation notes (syllable coverage, linguist review) | Documentation, kept in the dataset card |
| `.gitattributes` and download cache | Storage files, not data |
| Audio content (WAV) | Stored in object storage, not in the database; only `CLIPS.file_name` is kept |

**New columns not in the raw data** (filled by the system): `CLIPS.status` (starts as `PENDING`), `CLIPS.imported_at`, `DATASET_SOURCES.storage_base_url` (set when the audio is uploaded), and all workflow-table columns.

### Phase IV cleaning checklist
1. Split 3,992 clip codes into session and sentence numbers (text → NUMBER).
2. Build 3,992 audio file names; 0 match the raw code without conversion.
3. Remove the surrounding quotes from 3,992 text values.
4. Keep the 73 duplicate sentences as separate clips; do not merge them.
5. Leave the 28 double spaces, missing capitals and missing punctuation in `original_text`; they are fixed through the review process (`normalize_text`).
6. Load records in (session, sentence) order; 1 record is out of order (`TTS_9_137`).
7. Convert the two dataset dates from text to DATE.

## 8. Tracing One Record (Class Demonstration)
Raw line 1,572 in `tts-dataset.csv`:

```
TTS_10_175 "uko binjiye baririmba ni nako basohotse"
```

| Raw value | Ends up in | Stored value |
|---|---|---|
| `TTS_10_175` | `CLIPS.clip_code` | `TTS_10_175` |
| `10` (from the code) | `RECORDING_SESSIONS.session_no` → linked by `CLIPS.session_id` | `10` |
| `175` (from the code) | `CLIPS.sentence_no` | `175` |
| (built from the code) | `CLIPS.file_name` | `TTS 10_175.wav` |
| `uko binjiye baririmba ni nako basohotse` | `TRANSCRIPTS.original_text` | same text, without quotes |
| Dataset name, link, licence, language | `DATASET_SOURCES` (one row), linked by `RECORDING_SESSIONS.source_id` | mbazaNLP Kinyarwanda TTS, CC BY 4.0, `rw` |
| "voice actress" | `SPEAKERS` (one row), linked by `RECORDING_SESSIONS.speaker_id` | `SPK-01`, female |
| (new) | `CLIPS.status` | `PENDING` |

From this point the workflow tables take over: a contributor's correction is saved in `TRANSCRIPT_VERSIONS`, reviewers' decisions in `REVIEWS`, and every change in `AUDIT_LOG`.
