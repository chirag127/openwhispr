# Issue expansion: 2026-11-17

This pass focused on finding new, non-duplicate, evidence-backed issues rather than opening pull requests.

## Method

1. **Code search.** Used `rg` across `src/`, `main.js`, `preload.js`, `scripts/`, and the mobile app for `TODO`, `FIXME`, `unsupported`, `not implemented`, `not supported`, platform gates, and feature-specific code paths.
2. **History search.** Searched open and closed issues for every candidate (`playlist`, `PDF`, `per-app`, `snippet variable`, `bookmark`, `chapter`, `Notion`, `Obsidian`, `Todoist`, `Jira`, `create calendar event`, `Wayland screen`, `encrypt`, `portable`, `benchmark`, `watch folder`, `accessibility`, `issue template`).
3. **Web research.** Checked GitHub's issue/PR guidance and competitor feature sets (Wispr Flow, Superwhisper, MacWhisper, Spokenly) for patterns that are missing from OpenWhispr.
4. **Template compliance.** New issues use the repository feature-request structure. Issue #3247 was rewritten to the bug-report structure (description, reproduction, expected vs actual, OS/version, additional context).

## New issues

| #    | Title                                                                    | Category             | Closest existing issue     |
| ---- | ------------------------------------------------------------------------ | -------------------- | -------------------------- |
| 3271 | URL import: support YouTube playlists and batch URL queues               | Input / batch        | #1633, #1914, #3227        |
| 3272 | Import PDF, DOCX, and HTML documents as searchable notes                 | Input / knowledge    | #1633, #2232, #2556        |
| 3273 | Add per-app dictation profiles                                           | Workflow / context   | #1047, #304, #3084         |
| 3275 | Snippet variables: date, time, clipboard, selection, cursor              | Workflow / templates | #756, #1704                |
| 3276 | Meeting bookmarks and auto-generated chapters                            | Meeting / navigation | #3084, #3085, #2353, #2974 |
| 3277 | Add Notion and Obsidian export connectors                                | Integration / export | #2609, #2610, #1633        |
| 3278 | Add Todoist and Jira connectors for action items                         | Integration / tasks  | #2609, #2610               |
| 3279 | Create calendar events by voice with an approval card                    | Assistant / calendar | #3018, #3031, #2636        |
| 3280 | Wayland: screen-context capture through xdg-desktop-portal               | Platform parity      | #1958, #3085, #2354        |
| 3281 | Optional at-rest encryption for local notes/transcripts                  | Privacy / security   | #532, #2579                |
| 3282 | Windows: portable build with data beside the executable                  | Windows / packaging  | #2054, #1265               |
| 3283 | Hardware-aware local model recommendation and benchmark                  | Setup / performance  | #3182, #3040, #1047, #867  |
| 3284 | Add GitHub issue forms and a pull request template                       | Repo hygiene         | #1047, #2009               |
| 3286 | Watch folders: auto-transcribe new audio/video files                     | Automation / batch   | #1950, #1633, #3271        |
| 3287 | Accessibility: keyboard and screen-reader support for the dictation pill | Accessibility        | #2354, #969                |

## Why these are not duplicates

- The repository has no playlist, live-stream, PDF/document import, watch-folder, per-app profile, snippet-variable, meeting-bookmark/chapter, Notion/Obsidian connector, Todoist/Jira connector, calendar-write, Wayland screen-context, database-encryption, portable-Windows, hardware-benchmark, or dictation-pill accessibility issue.
- Existing issues cover adjacent problems but not these user actions:
  - #1633 imports Markdown/text notes; it does not ingest PDF/DOCX or watch folders.
  - #1958 requests active-window screenshot context; it does not cover the Wayland capture implementation.
  - #532 covers plaintext API keys; it does not cover notes/transcripts at rest.
  - #2609/#2610 cover connector platform work; they do not name Notion, Obsidian, Todoist, or Jira.
  - #3084/#3085 cover speaker memory and meeting screenshots; they do not cover bookmarks or chapters.
  - #1704 covers "paste last transcription"; it does not cover dynamic snippet variables.

## Template and quality rules applied

- One user problem per issue.
- Title names the missing action or failure, not "feature request".
- Evidence includes file:line references.
- Implementation options list at least three approaches with trade-offs.
- Reproduction steps and OS/version are included for the bug-template issue (#3247).
- Every issue includes a dedupe/history section and a checklist.
- No secrets, personal data, or generated filler.

## Web references used

- GitHub Docs, "About issue and pull request templates": https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/about-issue-and-pull-request-templates
- GitHub Docs, "Configuring issue templates": https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/configuring-issue-templates-for-your-repository
- GitHub Docs, "Quickstart for GitHub Issues": https://docs.github.com/en/issues/tracking-your-work-with-issues/learning-about-issues/quickstart
- "How to Write a Useful GitHub Issue": https://helpwanted.dev/blog/how-to-write-a-useful-github-issue
- Competitor feature comparisons (Wispr Flow, Superwhisper, MacWhisper, Spokenly), used only to identify missing workflows, not as evidence of OpenWhispr behavior.

## Pull request status

- #3255, #3256, #3257, #3259: all checks green.
- #3258: quality/typecheck/format/tests pass; platform builds were still running when this note was written.
- Existing PRs remain linked to their originating issues (#3248, #3250, #3252, #3246/#3247, #3253).

## Suggested next steps

1. Triage the new issues by user impact: #3272 (document import), #3273 (per-app profiles), #3281 (encryption), and #3282 (portable Windows) are likely the highest-value.
2. Pick one issue and write a design note before implementation; several options are intentionally listed to leave room for maintainer direction.
3. For #3284, convert the Markdown templates to YAML issue forms and add a PR template only after the maintainers agree on required fields.
