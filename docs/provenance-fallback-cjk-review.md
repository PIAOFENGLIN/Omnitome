# Provenance fallback and CJK sentence splitting review

## Behavior

Failed, skipped, or empty/invalid Signal B arbitration now produces `method: unverified`, `confidence: none`, and `grounded: false`. Candidate chunk IDs remain available for diagnostics, but no clickable sentence segments are generated. The reader distinguishes incomplete verification from a paragraph with no traced source. Existing fallback records are normalized when read; no database migration is needed. Successful existing C-only, C+A, and B decisions are unchanged.

Frontend and backend sentence splitting now support Korean sentence starts, Chinese/Japanese full-width terminal punctuation without intervening whitespace, and closing quotation marks. English abbreviation, initial, and decimal handling is retained.

## Verification

- `npm --prefix frontend ci --ignore-scripts --no-audit --no-fund`
- `npm run test:provenance`: 24 focused tests passed. These execute actual modules with mocked AI, embedding and repository boundaries, including persistence/API formatting and historical fallback normalization. The frontend splitter is transpiled from its actual TypeScript source and checked against the backend.
- `npm --prefix frontend run build`: TypeScript and production build passed; Vite reports a bundle size advisory.
- `git diff --check`: passed.

Tested on Node 24.22.1. The project recommends Node 22. Root dependency installation did not complete successfully in this environment, so the existing database/Python integration suite was not run. Windows desktop, GPU inference, live provider failure, and visual interaction require follow-up validation before merge.

## Review limits and follow-up

This change prevents a verification failure from being presented as success; it does not establish that similarity-based successful decisions prove semantic entailment. Empty evidence and unavailable arbitration have distinct fallback reason codes, with conservative shared reader treatment.

Stored successful CJK provenance segments created using the old splitter are not automatically rewritten. Force re-verification of affected chapters is required to align their sentence indices with the updated splitter. Sentence splitting remains heuristic rather than a complete language parser.

Before merging, exercise a Windows chapter with a failing provider, successful retry using force re-verification, an older fallback record, and mixed Korean/Chinese/Japanese text. Verify source selection and text-to-speech highlighting against the original passages.
