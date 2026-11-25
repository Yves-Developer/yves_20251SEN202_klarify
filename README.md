# Klarify: Kinyarwanda TTS Dataset Review and Verification Platform

| | |
|---|---|
| **Student** | MUGISHA Yves |
| **Student ID** | 20251SEN202 |
| **Course** | INSY 8311 – Database Development with PL/SQL |
| **Instructor** | Eric Maniraguha |
| **Institution** | Adventist University of Central Africa (AUCA) |

## Problem Statement
Text-to-speech (TTS) systems require large amounts of accurately transcribed audio. For Kinyarwanda, such data is scarce, and existing datasets contain spelling, punctuation and transcription errors. These errors reduce the quality of TTS models, and no structured process exists to correct and verify the data.

## Proposed Solution
Klarify is a platform built on an Oracle database. Native speakers listen to audio clips and correct the matching Kinyarwanda transcripts, and peer reviewers verify each correction. PL/SQL procedures, triggers and auditing enforce the review rules. The result is a verified dataset ready for TTS model training.

## Objectives
**Main objective:** To develop an Oracle PL/SQL platform that produces a verified Kinyarwanda TTS dataset.

**Specific objectives:**
1. To design a normalized (3NF) database for audio clips, transcripts, reviews and flags.
2. To implement PL/SQL procedures, functions and packages for transcript correction, review and export.
3. To enforce review rules with triggers and record every change in an audit log.
4. To develop a web interface, connected to the database, for contributors and reviewers.

## Data Source
| | |
|---|---|
| **Dataset** | [mbazaNLP/kinyarwanda-tts-dataset](https://huggingface.co/datasets/mbazaNLP/kinyarwanda-tts-dataset) |
| **Size** | 3,992 audio clips with transcripts |
| **Licence** | CC BY 4.0 |
| **Access** | Public; requires a free Hugging Face account and acceptance of the terms |

Audio files are not stored in this repository because of their size and access terms.

## Innovation
**Type:** User interface (custom web application)

A Next.js web application connects to the Oracle database through `node-oracledb`. Contributors play audio clips and correct transcripts, and reviewers approve or reject them. Each action calls a PL/SQL procedure, so all business rules remain in the database. A dashboard shows review progress and common error types.

## Planned Tables
| Table | Purpose |
|---|---|
| `USERS` | Contributors and reviewers |
| `CLIPS` | Audio clip details and status |
| `TRANSCRIPTS` | Current transcript of each clip |
| `TRANSCRIPT_VERSIONS` | History of all transcript edits |
| `REVIEWS` | Reviewer decisions |
| `FLAGS` | Reports of poor audio or text |
| `ERROR_TYPES` | Categories of errors (spelling, punctuation, mismatch, noise) |
| `AUDIT_LOG` | Record of all changes and denied actions |

## Planned PL/SQL Features
| Feature | Where it is used |
|---|---|
| **Procedures** | `submit_correction` saves a new transcript version; `review_clip` records a reviewer decision. |
| **Functions** | `get_approval_count` returns the number of approvals for a version; `normalize_text` corrects spacing and punctuation. |
| **Cursors** | Process pending clips for reviewer assignment and dataset export. |
| **Package** | `pkg_klarify` groups the correction, review and export logic. |
| **Triggers** | Approve a clip after two reviewers agree; prevent users from reviewing their own work; write every change to `AUDIT_LOG`. |
| **Exceptions** | Handle self-review, edits to approved clips and invalid status changes. |

## Phase I – Business Process Modeling
**Selected process: Transcript Review and Verification.** This process turns raw Kinyarwanda audio clips and their transcripts into a verified dataset for TTS training. A contributor corrects each transcript, two peer reviewers must approve it, and the Oracle database records every step. Unusable clips are flagged and either returned to the queue or archived.

| | |
|---|---|
| **Modeling tool** | diagrams.net (draw.io) |
| **Notation** | BPMN with swimlanes |
| **Published diagram** | [View diagram](https://viewer.diagrams.net/?tags=%7B%7D&lightbox=1&highlight=0000ff&edit=_blank&layers=1&nav=1&title=Klarify_Phase%20I_Business_Process.drawio&dark=auto#Uhttps%3A%2F%2Fdrive.google.com%2Fuc%3Fid%3D1KrMvxsCjUMIlmRRoZo_3l2iIdk5_ytnb%26export%3Ddownload) |
| **Diagram (PDF)** | [business_process_diagram.pdf](docs/business_process/business_process_diagram.pdf) |
| **Explanation** | [business_process_explanation.pdf](docs/business_process/business_process_explanation.pdf) |

### Actors and Responsibilities
| Actor | Responsibility |
|---|---|
| **Admin** | Imports the dataset, reviews flagged clips and decides whether to keep or archive them. |
| **Contributor** | Listens to each clip, flags unusable audio and corrects the transcript. |
| **Peer Reviewer** | Compares the audio with the corrected transcript and approves or rejects it. |
| **System (Oracle DB)** | Stores all data and runs the PL/SQL procedures and triggers that enforce the review rules. |

### Data Activities
| Activity | Type | Data created or used |
|---|---|---|
| Save clips as PENDING | Create | New rows in `CLIPS` and `TRANSCRIPTS` |
| Listen to audio and read transcript | Retrieve | Clip audio and current transcript |
| Flag clip | Create | New row in `FLAGS` with reason and error type |
| Review flagged clip | Retrieve | Flag details and clip |
| Send clip back to queue | Update | Clip status returns to `PENDING` |
| Archive clip | Delete / Archive | Clip status set to `ARCHIVED`; history is kept |
| Save new version (`submit_correction`) | Create, Update | New row in `TRANSCRIPT_VERSIONS`; `TRANSCRIPTS` updated; `AUDIT_LOG` entry |
| Review audio vs transcript | Retrieve | Clip and corrected transcript |
| Reject with comment | Create | New row in `REVIEWS` (rejected) |
| Save approval (`review_clip`) | Create | New row in `REVIEWS` (approved) |
| Mark clip APPROVED | Update | Clip status set to `APPROVED` |
| Export verified dataset | Retrieve | All approved clips and transcripts |

## Phase II – Data Sourcing, Normalization and Logical Design
In this phase, the raw dataset is organised into relational tables in Third Normal Form (3NF). The result is the logical design that will be built in Oracle in Phase IV.

### Raw Data
| | |
|---|---|
| **File** | [`data/raw/tts-dataset.csv`](data/raw/tts-dataset.csv), kept exactly as downloaded |
| **Records** | 3,992 (one Kinyarwanda sentence and its clip code per record) |
| **Format** | Plain text, no header row, values separated by a space |
| **Licence** | CC BY 4.0 |
| **Collected** | 25 November 2026 |
| **Description** | [Dataset card](data/dataset_card.md) |

The 3,992 audio files (about 2.1 GB) are not stored in this repository because of their size. The database stores only each clip's file name.

### Data Quality
The quality check counts problems but does not fix them; cleaning takes place in Phase IV. The main results are:
- **0** clip codes match their audio file name without conversion (`TTS_10_175` vs `TTS 10_175.wav`).
- **73** sentences appear in two different clips, so the text cannot be used as a key.
- **86** sentence numbers are missing within the recording sessions.
- **28** records contain double spaces, and no record has capital letters or final punctuation.

### Normalization
| Form | Change made |
|---|---|
| **UNF** | One wide row per clip; the dataset details repeat on all 3,992 rows. |
| **1NF** | The clip code is split into a session number and a sentence number, so each column holds one value. |
| **2NF** | Speaker and source details depend only on the session, so they move to `RECORDING_SESSIONS`. |
| **3NF** | Licence and language depend on the source, and gender depends on the speaker, so they move to `DATASET_SOURCES` and `SPEAKERS`. |

Each step is shown with real rows in [normalization.md](docs/design/normalization.md), together with the field mapping and the Phase IV cleaning checklist.

### Logical Design
The final design has **11 related tables**: the 8 tables planned in the proposal and 3 new tables found through normalization.

| Group | Tables |
|---|---|
| From the raw data | `DATASET_SOURCES`, `SPEAKERS`, `RECORDING_SESSIONS`, `CLIPS`, `TRANSCRIPTS` |
| From the business process | `USERS`, `TRANSCRIPT_VERSIONS`, `REVIEWS`, `FLAGS`, `ERROR_TYPES`, `AUDIT_LOG` |

`REVIEWS` and `FLAGS` are junction tables that resolve many-to-many relationships between users and the data they work on.

| Document | Content |
|---|---|
| [Dataset card](data/dataset_card.md) | Source, licence, format, size and quality report |
| [Normalization](docs/design/normalization.md) | UNF → 3NF with real rows, field mapping and record trace |
| [ERD](docs/design/erd.png) | Logical ERD in crow's-foot notation |
| [Data dictionary](docs/design/data_dictionary.md) | Every table and column, with keys and constraints |

## Repository Structure
```
docs/research/       Research log
docs/proposal/       Project proposal
docs/presentation/   Presentation slides
docs/business_process/   Phase I diagram and explanation
docs/design/         Phase II normalization, ERD and data dictionary
data/raw/            Raw dataset, exactly as downloaded
data/                Dataset card
innovation/          Innovation documentation
```

## Project Phases
| Phase | Focus | Status |
|---|---|---|
| 0 | Research, data source and proposal | Completed |
| I | Business process modeling | Completed |
| II | Data sourcing and normalization | Completed |
| III | Pluggable database creation | Not started |
| IV | Tables and data loading | Not started |
| V | PL/SQL development | Not started |
| VI | Advanced programming and auditing | Not started |
| VII | Documentation, innovation and final presentation | Not started |

## References
mbazaNLP. (2023). *Kinyarwanda TTS dataset* [Data set]. Hugging Face. https://huggingface.co/datasets/mbazaNLP/kinyarwanda-tts-dataset

Mozilla Foundation. (n.d.). *Common Voice*. Retrieved November 13, 2026, from https://commonvoice.mozilla.org/

National Institute of Statistics of Rwanda. (2023). *Fifth Rwanda Population and Housing Census, 2022: Thematic report on education characteristics of the population*. https://statistics.gov.rw/sites/default/files/documents/2025-02/Education.pdf

Novotney, S., & Callison-Burch, C. (2010). Cheap, fast and good enough: Automatic speech recognition with non-expert transcription. In *Proceedings of Human Language Technologies: The 2010 Annual Conference of the North American Chapter of the Association for Computational Linguistics* (pp. 207–215). Association for Computational Linguistics. https://aclanthology.org/N10-1024/

Oracle. (n.d.). *Executing PL/SQL*. In *node-oracledb documentation*. Retrieved November 13, 2026, from https://node-oracledb.readthedocs.io/en/latest/user_guide/plsql_execution.html

Rutunda, S., Kabanda, K., & Stan, A. (2023). *Kinyarwanda TTS: Using a multi-speaker dataset to build a Kinyarwanda TTS model* [Paper presentation]. ICLR 2023 Workshop on AfricaNLP, Kigali, Rwanda. https://openreview.net/forum?id=1gLgrqWnHF

Details of each source are in the [research log](docs/research/research_log.md).
