# AWS DevOps Engineer Professional Study Repo

## Purpose

This repository is a personal study workspace for the AWS Certified DevOps Engineer - Professional (DOP-C02) exam, planned for December 2026. The course material is primarily Adrian Cantrill's AWS DevOps Engineer Professional course.

## Repository Map

- `Study Guide/Study Schedule.md` is the source of truth for the study plan. It tracks video completion with Markdown checkboxes, groups videos by week and day, and records planned viewing time.
- `Notes/` contains personal notes taken from the course videos. Notes are organized as quick-reference outlines rather than polished documentation.
- `.vscode/settings.json` contains local editor appearance settings; do not change it unless explicitly asked.

## Study Schedule Rules

When changing the schedule:

- Preserve the exact course-video sequence unless the user explicitly requests a schedule revision.
- Keep each video assigned wholly to one study day.
- Keep each week at or below its stated five-hour maximum unless the user explicitly approves an exception.
- Preserve completion checkboxes and existing markers such as `REVIEW`, `NEEDS UPDATE`, `DOP-C02`, `DEMO`, `REFRESHER`, `MINIPROJECT`, and access labels such as `ASSOCIATESHARED` and `SHAREDALL`.
- Preserve the existing week/day headings and duration format. Recalculate day and week totals when adding, removing, or moving videos.
- Keep the revision notes at the end accurate after any schedule change.

## Notes Conventions

- Keep notes concise, factual, and scannable with headings and nested bullets.
- Preserve the author's terminology and emphasis. Fix typos or restructure notes only when asked, or when necessary to prevent a technical misunderstanding.
- Distinguish course facts from personal reminders, open questions, and follow-up items. Do not silently turn uncertainty into a definitive AWS claim.
- Retain useful exam distinctions, limits, policy evaluation rules, service relationships, and CLI or console workflow details.
- Use fenced code blocks for substantial YAML, JSON, policy documents, or commands. Keep inline AWS identifiers such as ARNs, actions, and intrinsic functions in backticks when adding new material.
- Preserve existing Obsidian-style image embeds and do not invent replacement assets when an embedded image is unavailable.
- Avoid storing credentials, access keys, account identifiers, or other secrets in notes.

## Editing and Validation

- Make focused edits and preserve unrelated user changes in the working tree.
- Prefer the existing Markdown structure and naming conventions; avoid broad formatting churn.
- Do not add application code, dependencies, or build tooling unless explicitly requested.
- Before finalizing a schedule edit, verify checkbox counts, ordering, day/week totals, and the five-hour weekly cap.
- Before finalizing note edits, check heading hierarchy, list indentation, code-block fences, and that technical corrections are clearly grounded in the course or current AWS documentation.
- There is no application test suite. For Markdown-only changes, use `git diff --check` and a targeted inspection of the changed section as validation.
