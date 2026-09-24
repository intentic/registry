# registry

The official intentic extension registry: a git repository of commit-pinned pointers that sandboxes browse and install from and that intentic.dev renders as its extension gallery.

```mermaid
flowchart LR
    topic["Repos tagged<br/>intentic-extension"] --> scan["scan.yml<br/>nightly"]
    scan -->|"one PR per new extension"| market(["marketplace.json"])
    scan -->|"stars, last push"| facts["registry.generated.json"]
    admission["extension-admission.yml<br/>audit, Trivy, gate"] --> market
    market --> sandbox["Sandbox<br/>Extensions tab, installs"]
    market --> site["intentic.dev/extensions"]
    facts --> site
```

- `.claude-plugin/marketplace.json` is Claude Code's plugin-marketplace format plus intentic's `kind` and `trust` fields. Each entry pins a source repository and a full commit sha, the exact bytes an install clones. It changes only through reviewed pull requests.
- `.claude-plugin/registry.generated.json` holds facts read off GitHub (stars, last push, `scannedAt`). The scan commits it straight to the default branch, so a refresh never conflicts with a review.
- Discovery never lists anything: `scan.yml` runs a pinned `@intentic/registry-scan` and opens a pull request per newly tagged repository.
- `extension-admission.yml` gates every change to `marketplace.json`. It reads the candidate file through the API without checking out the pull request, audits the diff, scans the pinned source with Trivy in a job without secrets, and asks the intentic security gate about each changed sha. Extension code is never run.
- Trust is `listed`, `verified` or `blocked`. A blocked entry stays in the file with its reason so installed sandboxes can be warned.
- The schema and the two file paths live in `@intentic/registry` in the intentic repository.

## Key files

- [.claude-plugin/marketplace.json](.claude-plugin/marketplace.json) — the curated listing.
- [.claude-plugin/registry.generated.json](.claude-plugin/registry.generated.json) — bot-written facts per entry.
- [.github/workflows/scan.yml](.github/workflows/scan.yml) — nightly discovery and fact refresh.
- [.github/workflows/extension-admission.yml](.github/workflows/extension-admission.yml) — the admission gate for listing changes.
