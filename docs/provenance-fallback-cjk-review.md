# Provenance fallback and CJK sentence splitting review

## Behavior

Failed, skipped, or empty/invalid Signal B arbitration now produces `method: unverified`, `confidence: none`, and `grounded: false`. Candidate chunk IDs remain available for diagnostics, but no clickable sentence segments are generated. The reader distinguishes incomplete verification from a paragraph with no traced source. Existing fallback records are normalized when read; no database migration is needed. Successful existing C-only, C+A, and B decisions are unchanged.

Frontend and backend sentence splitting now support Korean sentence starts, Chinese/Japanese full-width terminal punctuation without intervening whitespace, and closing quotation marks. English abbreviation, initial, and decimal handling is retained.

## Verification

- `npm --prefix frontend ci --ignore-scripts --no-audit --no-fund`
- `npm run test:provenance`: 41 focused tests passed. These execute actual modules with mocked AI, embedding and repository boundaries, including persistence/API formatting and historical fallback normalization. The frontend splitter is transpiled from its actual TypeScript source and checked against the backend.
- `npm --prefix frontend run build`: TypeScript and production build passed; Vite reports a bundle size advisory.
- `git diff --check`: passed.

Tested on Node 24.22.1. The project recommends Node 22. Root dependency installation did not complete successfully in this environment, so the existing database/Python integration suite was not run. Windows desktop, GPU inference, live provider failure, and visual interaction require follow-up validation before merge.

## Review limits and follow-up

This change prevents a verification failure from being presented as success; it does not establish that similarity-based successful decisions prove semantic entailment. Empty evidence and unavailable arbitration have distinct fallback reason codes, with conservative shared reader treatment.

Stored successful provenance segments are checked against the legacy splitter on read. If boundaries differ (including CJK and quoted English), their clickable segments are suppressed with `sentence_boundaries_changed` until force re-verification. Unaffected legacy paragraphs retain their links. New records persist `sentence_splitter_version: 2` inside the existing weights metadata, without a schema migration. Missing legacy content also fails closed. No automatic AI calls or database rewrites occur during reads. Force re-verification of affected chapters is required to restore their source links. Sentence splitting remains heuristic rather than a complete language parser.

Before merging, exercise a Windows chapter with a failing provider, successful retry using force re-verification, an older fallback record, and mixed Korean/Chinese/Japanese text. Verify source selection and text-to-speech highlighting against the original passages.

## Second review

- Deduplicate Signal B source IDs before normalizing weights. Reject negative, non-finite, nonnumeric, and all-zero weights rather than presenting them as verified evidence.
- Clear arbitration timeout handles after both success and failure; test actual provider parsing (mock provider responses) for empty evidence and malformed JSON retries.
- Restrict English abbreviation/initial exceptions to periods, so question marks and CJK punctuation cannot be swallowed by those rules.
- Protect old sentence indices in both the provenance API and cached verification responses. Tests cover old Korean, Chinese, Japanese, and quoted English, unchanged English, missing content, and new versioned records.

Remaining limitations: no semantic entailment benchmark, full database integration, Windows installer build, GPU test, or visual reader/TTS test has been performed. A model timeout clears the timer but does not cancel an in-progress provider request; provider-wide cancellation is separate work. The UI currently uses the shared verification-incomplete message for stale boundaries; a dedicated retry action would improve usability.
