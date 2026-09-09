# What is a Harness?

**Prompt:**
Let's describe the "harness" that the content analysis process is able to identify important information, make connections across the plans, etc. I am trying to explain our work to a colleague, and need help with a bit more articulation, how we are using subagents, skills, etc. and things Like the OCR core; if possible to create a "slide deck" on a web page

**Response:**
The harness we created is a local-first execution and assurance framework around the OCR engine. It
is broader than a test suite: it governs how documents enter the engine, how work is identified, how
outputs become durable, how interrupted jobs resume, and how every result can later be evaluated and
audited.

Its flow is roughly:

- Runtime diagnostics
- Read-only preflight
- Validated configuration + deterministic work key
- Native extraction / OCR pipeline
- Private artifact staging
- Checksum-verified publication + commit marker
- Manifest + optional SQLite persistence
- Progress, provenance, evaluation, and resume evidence

The main pieces are:

- Runtime harness: ocr-cli doctor resolves PDFium and Tesseract, checks versions and language packs,
and reports actionable failures without opening a document.
- Preflight harness: inventories a batch, checks source limits and runtime readiness, and reports
capabilities before processing. It does not invent duration estimates before local observations
exist.
- Deterministic execution harness: every reusable unit of work is keyed by source hash, validated
configuration, engine version, output schema, processing-contract version, and artifact selection.
Volatile details such as timestamps and run IDs cannot alter content artifacts.
- Transactional artifact harness: outputs are written to private staging, synced, published without
overwriting conflicting content, and finalized with a checksum-bearing commit marker. A manifest
success record is written only after this boundary is complete.
- Recovery and resume harness: interrupted or corrupted runs are detected through commit markers,
artifact inventories, and SHA-256 checksums. Matching work is skipped safely; incomplete or
conflicting work is retried or rejected.
- Resource and failure harness: source size, page count, rendered pixels, region count, artifact
size, and subprocess timeouts are enforced. Failures remain scoped as fatal, document, page,
region, or artifact failures rather than being misreported as empty content.
- Persistence harness: optional SQLite storage uses ordered transactional migrations, schema
validation, transactional document replacement, provenance event storage, and conservative
compatibility handling.
- Verification harness: Rust tests exercise deterministic output, injected failures, migration
rollback, timeout behavior, staged-publication recovery, corruption detection, and resume
semantics without processing real PDFs. The recorded F2 gate currently includes 327 Rust test
executions across its principal suites, plus clippy, formatting, Python parsing, schema parsing,
and source-network scans.
- Controlled runtime harness: real PDFium and Tesseract validation is kept separate from pure
verification and requires explicit authorization. Previous controlled evidence is recorded, but
normal tests do not open PDFs.
- Governed evaluation harness: committed JSON schemas define corpus governance, coverage dimensions,
visual adjudication, and measures such as CER, WER, reading order, layout F1, table structure,
visual preservation, silent loss, determinism, and recovery. Protected documents and results stay
in ignored local directories.
- Provenance harness: observational IEEE 2890-adapted schemas capture Data Actors, authorization,
custody, derivation, publication, and failures through a hash-chained local ledger and SQLite. It
supports disabled and observe; enforcement is deliberately unavailable.

The defining principle is that material content must not silently disappear. If the engine cannot
extract, preserve, classify, or publish something, the harness requires that outcome to remain
visible as failure or review evidence.
