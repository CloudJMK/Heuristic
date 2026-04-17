# Classification Rubric (Builder)

## Step 1: Content-type detection
- `text_note`: markdown/txt/html cleaned text
- `pdf_doc`: pdf with extractable text
- `media_asset`: image/audio/video
- `dataset`: csv/parquet/jsonl/sql dump
- `code_repo`: git URL/archive/source tree
- `binary_other`: non-text executables or archives

## Step 2: Value scoring (0-100)
- Source authority (0-25)
- Relevance to active projects/areas (0-25)
- Novelty vs existing vault content (0-20)
- Actionability/reference durability (0-20)
- Provenance completeness (0-10)

## Step 3: PARA inference hints
- Active deliverable support + due date => Projects
- Ongoing responsibility/SOP/operational playbook => Areas
- Evergreen reference/research background => Resources
- Inactive/superseded/historical => Archives

## Step 4: Confidence scoring
- Base confidence from classifier probability.
- Subtract penalties:
  - Missing metadata: -15
  - Ambiguous domain mapping: -10
  - Duplicate conflict unresolved: -20
- If final confidence <85, require human review.

## Step 5: Regression corpus gate (anti-drift)
- Maintain a versioned corpus of labeled inbox items across all major content types.
- Require:
  - macro-F1 >= previous baseline - 0.02
  - PARA placement accuracy >= 95%
  - false-autoplace rate (wrong with confidence >=85) <= 1%
- If any gate fails, freeze autoplace and send all items to review queue until corrected.
