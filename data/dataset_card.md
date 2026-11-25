# Dataset Card: Kinyarwanda TTS Dataset (mbazaNLP)

## Summary
| | |
|---|---|
| **Name** | Kinyarwanda TTS dataset |
| **Publisher** | mbazaNLP (language creators: Digital Umuganda) |
| **Source link** | https://huggingface.co/datasets/mbazaNLP/kinyarwanda-tts-dataset |
| **Licence** | CC BY 4.0 on the dataset page; the included `LICENSE` file says CC0 1.0 (see note below) |
| **Language** | Kinyarwanda (`rw`) |
| **Date collected** | 25 November 2026 (downloaded with the Hugging Face CLI) |
| **Dataset version** | Last updated by the publisher on 27 June 2023 |
| **Raw file** | [`raw/tts-dataset.csv`](raw/tts-dataset.csv), kept exactly as downloaded |
| **Checksum (SHA-256)** | `fd15c12e82156037c13a9ce6adb36d926791f455df024b0a51fde4b238dd746f` |

**Licence note.** The dataset page lists CC BY 4.0, while the `LICENSE` file in the download contains CC0 1.0. Both allow educational use and redistribution. This project follows the stricter licence, CC BY 4.0, and credits mbazaNLP.

## Format and Size
| Part | Format | Size |
|---|---|---|
| Text | `tts-dataset.csv`: plain text, ASCII, LF line endings, **no header row**; each line holds a clip code and the sentence in double quotes, separated by a space | 3,992 records (391,307 bytes) |
| Audio | 3,992 WAV files in `audio/`, named `TTS <session>_<sentence>.wav` | about 2.1 GB |

The audio files are **not stored in this repository** because of their size. They are kept locally and will be moved to object storage when the clips are loaded (Phase IV). The database stores only each clip's file name.

## What One Record Represents
One record is **one Kinyarwanda sentence read aloud in a studio by a single voice actress**. It holds the clip code and the written text of the sentence. The clip code links the text to its audio file.

```
TTS_10_175 "uko binjiye baririmba ni nako basohotse"
```

| Field (named by this project) | Example | Meaning |
|---|---|---|
| `clip_code` | `TTS_10_175` | Recording session 10, sentence 175 |
| `text` | `uko binjiye baririmba ni nako basohotse` | The sentence read in the audio |

The matching audio file is `TTS 10_175.wav`.

## Personal Data
The dataset contains **no names, phone numbers or ID numbers**. The only person involved is the voice actress, who is not named in the source. She is recorded in this project as the anonymous speaker `SPK-01`. Contributor and reviewer accounts are created later by the application, not imported from this dataset.

## Quality Report
All counts below come from the raw file and the downloaded audio. **Nothing has been changed**; cleaning takes place in Phase IV.

### File structure
| Check | Count |
|---|---|
| Records (lines) | 3,992 |
| Blank lines | 0 |
| Lines that do not follow `CODE "text"` | 0 |
| Header row | **missing**: columns have no names |
| Separator | **space**, not comma, despite the `.csv` extension |
| Non-ASCII characters | 0 |
| Records out of (session, sentence) order | 1 (`TTS_9_137` comes before `TTS_9_135`) |

### Clip codes and audio files
| Check | Count |
|---|---|
| Codes in the form `TTS_<session>_<sentence>` | 3,992 |
| Duplicate codes | 0 |
| Codes that hold two facts (session and sentence) in one value | 3,992 |
| Codes that match the audio file name **without conversion** | **0**: `TTS_10_175` vs `TTS 10_175.wav` |
| Records that match an audio file **after conversion** | 3,992 |
| Records with no audio file | 0 |
| Audio files with no record | 0 |
| Audio file names containing a space | 3,992 |
| Recording sessions | 20 (`TTS 1` to `TTS 20`) |
| Clips per session | 61 to 298 (average 199.6) |
| Missing sentence numbers inside sessions | **86**, across 18 of 20 sessions (session 13 alone: 16) |
| Sessions that do not start at sentence 1 | 3 (sessions 1 and 8 start at 2; session 4 starts at 4) |

### Transcript text
| Check | Count |
|---|---|
| Empty text | 0 |
| **Duplicate sentences** (same text, different clips) | **73** sentences appear twice, e.g. `TTS_4_26` and `TTS_11_88` |
| Leading or trailing spaces | 0 |
| **Double spaces inside the text** | **28** |
| Text containing capital letters | **0**: all text is lowercase, including names such as *rwanda* |
| Text ending with `.`, `?` or `!` | **0**: no sentence has final punctuation |
| Text containing commas | 0 |
| Text containing digits | 0: numbers are written as words |
| Text containing an apostrophe | 1,952 (2,769 apostrophes in total) |
| Apostrophes used as quotation marks | 14 records (3 use a doubled `''`) |
| Foreign words with *q* (e.g. *pratique*, *mozambique*) | 2 |
| Words per sentence | 3 to 24 (median 12) |
| Characters per sentence | 19 to 179 (median 84) |
| Total words / distinct words | 47,440 / 14,675 |

### What these results mean
- The text is already **consistent**: no empty values, no digits, no stray spaces at the ends.
- It is **not ready for display or TTS training as it is**: there are no capital letters and no punctuation, and 28 records contain double spaces. Contributors will add these corrections through the review process.
- The clip code must be **split and converted** to link text to audio and to store the session and sentence as numbers.
- The 73 duplicate sentences are **separate recordings**, so they remain separate clips. The text cannot be used as a key.
- The 86 gaps show that **sentence numbers are not continuous**, so each clip needs its own identifier.
