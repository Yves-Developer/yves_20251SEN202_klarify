# Klarify: Kinyarwanda TTS Dataset Review and Verification Platform

| | |
|---|---|
| **Student** | MUGISHA Yves |
| **Student ID** | 20251SEN202 |
| **Course** | INSY 8311 – Database Development with PL/SQL |
| **Instructor** | Eric Maniraguha |
| **Institution** | Adventist University of Central Africa (AUCA) |

## Problem Statement
Kinyarwanda text-to-speech models need large amounts of accurately transcribed audio, but available datasets are scarce and contain spelling, punctuation and transcription errors. Klarify is a platform built on an Oracle database where native speakers listen to audio clips, correct or write the matching Kinyarwanda transcript, and peer reviewers verify each edit. PL/SQL procedures, triggers and auditing enforce the review rules, producing a clean, training-ready TTS dataset.

## Objectives
**Main objective:** Build an Oracle PL/SQL–based platform that produces a verified, training-ready Kinyarwanda TTS dataset.

**Specific objectives:**
1. Design a normalized (3NF) Oracle database for clips, transcripts, versions, reviews and flags.
2. Implement PL/SQL procedures, functions and packages for transcript correction, review and export.
3. Enforce review rules with triggers (e.g. approval after two reviewer agreements, no self-review) and log every change in an audit table.
4. Deliver a web interface connected to Oracle for contributors and reviewers.

## Data Source
- **Dataset:** [mbazaNLP/kinyarwanda-tts-dataset](https://huggingface.co/datasets/mbazaNLP/kinyarwanda-tts-dataset)
- **Size:** 3,992 audio clips (`.wav`) with transcripts (`tts-dataset.csv`)
- **License:** CC-BY-4.0
- **Access:** Public, gated (free Hugging Face login and accept terms)
- Audio files are **not** stored in this repository (gated and large).

## Innovation
**Other – User interface:** a custom Next.js/React web app connected to Oracle via `node-oracledb`. Contributors play audio clips and correct Kinyarwanda transcripts; reviewers approve or reject them. Every action calls PL/SQL package procedures; triggers enforce approval rules and audit logging. A dashboard page shows review progress and common error types.

## Planned Tables
| Table | Purpose |
|---|---|
| `USERS` | Contributors and reviewers (roles) |
| `CLIPS` | Audio clip metadata (file name, duration, status) |
| `TRANSCRIPTS` | Current transcript per clip |
| `TRANSCRIPT_VERSIONS` | Full history of every edit |
| `REVIEWS` | Reviewer decisions (approve / reject) |
| `FLAGS` | Reports of bad audio or text |
| `ERROR_TYPES` | Lookup: spelling, punctuation, mismatch, noise… |
| `AUDIT_LOG` | Record of all inserts, updates and denied actions |

## Planned PL/SQL Features
- **Procedures:** `submit_correction` saves a new transcript version; `review_clip` records a reviewer decision.
- **Functions:** `fn_agreement_count` returns how many reviewers approved a version; `fn_normalize_text` cleans spacing and punctuation.
- **Cursors:** iterate pending clips to assign them to reviewers; build export batches.
- **Packages:** `pkg_klarify` groups transcription, review and export logic.
- **Triggers:** auto-approve a clip once two reviewers agree; block a user from reviewing their own transcript; write every change to `AUDIT_LOG`.
- **Exceptions:** custom errors for self-review, editing approved clips and invalid status changes.

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
| 0 | Research, data source, proposal | In progress |
| I | Business process modeling | Not started |
| II | Data sourcing and normalization | Not started |
| III | Database (PDB) creation | Not started |
| IV | Tables and data loading | Not started |
| V | PL/SQL development | Not started |
| VI | Advanced programming and auditing | Not started |
| VII | Documentation, innovation, final presentation | Not started |

## References
See [`docs/research/research_log.md`](docs/research/research_log.md).

- mbazaNLP. (2023). *Kinyarwanda TTS dataset* [Data set]. Hugging Face. https://huggingface.co/datasets/mbazaNLP/kinyarwanda-tts-dataset
