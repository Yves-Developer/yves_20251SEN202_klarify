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

## Repository Structure
```
docs/research/       Research log
docs/proposal/       Project proposal
docs/presentation/   Presentation slides
data/                Data source and sample records
innovation/          Innovation documentation
```

## Project Phases
| Phase | Focus | Status |
|---|---|---|
| 0 | Research, data source and proposal | In progress |
| I | Business process modeling | Not started |
| II | Data sourcing and normalization | Not started |
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
