## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/14

**Issue title:** Add support for parsing GitHub Actions workflow files to detect CI/CD skills
- I choose this issue because it connects directly to what I do at my current job. I interact with GitHub Actions workflow files regularly so I'm already familiar with the YAML structure and the uses: / run: patterns, but I've always engaged with them as a writer, configuring jobs and steps to get pipelines working. This issue gave me a reason to look at the same files from the other direction: what does a parser actually see when it reads them programmatically, and how do you map that structure to meaningful skill signals? That framing made the scope feel purposeful rather than arbitrary
- The gap it closes also felt real to me: a developer whose most skilled work lives in their CI/CD pipelines is currently invisible to PathReview's indexer. That's not an edge case.

**Tier:** Tier 3

**Problem summary:** 
- The pathreview's ingestion pipeline has no parser for .github/workflows/*.yml files, which means CI/CD skills are silently invisible during indexing. A developer who writes and maintains GitHub Actions workflows, Docker build steps, or deployment pipelines cannot surface those DevOps skills in their profile, because they don't appear in import statements or README text, which are the only sources currently parsed. 
- The missing piece is a workflow parser that reads raw workflow YAML, walks the jobs.<job>.steps tree, and maps uses: action prefixes and run: shell commands to inferred skills like GitHub Actions, Docker, pytest, Kubernetes, and Terraform. 
- A successful fix for this missing feature should be a WorkflowParser added into `ingestion/parsers/`, then wired into `IngestionPipeline.ingest_workflow()` in `ingestion/pipeline.py`. The "workflow" source type should also be registered in `ingestion/chunking/strategy_selector.py` so the resulting text gets chunked and embedded alongside resumes and READMEs. Once in place, any developer whose repo contains workflow files will have their CI/CD and deployment skills accurately reflected in their indexed profile.

**Branch name:** feat/14-support-parsing-GHA-workflow-files

**Setup confirmation:** App runs locally at localhost:5173

**Cohort ledger:** Issue added to cohort ledger


---------------------------------------------------------------------

## Week 8 — Reproduction & solution planning

**Reproduction commit link:** https://github.com/MiaNguyen912/pathreview/commit/2ebfa036f0913fd696fe29df2b96a338532f68bf

**Reproduction summary:** see section "ANALYSIS: Replicate Issue #14 (Detailed Steps)" in [PLAN.md](PLAN.md)

**PLAN.md link:** [PLAN.md](PLAN.md)

**Walkthrough video (recommended):** N/A

**Blockers or open questions:**
- GitHub API credentials needed to fetch actual workflow files (currently using placeholder)
- Need to decide: should IngestionPipeline be instantiated in _run_ingestion_pipeline() or elsewhere?

---------------------------------------------------------------------

## Week 9 — Solution building & PR submission

### Check-in 1 (mid-week)

**Current progress:**
- Phase 1 in [PLAN.md](PLAN.md) is completed: 
    - Created WorkflowParser class with trigger extraction, deploy detection, and skill extraction via `SkillExtractor`.
    - Updated `ingestion/parsers/skill_extractor.py`: added more tools to the TOOLS dict and the override names for some of them (e.g kubectl→Kubernetes, aws→AWS, gcp→GCP)
    - Enhanced unit test suite with 29 test cases covering valid/malformed YAML, skill detection, multiple jobs, realistic CI/CD workflows, and edge cases. All tests passing (29/29).

**Next steps:**
- Phase 2: Add `ingest_workflow()` method to IngestionPipeline (following ingest_resume/ingest_readme pattern). 
- Phase 3: Update StrategySelector to handle "workflow" source_type. 
- Phase 4: Wire API layer to call IngestionPipeline.ingest_workflow(). 
- Phase 5: Integration testing.

**Blockers:**
None

---------------------------------------------------------------------

### Check-in 2 (end of week)

**PR link:** https://github.com/ascherj/pathreview/pull/603

**Branch:** `feat/14-support-parsing-GHA-workflow-files`

**What you built:**
Completed Phases 1–3 of Issue #14: built a `WorkflowParser` that reads GitHub Actions workflow YAML files, extracts CI/CD skills (Docker, Kubernetes, Terraform, pytest, etc.) via `SkillExtractor`, and integrated it into the ingestion pipeline. Added `ingest_workflow()` method to `IngestionPipeline` following the established pattern of `ingest_readme()`, and registered "workflow" source_type in `StrategySelector` to route workflow content to semantic chunking. Workflow files are now parsed, chunked, indexed with skill metadata in ChromaDB, and fully integrated into the ingestion architecture.

**Tests added or updated:**
- `tests/unit/test_workflow_parser.py` (NEW) — 29 unit tests covering YAML parsing, trigger extraction, deploy detection, skill detection, realistic workflows, and edge cases (empty workflows, malformed YAML, multiple jobs)
- Test results: 53 failed, 404 passed (29 new passing tests, no new failures)
    - **Baseline (main branch):**
        - Unit tests: 53 failed, 375 passed
        - Linting: Existing failures in other modules (bias_detector, faithfulness_checker, keyword_search, pii_scrubber, prompt_defense, readme_parser, resume_parser, review_service, skill_extractor, structural_chunker, tech_detector)
        - Type checking: Pre-existing issues in api/, core/, rag/ modules
    - **Feature branch (feat/14-support-parsing-GHA-workflow-files):**
        - Unit tests: 53 failed, 404 passed
        - New tests: 29 passing tests from test_workflow_parser.py
        - Linting: All checks passed for new/modified files (workflow_parser.py, skill_extractor.py)
        - Type checking: All checks passed for new/modified files
        - No new test failures introduced
    - **Conclusion:** Feature branch implementation adds 29 new passing tests without introducing any new failures. All pre-existing test failures remain the same.

**Self-review confirmation:** [x] make check passes (for new/modified files)  [x] make test-unit passes

**Draft PR feedback received from:** N/A

