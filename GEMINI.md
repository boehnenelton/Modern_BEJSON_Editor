# GEMINI.md — Merged Findings (Claude Audit + Gemini MFDB Roadmap)

> **Status: PARTIAL REMEDIATION APPLIED (2026-09-18/19), 104-family scope only.**
> **Fork added 2026-09-19: `Modern_BEJSON_Editor_104_ONLY.html`** — a second build with all `105`/`105a`/`105db` support completely removed (format picker option, validator acceptance, `field_uuid`/`record_uuid` generation, UUID-column offset math and display, all "105" UI text). `104db` was **not** added to this fork either, per explicit instruction — it supports exactly `104` and `104a`. All 104-family fixes below are present in both builds. Live-verified with 6/6 checks (title/radios, validator rejects "105", no UUID generation anywhere, array-cell parsing intact, MFDB generation still correct, About tab text accurate, zero console errors). The two builds are maintained in parallel going forward: `Modern_BEJSON_Editor.html` (104+105, current) and `Modern_BEJSON_Editor_104_ONLY.html` (104-only).
> **105 bug-fix pass added 2026-09-19 (`Modern_BEJSON_Editor.html` only):** two real bugs found and fixed in the 105/105a validation path (see new Tier 2 items 6b–6c below). **105db and 104db are explicitly, permanently out of scope** — 105db has been removed entirely from `Modern_BEJSON_Editor.html` (it was previously accepted by the validator and half-implemented in `btnAddRow`; both are now gone). This editor supports exactly `104`, `104a`, `105`, `105a`.
> **Tier 3 (Gemini's MFDB folder/mount roadmap) BUILT 2026-09-20, in both `Modern_BEJSON_Editor.html` and `Modern_BEJSON_Editor_104_ONLY.html`.** All 4 phases (see Tier 3 below) implemented and live-verified via Playwright on both files, including an isolated mount → edit → save test that confirms the write-back actually reaches the mocked file handle with the edited content. This is code-identical across both builds since MFDB manifest/entity handling never touches 105-series logic. Phase 5 (drag-and-drop, Flask sibling) remains deferred as originally scoped.
> **105-series schema constraints BUILT 2026-09-20 (`Modern_BEJSON_Editor.html` only — not applicable to `104_ONLY.html`, which has no 105 code).** The real "Integrity Era" schema-constraint feature (`required`/`unique`/`enum`/`min`/`max`/`minLength`/`maxLength`), previously flagged as a known gap in the tool's 105 support, is now fully built: Add/Edit Field modal UI (auto-hidden for 104/104a), storage on the field object, and full `validateBEJSON()` enforcement using the real library's own error-code prefixes (E90–E94). All six constraint types live-verified individually plus a full regression pass.
> **Constraint UI reworked to a dynamic list, 2026-09-20.** Per explicit request, the fixed one-of-each layout (checkboxes + four separate inputs) was replaced with a dynamic vertical list: each constraint is its own row (type dropdown + adaptive value input + remove button), with a "+ Add Constraint" button to append more. Same underlying storage/validation as above — this was a UI-only rework, not a behavior change. Live-verified: row add/remove, type-dependent input disabling, save→reopen round-tripping, and an end-to-end validation test confirming a UI-built constraint is actually enforced.
> This file merges two independent review passes on `Modern_BEJSON_Editor.html` — Gemini's original MFDB folder/mount feature roadmap (preserved in full below) and Claude's separate spec-compliance audit (`report/Audit_Report.md`) — into one prioritized remediation reference. A first remediation pass has since been applied and live-verified (Playwright, 8/8 checks passing) against the 104/104a/MFDB-manifest items only, per explicit instruction to focus on 104 first and to hold off on 104db. **105-family, 104db, and the Gemini MFDB folder/mount roadmap (Tier 3) remain untouched.**

---

## REMEDIATION INSTRUCTIONS (Merged, Priority-Ordered)

Items now marked **✅ DONE** have been applied to `Modern_BEJSON_Editor.html` and live-verified. Everything else is still pending approval/implementation.

### Tier 1 — Spec-Correctness Blockers

1. **Resolve the `105`/`105a`/`105db` format question.** — **RESOLVED: 105/105a in, 105db (and 104db) permanently out.** Confirmed via `libraries_105.zip`: 105-series is a real, deliberate "Integrity Era" format (`field_uuid` per field, `record_uuid` per row, opt-in schema constraints `required`/`unique`/`enum`/`min`/`max`/`minLength`/`maxLength`, error codes 90–94) with actual TS/PY library code and its own test suite — not scope creep, but also not present in either crash course (v23 or the superseding v24), and the prototype libraries have their own open, independently-audited bugs (see the included `105 audit.txt`). **Decision made 2026-09-19:** 105/105a stay in `Modern_BEJSON_Editor.html`, with their two real strict-integrity/validator bugs now fixed (see 6b/6c below). 105db has been removed entirely, and 104db remains unimplemented — both are final, not deferred. **Schema-constraint UI built 2026-09-20** (see the status note at the top of this file) — this format is now fully implemented per its own real spec, not just its identity layer.
2. **Add native `104db` support** — **DEFERRED PER EXPLICIT INSTRUCTION ("do not add 104db support yet").** Validator still does not accept `"104db"`; Hub format picker unchanged. Revisit when instructed.
3. ✅ **DONE — Fixed `MFDB_Version` to `"1.31"`** (was hardcoded `"1.40"`) **and fixed `Parent_Hierarchy` to the fixed, invariant `"104a.mfdb.bejson"` filename** (was a database-name-derived filename, e.g. `DemoDatabase.mfdb.bejson`, which would never resolve). Now emits `"../104a.mfdb.bejson"` from an entity file, matching the spec's worked example for a `data/<entity>.bejson`-nested entity. Live-verified via Playwright.
4. ✅ **DONE — Fixed `file_path` casing** in generated manifests to the spec convention (`data/<entity_name_lowercase>.bejson`; was PascalCase, e.g. `data/Users.bejson` → now `data/users.bejson`). Live-verified.

### Tier 2 — High-Impact Independent Bugs

5. **Fix the general save-disconnect bug** *(Gemini)* — ✅ **RESOLVED for mounted tables as of the 2026-09-20 Tier 3 build.** `saveActiveBejsonFile()` now writes directly back to a table's `FileSystemFileHandle` via `createWritable()` whenever one exists (i.e., whenever the table was loaded via "Mount MFDB Folder"). A table loaded any other way (plain file import, multi-file load, folder load without mounting, Hub creation) still uses the download-based save, since there's no OS file handle to write back to in those paths — that's an inherent browser-security constraint, not a remaining gap. See Tier 3 item 11 below for the implementation.
6. ✅ **DONE — Fixed the Live Schema tab's stale-content bug.** Added the missing `renderRawSchema()` call to the table-selector's `change` handler. Live-verified: switching tables while Live Schema is open now updates immediately.
7. ✅ **DONE — Repaired the corrupted syntax-highlighter regex.** Restored proper `\s`, `\d`, `\.`, `\b` escapes (previously missing backslashes / literal 0x08 backspace bytes). Confirmed zero backspace bytes remain in the file and the page loads with no console/JS errors.
8. **Enforce append-only Fields editing** *(Gemini)* — **NOT DONE this pass.** Deferred; would need a `Schema_Version`-tracking mechanism added first (the editor currently has no `Schema_Version` concept at all — see the crash-course verification findings from the prior turn), which is a larger, separate change than a one-line fix.

**New 105-series bugs found and fixed 2026-09-19 (`Modern_BEJSON_Editor.html` only — not applicable to `104_ONLY.html`, which has no 105 code at all):**

6b. ✅ **DONE — Fixed a false-positive validator failure affecting every 105/105a document with data.** The generic row-length check (`row.length !== doc.Fields.length`) didn't account for the reserved `record_uuid` column 105-series rows carry ahead of the declared `Fields`. Practical effect: `validateBEJSON()` rejected 100% of 105/105a documents created via the editor's own "+ Add Row" button with a false "Row length mismatch" error. Now computes an offset-aware expected length. Live-verified: a 105 document with 2 added rows now validates `true`.

6c. ✅ **DONE — Added real strict-integrity validation for 105/105a** (the real prototype library's own "Fail-on-Switch" requirement, previously completely unchecked): every `Fields` entry must have a non-empty, unique `field_uuid`; every `Values` row must have a non-empty, unique `record_uuid`. Live-verified: a cloned document with a duplicated `field_uuid` or a missing/duplicated `record_uuid` is now correctly flagged invalid with a specific error message; a well-formed document still validates clean.

**105db and 104db — final decision, not deferred:** 105db has been **removed** from `Modern_BEJSON_Editor.html` entirely — it is no longer in `validVersions`, and the `btnAddRow` discriminator-column logic that referenced it (including a since-reverted attempt at a proper discriminator prompt) has been stripped back out. 104db remains unimplemented as before. **Neither is planned.** This editor's permanent supported-format set is `104`, `104a`, `105`, `105a`.

**New item found and fixed during this pass (not in the original list from either source):**

8a. ✅ **DONE — Fixed array/object-typed cell values being stored as raw strings instead of parsed JSON.** `updateCellData()` now `JSON.parse()`s the input for `array`/`object` fields and validates the parsed type matches the field's declared type, rejecting (with a clear alert, no data change) anything that doesn't parse or doesn't match. Also fixed the companion display bug where re-opening such a cell showed `[object Object]` (via `String(val)`) instead of its actual JSON — both the inline grid cell and the Advanced Cell Editor modal now `JSON.stringify()` object/array values for display. Live-verified: a valid array save now passes the editor's own `validateBEJSON()`; an invalid one is rejected and the prior value is preserved.

### Tier 3 — MFDB Folder/Mount Feature Work (Gemini's roadmap)

**Built 2026-09-20, in both `Modern_BEJSON_Editor.html` and `Modern_BEJSON_Editor_104_ONLY.html`.**

9. ✅ **DONE — Phase 1, Detection & Awareness.** Added `classifyBejsonDoc()`, wired into a new shared `pushLoadedTable()` helper used by every load path (single-file import, Hub creation, multi-file/folder load, and mount). Loaded tables are automatically tagged `[Manifest]` / `[Entity]` in the dropdown; no behavior change to loading/saving beyond the label. Live-verified.
10. ✅ **DONE — Phase 2, Broadened Manual Loading.** Added a "Load Multiple Files" button (plain multi-file picker) and a "Load MFDB Folder" button (`webkitdirectory`, captures real `webkitRelativePath` values). Added `resolveMfdbLinks()`: matches every loaded manifest's declared entity rows against currently loaded tables (primary match on `Records_Type[0]` vs. `entity_name`, secondary match on relative-path/basename), and renames the manifest's dropdown entry to show `(x/y resolved)` plus an alert summarizing anything missing. Live-verified with a real 3-file MFDB fixture (manifest + 2 entities in a `data/` subfolder): folder load correctly resolved "2/2" via both the flat multi-file path and the real-relative-path folder path.
11. ✅ **DONE — Phase 3, True Mount.** Added "Mount MFDB Folder (Chrome/Edge)", shown only when `'showDirectoryPicker' in window`. `mountMfdbFolder()` opens the picker, locates the fixed `104a.mfdb.bejson` filename at the folder root, and walks each entity's `file_path` via a new `resolvePathInDirectory()` helper to locate and load it, storing a live `FileSystemFileHandle` per table. `saveActiveBejsonFile()` was rewritten (now `async`) to check for a stored file handle and write directly back via `createWritable()` when one exists — **this closes Gemini's originally-flagged "general save-disconnect bug" (Tier 2 item 5) for every mounted table**, not just MFDB entities — falling back to the existing download behavior for any non-mounted table. Live-verified end-to-end with a mocked directory-handle tree: mount → add a row → save, then read back the exact bytes written to the mocked handle and confirmed the edit was present. Item 5 above is now effectively resolved for the mounted case; a non-mounted single-file document still uses the download-based save as before (unavoidable without a mount, since there's no OS file handle to write back to).
12. ✅ **DONE — Phase 4, Export Pipeline.** Added "Export Loaded Tables as ZIP", using a hand-rolled store-only ZIP writer (own CRC32 table, no external dependency, consistent with the file's zero-dependency design). Runs `validateBEJSON()` across every currently loaded table first and blocks the export with a specific per-table error list on any failure; on success, resyncs every manifest's `record_count` field against the actual row count of its matched loaded entity before writing the archive — **this also closes the previously-flagged "record_count never resynced" gap** from the original Claude audit report. Live-verified: the exported zip opens correctly with Python's standard `zipfile` module, contains the correct relative paths, and shows the resynced `record_count` (tested going from `0` to the real row count after adding rows).
13. **Phase 5, Optional/Deferred** — still not built, as originally scoped. Directory drag-and-drop and a possible Flask MFDB-aware sibling app remain future, non-committed ideas.

### Tier 4 — Governance & Polish

14. ✅ **DONE — Changed the default type of the first user-entered field** in the Hub's quick-create flow from `integer` to `string`. Live-verified.
15. ✅ **DONE — Added the optional MFDB manifest headers** `Author`, `Created_At`, `DB_Description` at generation time. Live-verified present in generated manifests.
16. ✅ **DONE — Added a file-header comment block**, unified the tool's name to **"BEJSON Editor"** across `<title>`, header bar, and About tab (was three different names), added the full four-part credit line to the About tab, converted the displayed version to a `VERSION`/`PACKAGE_VERSION` JS-constant-driven solid-integer-style display (`101 (Pkg 146)`, bumped from `1.0.0 (Pkg 145)`), and fixed the light-theme active-tab color to `#DE2626` red (was black, correct only in dark theme). Live-verified: title, credit line, version display, and both themes' active-tab color all confirmed via Playwright.
17. ✅ **DONE — Extended the `__proto__`/`constructor`/`prototype` forbidden-name check** to the custom-header-key input path in `addCustomHeader()`. Live-verified: attempting to add a `__proto__` custom header now alerts and is rejected, matching the existing field-name guard.
18. Add an accessible label to the grid search input — **NOT DONE this pass** (low priority, deferred with Tier 3).

---

## Source A: Claude Audit Findings (Summary)

Full detail, line numbers, code excerpts, and screenshots live in `report/Audit_Report.md` (398 lines) from the companion audit package — not reproduced here in full to avoid duplicating a 300+ line document inside this one. Section references above (§3.x) point into that file. Headline severities:

- **High:** missing `104db` support; undocumented `105`/`105a`/`105db` format family; wrong `MFDB_Version` and `Parent_Hierarchy` in generated MFDB manifests.
- **Medium:** Live Schema stale-content bug on table switch; corrupted syntax-highlighter regex (also flagged as a pipeline/process signal, not just a code bug).
- **Low:** first-field-forced-to-integer default; governance/branding gaps (file header, credit line, naming consistency, version format, light-theme palette); `__proto__` guard gap on custom headers; minor accessibility gaps.

## Source B: Gemini MFDB Roadmap (Original, Preserved Verbatim)

The following is Gemini's original actionable-suggestions and roadmap text, unchanged from the uploaded file, retained here for provenance:

### Actionable Suggestions (Gemini, original)

- Ship the detection algorithm first, independent of everything else. Add `classifyBejsonDoc()` to the existing `ingestDocument()` path. Zero risk, zero new browser capability required, immediately useful (manifest and entity files get labeled correctly in the combo box even before any auto-loading exists).
- Add `multiple` to the existing file input as the very next step. One-line HTML change, universal browser support, and it already unlocks "select manifest + all entities in one dialog" today.
- Add a `webkitdirectory` "Load MFDB Folder" button as a second, explicit entry point next to the existing single-file/paste/drop options — don't replace the existing flows, add to them. This is the first mechanism that preserves real relative paths.
- Implement Mechanism 3 (File System Access API) behind a feature-detection guard (`if ('showDirectoryPicker' in window)`), exposed as a clearly-labeled "Mount MFDB Folder (Chrome/Edge only)" action so it's obvious to a non-Chromium user why it's missing rather than silently broken.
- Fix the general save-disconnect bug independent of MFDB work. Any single loaded file, MFDB or not, currently loses its link to its source location on save. Wiring `createWritable()` into `saveActiveBejsonFile()` whenever a File System Access handle exists for the active document fixes this for the whole editor, not just MFDB entities — this is likely the single highest real-world-impact change in this report.
- Gate `record_count` resync and `Is_Mounted` correctness into the export routine, not into every save — these are export-time concerns per spec, not per-edit concerns.
- Add an "unresolved manifest row" visual state to the combo box so a partially-loaded MFDB is never silently mistaken for a fully-loaded one.
- Enforce append-only Fields editing in the schema tab, closing the one real structural risk manual editing currently carries, independent of the MFDB work.
- Build the client-side zip export as a standalone feature usable even without full MFDB auto-detection — useful the moment multiple files are loaded in any combination.
- Defer the Flask MFDB variant entirely until/unless cross-browser or fully-unattended directory scanning becomes an explicit, separate requirement — do not fold it into this feature's roadmap by default.

### Implementation Roadmap (Gemini, original)

**Phase 1 — Detection & Awareness (near-zero risk)**
- `classifyBejsonDoc()` wired into `ingestDocument()`.
- Combo box entries visually tagged: `[Manifest]`, `[Entity]`, `[Standalone]`.
- No behavior change to loading/saving yet — purely informational.

**Phase 2 — Broadened Manual Loading**
- `multiple` on the existing file input; basename-matching against a loaded manifest.
- `webkitdirectory` "Load MFDB Folder" button; relative-path matching against a loaded manifest.
- Combo box gains "resolved / unresolved" states per manifest row.

**Phase 3 — True Mount (Chromium-first, feature-detected)**
- File System Access API directory mount.
- `resolveRelativePath()` walking every manifest `file_path` on demand.
- `createWritable()` wired into save — closes the "won't mount back" problem for real, for both MFDB entities and ordinary single-file documents.
- Graceful, explicit fallback messaging on non-Chromium browsers pointing back to Phase 2 tools.

**Phase 4 — Export Pipeline**
- In-browser zip assembly (hand-rolled store-only writer to preserve zero-dependency status, or JSZip via CDN if a dependency is acceptable).
- Pre-export Level 1/Level 2 validation pass across the full loaded set.
- `record_count` resync, `Package_Version` bump, `Is_Mounted` correctness, standard naming convention on the output archive.

**Phase 5 — Optional, Deferred**
- Directory drag-and-drop as a UX convenience layered on whichever of Mechanism 2/3 is already implemented.
- Flask MFDB-aware sibling build, only if a concrete cross-browser/unattended-scan requirement emerges — evaluated as its own decision, not a default continuation of this roadmap.

---

*Merged by Claude (Anthropic) on request; partial remediation applied and live-verified 2026-09-18/19 per explicit instruction (104-family focus, 104db deferred). Original Gemini content preserved verbatim in Source B. Tier 3 (Gemini's MFDB folder/mount roadmap) and the 104db/105-family Tier 1 items remain unimplemented pending further instruction.*
