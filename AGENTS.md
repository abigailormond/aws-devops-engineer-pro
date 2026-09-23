# AWS DevOps Engineer Professional Study Repo

## Purpose

This repository is a personal study workspace for the AWS Certified DevOps Engineer - Professional (DOP-C02) exam, planned for December 2026. The course material is primarily Adrian Cantrill's AWS DevOps Engineer Professional course.

## Repository Map

- `course-outline.md` is the source of truth for the course's contents: the full list of sections and videos, in course order. Any change to what videos exist or how they are titled/grouped should start here.
- `schedule.md` takes the videos from `course-outline.md` and organizes them into a day-by-day study schedule. It tracks video completion with Markdown checkboxes, groups videos by week and day, and records planned viewing time. It does not introduce new videos or reorder the course; it schedules the outline's existing sequence.
- `notes/` contains personal notes taken from the course videos. Notes are organized as quick-reference outlines rather than polished documentation.
- Note files use a two-digit numeric prefix and lowercase filenames, such as `notes/00 iam accounts and organizations.md` and `notes/01 cloudformation.md`. Preserve the existing numeric sequence when adding or renaming course sections.
- `.vscode/settings.json` contains local editor appearance settings; do not change it unless explicitly asked.

## Study Schedule Rules

When changing the schedule:

- Treat `course-outline.md` as authoritative for video existence, titles, and course order; the schedule should reflect it, not diverge from it.
- Preserve the exact course-video sequence unless the user explicitly requests a schedule revision.
- Keep each video assigned wholly to one study day.
- Keep each week at or below its stated five-hour maximum unless the user explicitly approves an exception.
- Preserve completion checkboxes and existing markers such as `!` (revisit), `NEEDS UPDATE`, `DOP-C02`, `DEMO`, `REFRESHER`, `MINIPROJECT`, and access labels such as `ASSOCIATESHARED` and `SHAREDALL`.
- Preserve the existing week/day headings and duration format. Recalculate day and week totals when adding, removing, or moving videos.
- Keep the revision notes at the end accurate after any schedule change.

## Notes Conventions

- Keep notes concise, factual, and scannable with headings and nested bullets.
- Use Markdown `##` headings for video or major lesson sections; keep supporting subtopics as plain text or nested list items unless the existing note structure calls for another level.
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
