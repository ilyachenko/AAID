---
layout: default
title: Maintaining CLAUDE.md and CHANGELOG.md
---

# Maintaining CLAUDE.md and CHANGELOG.md: Best Practices Guide

This guide defines **best practices** for maintaining `CLAUDE.md` and related documentation.
It is designed for team-wide consistency and long-term clarity.

---

## What are CLAUDE.md and CHANGELOG.md?

* **CLAUDE.md**: A living documentation file placed in the repository. It explains how Claude (the AI assistant) should interact with the project: coding style rules, workflows, commands, risky areas, and domain context. Its purpose is to guide AI and contributors in day-to-day tasks.

* **CHANGELOG.md**: A history log of notable changes across releases or versions. It follows a structured format (e.g., Keep a Changelog) and records additions, changes, fixes, and removals. Its purpose is to inform humans about project evolution, not to guide AI behavior.

---

## 1. Principles

* **Docs-as-Code**: Treat documentation the same way we treat code (version control, PRs, reviews, CI).
* **Single Source of Truth**: `CLAUDE.md` holds knowledge needed for Claude to be effective.
* **High-Signal Only**: Capture key conventions, rules, workflows—not routine fixes. For JavaScript projects, examples include documenting ESLint/Prettier rules, Node.js script usage (e.g., `npm run build`, `npm run lint`), directory structure changes (like moving `src/components`), or conventions for async/await error handling.
* **Shared Responsibility**: Everyone updates docs when making relevant changes, but each doc has an **owner** accountable for quality.

---

## 2. When to Update `CLAUDE.md`

Update **in the same PR** if any of these apply:

1. **Tools & Workflows**

   * New scripts, CLI commands, build/test tasks.
   * Example (JavaScript): adding `npm run build`, `npm run lint`, or `npm test` tasks, or introducing new Node.js CLI utilities.

2. **Codebase Structure**

   * Moving or renaming directories, entrypoints, or generated files.
   * Example: Document that `src/services/api/` replaced `src/api/`.

3. **Team Conventions**

   * Coding style, lint/test thresholds, PR/review rules.
   *
   * Example (CSS): “Use CSS variables for colors and spacing instead of hardcoded values.”
   * Example (JavaScript): “Always use async/await instead of raw Promises for readability.”

4. **Domain Knowledge**

   * Business rules, invariants, constraints Claude must assume.
   * Example: “All dates are stored in UTC, never local time.”

5. **Risk Warnings**

   * Security-sensitive files, generated code to avoid, common pitfalls.
   * Example: “Do not edit `schema.generated.ts`; update `schema.yaml` instead.”

If none of these apply → use commits and `CHANGELOG.md`, not `CLAUDE.md`.

---

## 3. Documentation Cadence

* **Event-driven first**: Update `CLAUDE.md` with each relevant change.
* **Scheduled audits**: Light monthly or quarterly sweep to prune stale info and confirm accuracy.
* **Version alignment**:

  * For release-based projects: branch docs with releases (`v1.0-docs`).
  * For continuous projects: keep one `main` doc branch, archive outdated info.

---

## 4. Process Guardrails

### PR Template Checklist

```markdown
- [ ] This PR affects Claude’s behavior → updated CLAUDE.md (or marked N/A).
```

### CODEOWNERS

```text
/CLAUDE.md   @team-ml-enablement
```

Ensures proper review.

### CI Automation

* Lint docs for grammar, broken links, headings.
* Optional guard: if `tools/` or `scripts/` change without `CLAUDE.md` change, fail CI.

---

### 5. Writing Guidelines

* Be **clear, concise, and active**.
* Use **headings, lists, and examples**.
* Always explain the **why**, not just the what.

### Example Structure

```markdown
# CLAUDE.md — Assistant Usage Guide

## 1. Project Overview & Context
- What the project is, domain terms, critical invariants.

## 2. Do / Don’t Checklist
- ✅ Use `make test` before PRs  
- ❌ Don’t edit generated files

## 3. Tools & Commands
- `make build` → compile  
- `make verify-api` → schema checks

## 4. Code & Review Norms
- Follow PEP8.  
- Use Conventional Commits (`feat:`, `fix:`, `docs:`).  
- At least 1 reviewer required.

## 5. Risk Areas
- Security configs live in `/secrets/` (never commit secrets).  
- Avoid editing `schema.generated.ts`.

## 6. Contact & Maintenance
- Maintainer: @team-docs  
- Last reviewed: 2025-09-01
```

---

## 6. Changelog & Commit Messages

* **Commits**: Use [Conventional Commits](https://www.conventionalcommits.org).
  Example:

  ```
  docs(claude): add new verify-api command
  ```
* **CHANGELOG.md**: Follow [Keep a Changelog](https://keepachangelog.com) format.
  Example:

  ```markdown
  ## [Unreleased]
  ### Added
  - `make verify-api` command (documented in CLAUDE.md)
  ```

---

## 7. Continuous Improvement

* **Audits**: Quarterly check for outdated content.
* **Feedback loop**: Team can open “doc update” issues when something feels wrong.
* **Retrospective review**: Ask “Does CLAUDE.md reflect reality?” during sprint retros.

---

## 8. Sample Workflow

1. Add new script `tools/lint.sh`.
2. Update `CLAUDE.md`:

   ```diff
   + Run `make lint` to check code style.
   ```
3. Commit:

   ```
   docs(claude): document new lint script
   ```
4. Update `CHANGELOG.md` under “Added”.
5. Submit PR with checklist checked.
6. CI verifies docs updated.
7. CODEOWNER reviews.
8. Merge → docs stay accurate.

---

## Summary

* **Update CLAUDE.md** when workflows, conventions, or assistant-facing context changes.
* **Use commits + changelog** for day-to-day fixes and version notes.
* **Automate + enforce** with PR templates, CODEOWNERS, and CI.
* **Keep docs short, clear, and alive** through event-driven updates and periodic sweeps.
