## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/14

**Issue title:** Add support for parsing GitHub Actions workflow files to detect CI/CD skills

**Tier:** Tier 3

**Problem summary:** 
- The pathreview's ingestion pipeline has no parser for .github/workflows/*.yml files, which means CI/CD skills are silently invisible during indexing. A developer who writes and maintains GitHub Actions workflows, Docker build steps, or deployment pipelines cannot surface those DevOps skills in their profile, because they don't appear in import statements or README text, which are the only sources currently parsed. 
- The missing piece is a workflow parser that reads raw workflow YAML, walks the jobs.<job>.steps tree, and maps uses: action prefixes and run: shell commands to inferred skills like GitHub Actions, Docker, pytest, Kubernetes, and Terraform. 
- A successful fix for this missing feature should be a WorkflowParser added into `ingestion/parsers/`, then wired into `IngestionPipeline.ingest_workflow()` in `ingestion/pipeline.py`. The "workflow" source type should also be registered in `ingestion/chunking/strategy_selector.py` so the resulting text gets chunked and embedded alongside resumes and READMEs. Once in place, any developer whose repo contains workflow files will have their CI/CD and deployment skills accurately reflected in their indexed profile.

**Branch name:** feat/14-support-parsing-GHA-workflow-files

**Setup confirmation:** App runs locally at localhost:5173

**Cohort ledger:** Issue added to cohort ledger
