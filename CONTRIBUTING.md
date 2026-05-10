# Contributing to Documents Repository

This repository stores all formal project documentation such as SRS, SDS, diagrams, and meeting notes.

---

## Branch Strategy

- `main` → Protected, final approved version
- `draft/<document-name>` → Work in progress
- `fix/<issue-number>` → Review corrections

Example:
- `draft/srs-chapter-2`
- `fix/12`

---

## Workflow

1. Pull latest `main`
2. Create a draft branch
3. Make changes
4. Commit regularly
5. Push branch
6. Open Pull Request
7. Request review (minimum 1 approval required)
8. Merge after approval

---

## Commit Messages

Use Conventional Commits:

- `docs:` Add or update documentation
- `feat:` Add new section or content
- `fix:` Correct errors or review feedback
- `refactor:` Restructure without changing meaning
- `style:` Formatting only
- `chore:` Maintenance tasks

Examples:
- `docs: update SRS section 3.2 functional requirements`
- `feat: add use case diagram for booking flow`
- `fix: correct typo in approval section`

---

## File Naming

- `srs-chapter-2.md`
- `GarageGo_SRS_v1.0.pdf`
- `2026-01-20_standup.md`

---

## Pull Request Rules

- At least 1 approval required
- Do not merge your own PR
- Address all review comments before merging
