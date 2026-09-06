# Estuary batch character test PDFs

100 synthetic character dossier PDFs for 73 fictional characters, hosted here so
the Estuary batch character creation endpoint (`POST /api/v1/characters/batch`,
SCRUM-245) can fetch them by URL during testing. Everything is fictional and
machine-generated.

## Files

| Path | What it is |
|---|---|
| `pdfs/*.pdf` | The 100 documents, 1 to 85 pages, 3 KB to 170 KB |
| `urls.txt` | The 100 public URLs, one per line |
| `manifest.json` | Characters with their documents, canonical facts, and retrieval questions, each tagged with the one file that holds the answer |
| `batch_request.json` | Ready-to-POST body: 73 items, 100 documents in total (exactly the per-batch document cap) |
| `batch_request_10.json` | The first 10 items. Includes a 1-, 2-, 3- and 10-document character |
| `batch_request_multi.json` | Only the 15 characters that own more than one PDF (42 documents) |
| `generate_pdfs.py` | Deterministic generator (seed 20260905). Needs `reportlab`; uses `pymupdf` for self-verification when present |

URL pattern:

```
https://raw.githubusercontent.com/karen93shieh/estuary-batch-test-pdfs/main/pdfs/001-torvald-nightingale.pdf
https://raw.githubusercontent.com/karen93shieh/estuary-batch-test-pdfs/main/pdfs/004-bram-marchetti-v3-chronicle-1.pdf
```

## Characters and how many PDFs each owns

| PDFs per character | Characters | Volumes |
|---|---|---|
| 1 | 58 | one complete dossier (`NNN-first-last.pdf`) |
| 2 | 10 | dossier + relationships |
| 3 | 4 | dossier + relationships + one chronicle |
| 10 | 1 | dossier + relationships + eight chronicles (the per-item cap) |

Multi-PDF files are named `NNN-first-last-vK-kind.pdf` where `NNN` is the
character number and `K` the volume number. Items 1 to 4 of every request body
are fixed at 1, 2, 3 and 10 documents; the other multi-PDF characters are
spread through the list.

Each item's `customId` is `test-npc-NNN-firstname`, matching the PDF numbers.

## Facts are split across volumes on purpose

For a multi-PDF character, each volume holds facts that appear in no other
volume, so you can tell which documents were actually ingested and linked:

| Volume | Facts only found there | Example question |
|---|---|---|
| dossier | favourite dish, lucky number, pet | "What is Bram's lucky number?" |
| relationships | workshop password, rival, mentor, friend, secret | "What is the password to Bram's workshop?" |
| chronicle k | what the character calls one possession | "What does Bram call the sextant?" |

A single-PDF character has all of these in the one file (minus possessions).
`manifest.json` lists every question under `characters[].retrieval_checks`
with `expect` and `source_file`. Ask the created character and compare.

Every dossier also has an overview, background, personality and speech notes,
a "Verified facts" table and sample dialogue. Characters 7, 22, 37, 52 and 67
carry a Chinese and a Japanese summary paragraph in their first volume.

## Sizes

688 pages and 1.5 MB in total. The largest single file is 85 pages and 170 KB.
Everything is well under the fetcher caps (50 MB per file, 500 pages per
document).

## Submitting a batch

```bash
curl -X POST "$ESTUARY_API/api/v1/characters/batch" \
  -H "X-API-Key: $ESTUARY_API_KEY" \
  -H "X-Org-Id: $ESTUARY_ORG_ID" \
  -H "Content-Type: application/json" \
  --data @batch_request_10.json
```

Then poll `GET /api/v1/characters/batch/{batchId}` until every item's
documents report `ready` or `failed`.

## Regenerating

```bash
pip install reportlab pymupdf
python generate_pdfs.py . https://raw.githubusercontent.com/karen93shieh/estuary-batch-test-pdfs/main/pdfs
```

The seed is fixed, so the output is stable apart from PDF creation
timestamps. Keep `.gitattributes`: reportlab writes uncompressed PDFs that git
would otherwise treat as text and CRLF-convert on Windows checkouts.
