# RLM-MEM 🧠

**RLM-MEM** is an [RLM](https://arxiv.org/html/2501.11223v1)-inspired memory operating layer for coding agents. The idea is simple, Agents use tools to sarfch memory instead of reading entire files.,  allowing a much larger context of memory available to the agent that remains consistent.

Inspired by the Recursive Language Model direction, this package turns that idea into practical local agent infrastructure: scoped memory, policy controls, and reliable retrieval workflows.

## Why It Exists

Agents usually fail in three ways:

- they forget important project context
- they leak context across scopes
- they rely on brittle, ad-hoc memory files

RLM-MEM fixes that with explicit contracts:

- layered memory scopes with policy boundaries
- structured write/recall/reason operations
- safety + validation before persistence
- guard scripts to prevent runtime drift

## What You Get

- **4-layer memory model:** `project_agent`, `project_global`, `user_agent`, `user_global`
- **Core ops:** `RememberOperation`, `RecallOperation`, `ReasonOperation`
- **Layered storage:** JSONL-based memory with deterministic retrieval planning
- **Safety:** redaction + write-scope enforcement
- **Migration support:** legacy chunk-store migration tooling
- **Compatibility overlays:** optional personality/slider/LIVEHUD assets for hosts that use them
- **Soul + user controls:** switch agent operating style and preserve user-specific preferences
- **Disk history backups:** timestamped `.bak` snapshots for safe rollback of soul/user changes
- **Simple status signals:** operation-level success/confidence plus cache telemetry for fast health checks

## Prerequisites

- Python 3.10+ available as `python`
- Git
- A cloned copy of this repo

Dependency reality:

- No mandatory third-party package is required for core runtime usage.
- Optional extras:
  - `tiktoken` for more accurate token counting in chunking
  - `PyYAML` for richer YAML parsing (there is a built-in fallback parser)

## Install / Setup

Clone and initialize:

```powershell
git clone https://github.com/zenchantlive/meridian.git
cd meridian
```

Quick setup checks from repo root:

```powershell
$env:PYTHONPATH=(Resolve-Path RLM-MEM).Path
python -c "from brain.scripts import LayeredMemoryStore, LayeredChunkStoreAdapter, MemoryPolicy; print('OK')"
python RLM-MEM/scripts/check_no_runtime_duplicates.py
python RLM-MEM/scripts/check_skill_only_integrity.py
```

If these pass, RLM-MEM is ready to use.

Optional dependency install:

```powershell
python -m pip install tiktoken pyyaml
```

## One-Liner For Another Agent 🚀

```text
Install and set up RLM-MEM from https://github.com/zenchantlive/meridian.git. Use only `RLM-MEM/` as source of truth, read `RLM-MEM/SKILL.md`, run import + guard checks, patch only `RLM-MEM/**`, and run troubleshooting tests only if behavior breaks.
```

## Soul + User Personalization

RLM-MEM supports two practical customization layers:

- `ACTIVE_SOUL.md`
  - the active operating constitution for the agent (tone, rigor, behavior constraints)
  - default is whatever file is currently copied into `ACTIVE_SOUL.md` (currently the Linus profile)
- `USER.md`
  - user-specific preferences and local working style

You can switch souls any time; this does not require runtime refactors.

Manage these with built-in scripts:

```powershell
python RLM-MEM/scripts/manage_soul.py list
python RLM-MEM/scripts/manage_soul.py switch linus
python RLM-MEM/scripts/manage_soul.py update linus --file .\my_soul.md
python RLM-MEM/scripts/manage_user.py --file .\my_user_prefs.md
```

Why this is useful:

- you can switch behavior profiles quickly without editing runtime code
- user preferences stay explicit and versionable
- changes are safer because updates are backed up automatically

## Backup History (Disk Snapshots)

Whenever `manage_soul.py update` or `manage_user.py` writes changes, RLM-MEM creates timestamped backups in:

- `RLM-MEM/user_backups/`

Backup filename pattern examples:

- `USER.md.20260211153045.bak`
- `linus_soul.md.20260211153110.bak`

This gives you local disk history for rollback and auditability.
If `user_backups/` is currently empty, no managed update has been run yet.

## Simple Status Model (Core Runtime)

RLM-MEM status is meant to be lightweight and practical, not a forced dashboard format.

Core status signals already available:

- `RememberOperation.remember(...)` returns:
  - `success`
  - `chunks_created`
  - `total_tokens`
  - `error` (when applicable)
- `RecallOperation.recall(...)` returns:
  - `confidence`
  - `source_chunks`
  - `iterations_used`
- `ReasonOperation.reason(...)` returns:
  - `confidence`
  - `insights`
  - `contradictions`
- `CacheManager.stats()` / `CacheManager.telemetry()` returns:
  - `memory_hit_rate`
  - `miss_rate`
  - hit/miss/eviction counters

This gives you machine-readable status that works across hosts without enforcing a universal response UI.

Compatibility note:
- Legacy LIVEHUD/parsing assets remain in compatibility code paths, but they are not the core status contract.

## Canonical Layout

- `RLM-MEM/SKILL.md` -> agent operator manual
- `RLM-MEM/FRESH_AGENT_CHECKLIST.md` -> deep validation runbook
- `RLM-MEM/brain/scripts/` -> runtime code + tests
- `RLM-MEM/scripts/` -> guard/setup/management scripts
- `RLM-MEM/brain/` -> compatibility assets and related docs

## Troubleshooting (Only When Needed)

If setup passes but behavior is wrong, run:

```powershell
$env:PYTHONPATH=(Resolve-Path RLM-MEM).Path
python -m unittest brain.scripts.test_memory_schema brain.scripts.test_memory_policy brain.scripts.test_memory_layers brain.scripts.test_memory_safety brain.scripts.test_layered_writer -v
python -m unittest brain.scripts.test_remember_layered_integration brain.scripts.test_recall_layered_integration brain.scripts.test_reason_layered_integration brain.scripts.test_multi_agent_isolation -v
python -m unittest brain.scripts.test_final_integration -v
```

Common causes:

- `ImportError: brain.scripts` -> `PYTHONPATH` not set to `RLM-MEM`
- duplicate runtime guard failure -> conflicting runtime file names outside canonical path
- integrity guard failure -> legacy authoritative paths were reintroduced

## For Maintainers

Run the full test matrix before release PRs or runtime internals changes.
This keeps the distribution stable for fresh agents and avoids silent memory regressions.

## References

- RLM paper: `https://arxiv.org/html/2501.11223v1`
- Operator guide: `RLM-MEM/SKILL.md`
