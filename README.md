# Estuary batch character test PDFs

100 synthetic character dossiers, hosted here so the Estuary batch character
creation endpoint (`POST /api/v1/characters/batch`, SCRUM-245) can fetch them by
URL during testing. Everything is fictional and machine-generated.

## Files

| Path | What it is |
|---|---|
| `pdfs/NNN-first-last.pdf` | One dossier per character, 2 to 85 pages, 4 KB to 171 KB |
| `urls.txt` | The 100 public URLs, one per line |
| `manifest.json` | Per document: URL, size, page count, the character's canonical facts, and five retrieval questions with expected answers |
| `batch_request.json` | Ready-to-POST body: 100 items, one document each (exactly the per-batch document cap) |
| `batch_request_10.json` | The first 10 items, for a quick smoke test |
| `generate_pdfs.py` | Deterministic generator (seed 20260905). Needs `reportlab`; uses `pymupdf` for self-verification when present |

URL pattern:

```
https://raw.githubusercontent.com/karen93shieh/estuary-batch-test-pdfs/main/pdfs/001-torvald-ashgrove.pdf
```

## What is in each PDF

Every dossier has the same sections: overview, background, personality and
speech, a "Verified facts" table, a secret, sample dialogue, and (for longer
documents) a chronicle of templated diary entries used as padding. Documents
`007`, `027`, `047`, `067` and `087` also carry a Chinese and a Japanese summary
paragraph.

The verified facts are unique per character and are what `manifest.json`
lists under `retrieval_checks`:

- favourite dish
- lucky number (three digits)
- workshop password (two hyphenated words, for example `amber-lantern`)
- pet name and species
- mentor, rival and closest friend (other characters in the same set)

Ask the created character one of those questions and compare with `expect`.

## Size distribution

| Bucket | Documents | Pages each |
|---|---|---|
| short | 55 | 2 to 4 |
| medium | 30 | 6 to 11 |
| long | 12 | 14 to 35 |
| very long | 3 | 57 to 85 |

Total: 911 pages, 1.9 MB. All well under the fetcher caps (50 MB per file,
500 pages per document).

## Submitting a batch

```bash
curl -X POST "$ESTUARY_API/api/v1/characters/batch" \
  -H "X-API-Key: $ESTUARY_API_KEY" \
  -H "X-Org-Id: $ESTUARY_ORG_ID" \
  -H "Content-Type: application/json" \
  --data @batch_request_10.json
```

Then poll `GET /api/v1/characters/batch/{batchId}` until every item's
documents report `ready` or `failed`. Each item's `customId` is
`test-npc-NNN-firstname`, matching the PDF number.

## Regenerating

```bash
pip install reportlab pymupdf
python generate_pdfs.py . https://raw.githubusercontent.com/karen93shieh/estuary-batch-test-pdfs/main/pdfs
```

The seed is fixed, so the output is byte-for-byte stable apart from PDF
creation timestamps.
