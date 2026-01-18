# AEL Documentation Gaps Tracking

This document tracks all identified gaps in the AEL public documentation and their resolution status.

## Summary

| Priority | Total | Fixed | Remaining |
|----------|-------|-------|-----------|
| Critical | 5 | 5 | 0 |
| High | 4 | 4 | 0 |
| Medium | 3 | 3 | 0 |
| Low | 2 | 2 | 0 |

---

## Critical Issues

### 1. ✅ Installation via pip is incorrect

**Files:** `installation.md`, `index.md`, `quickstart.md`, `troubleshooting.md`

**Problem:** Documentation says `pip install ael` but AEL is not published to PyPI.

**Fix:** Removed pip install instructions, documented source installation as primary method.

**Status:** [x] Fixed

---

### 2. ✅ CLI setup instructions missing

**Files:** `installation.md`, `cli-reference.md`

**Problem:** After `uv sync`, users did not know how to invoke the CLI.

**Fix:** Added CLI invocation section explaining `uv run ael`, venv activation, and direct path methods.

**Status:** [x] Fixed

---

### 3. ✅ Configuration mode not documented

**Files:** `installation.md`, `config-reference.md`

**Problem:** AEL has "configuration mode" vs "running mode" - was completely undocumented.

**Fix:** Added comprehensive section explaining:
- What is configuration mode
- When AEL enters it (no config found)
- Config tools available
- How to transition to running mode

**Status:** [x] Fixed

---

### 4. ✅ Incomplete ways to run AEL

**Files:** `installation.md`, `quickstart.md`, `cli-reference.md`

**Problem:** Missing valid methods: `uv run ael`, `python -m ael.cli`, Docker Compose.

**Fix:** Documented all valid execution methods including Docker Compose.

**Status:** [x] Fixed

---

### 5. ✅ Tool names may be incorrect

**Files:** `tool-integration.md`, `workflow-authoring.md`, examples

**Problem:** References tools like `http_get`, `fetch/fetch` that may not exist.

**Fix:** Updated to use correct tool names from native_tools server and MCP community servers.

**Status:** [x] Fixed

---

## High Priority Issues

### 6. ✅ Docker image location unclear

**Files:** `installation.md`, `DOCKER.md`

**Problem:** `docker pull ghcr.io/ostanlabs/ael:latest` - unclear if published.

**Fix:** Documented local build as primary method with Docker Compose.

**Status:** [x] Fixed

---

### 7. ✅ Config file search order incorrect

**Files:** `config-reference.md`

**Problem:** Listed `~/.config/ael/config.yaml` but actual is `~/.ael/config.yaml`.

**Fix:** Corrected to actual search order:
1. CLI flag: `--config`
2. Environment: `AEL_CONFIG_PATH`
3. Current directory: `./ael-config.yaml`
4. Home: `~/.ael/config.yaml`

**Status:** [x] Fixed

---

### 8. ✅ Missing REST API guide

**Files:** `cli-reference.md`

**Problem:** No documentation on REST API endpoints, authentication, examples.

**Fix:** Added REST API section to cli-reference.md with endpoints and examples.

**Status:** [x] Fixed

---

### 9. ✅ AEL as MCP server not explained

**Files:** `index.md`

**Problem:** Concept of AEL exposing workflows as MCP tools not explained.

**Fix:** Added architecture diagram and explanation of AEL as MCP server.

**Status:** [x] Fixed

---

## Medium Priority Issues

### 10. ✅ Python version inconsistency

**Files:** Various

**Problem:** Some said 3.11+, others 3.12+. Actual: 3.12.

**Fix:** Standardized to Python 3.12+ across all documentation.

**Status:** [x] Fixed

---

### 11. ✅ Examples may not work

**Files:** `examples/*.md`

**Problem:** Examples referenced tools/patterns that may not match implementation.

**Fix:** Updated all examples to use `uv run ael` and correct tool names.

**Status:** [x] Fixed

---

### 12. ✅ Troubleshooting references pip

**Files:** `troubleshooting.md`

**Problem:** Said "Ensure AEL is installed: `pip install ael`"

**Fix:** Updated to source installation method with uv.

**Status:** [x] Fixed

---

## Low Priority Issues

### 13. ✅ Missing architecture overview

**Problem:** No high-level architecture diagram.

**Fix:** Added ASCII architecture diagram to index.md showing AEL components.

**Status:** [x] Fixed

---

### 14. ✅ Missing conceptual overview

**Problem:** No "How AEL works" section explaining core concepts.

**Fix:** Added key concepts table and explanation to index.md.

**Status:** [x] Fixed

---

## Resolution Log

| Date | Issue # | Action | Commit |
|------|---------|--------|--------|
| 2026-01-18 | 1-14 | Fixed all documentation gaps | TBD |
