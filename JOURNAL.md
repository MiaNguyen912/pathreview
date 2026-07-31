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

**PR link:** [link to your submitted pull request]

**Branch:** [the branch name you worked on, e.g. `fix/123-short-description`]

**What you built:**
[1–3 sentences summarizing what your fix does and how it works]

**Tests added or updated:**
[Which test files did you touch? What do they cover?]

**Self-review confirmation:** [ ] make check passes  [ ] make test-unit passes

**Draft PR feedback received from:** [name or Slack handle, or "none"]



