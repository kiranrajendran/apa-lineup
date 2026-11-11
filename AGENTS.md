# AGENTS.md

Standalone, single-file static web pages for APA 9-ball league play. No build system, package manager, tests, linter, or CI. No external dependencies or network calls. Open the HTML file directly in a browser to run/verify.

## Files

- `index.html`: landing page with two buttons linking to the tools below.
- `apa-lineup-generator.html`: **Lineup generator**. Given available players and an APA skill-level cap (default 23), lists every valid 5-player lineup (distinct skill combinations).
- `apa-9ball-scorekeeper.html`: light-themed 9-ball scorekeeper (drag balls to players, per-rack innings, per-player defenses). Its "export" builds a standalone mobile-friendly HTML score sheet inside `exportSheet()`; the export CSS is a separate string from the app CSS.
- `mike-v-k.html`, `scoresheets/`: previously exported score sheets (re-style by hand if the export template changes).

## apa-lineup-generator.html architecture

- CSS is minified into a few very long lines in `<style>`; JS is one `<script>` block, also written as dense one-line functions (some lines are 500+ chars). Use `grep -n`/`cut -c` rather than viewing whole files, and match exact text carefully when editing.
- **Roster** is a hardcoded `const roster = [{name, skill}, ...]` at the top of the script, between the `EDIT YOUR TEAM ROSTER HERE` banner comments. Changing the team = edit this array (git history shows roster edits are the common change). Skills in use are 3-6 but valid levels are 1-9 (`skillLevels`).
- Two modes toggled by `setMode`: `team` (pick named players from roster, tracked in `selectedPlayers` Set of roster indices) and `generic` (per-skill-level counters in `genericCounts`).
- `calculate()` is the central function: reads `#cap`, `#minCap`, `#replay`, builds lineups via `teamLineups` (team) or `genericLineups` (generic), sorts by total desc, updates the summary metrics, warnings, and `#results` innerHTML. Every UI change calls it; there is no framework or state store.
- **Replay** option: one player plays twice (5th match uses a repeat of one of 4 distinct players). Implemented via `combArr(players,4)` plus an extra player in team mode, and `countComb(c,4)` plus an extra skill in generic mode. Replay entries are tagged by `replayId` / `replaySkills` and rendered with a `REPLAY` tag.
- Lineups are deduplicated by skill combination (`key()`); team mode groups multiple named combos under one skill combo.
- DOM elements are referenced via implicit global IDs (e.g. `cap`, `results`, `available`, `reset`) as well as `getElementById`. Don't rename element IDs without grepping the script.
- User-supplied text is escaped with `esc()` before going into innerHTML; keep doing this for names.

## Gotchas

- `countsFromSkills` initializes only keys 3-6, so a roster skill outside 3-6 would produce `NaN` counts in team mode (see `key`/`total`, which read `c[s]||0` and so mostly mask it, but `c[s]++` on an undefined key yields `NaN`). Extend that object if adding players with other skills.
- Cap/min defaults (23 / 0) are duplicated in the HTML inputs, the `||23` fallback in `calculate()`, and the reset handler.
- Lineup enumeration is combinatorial (`combArr`); fine for small rosters only.
