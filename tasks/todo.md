# v0.3 Release Plan (script-only)

Branch: `feature/v0.3-script`
Previous version: 0.2.2

## Strategy

v0.3 stays a **single PowerShell Gallery script** (`Publish-Script`, same as 0.2.x).
The module migration from the old `feature/v0.3` branch was dropped: converting the
PSGallery package from script to module would require freeing the package name via
PSGallery support and retraining users, for a distribution that v0.4 replaces anyway.

- `feature/v0.3-script` branches from the last pre-migration commit (`bcdfc4a`), so all
  v0.3 bug fixes and features are already in script form.
- The functional content of the three module-only commits was backported:
  OData filter escaping, Defender settings-gating, device-code auth, dashboard
  loading feedback, and the search-filter documentation notes.
- v0.3 is the **final PowerShell release**. v0.4 is the WinUI app rework
  (branch `codex/v0.4-winui-app`) and fully replaces the script.
- Old PR #61 (module migration) should be closed in favor of the PR from this branch.

## Goal

Fix everything users have filed issues for that can reasonably be fixed in the script,
then ship 0.3.0. Features that require a real app platform move to v0.4.

## Issue triage (all open issues as of 2026-07-07)

### Fixed on this branch

- [x] **#9 Partial serial search** — "Contains (partial match)" search mode; Autopilot partial serial matching is client-side.
- [x] **#11 Defender offboarding** — Optional, disabled by default, gated behind the Prerequisites-dialog setting; app-only + delegated token acquisition.
- [x] **#13 LAPS information** — LAPS password shown in the confirmation dialog (`DeviceLocalCredential.Read.All`).
- [x] **#15 Playbooks grayed out / unavailable** — All 10 playbooks implemented; playbooks are now bundled inside the script and extracted on Gallery installs.
- [x] **#17 Device stays in Autopilot** — Exact serial correlation resolves the Autopilot identity before offboarding; BitLocker 403 fixed via `BitlockerKey.Read.All` scope.
- [x] **#21 Autopilot group tags** — "Set Group Tag" action for selected devices.
- [x] **#33 Query and multi-actions in device list** — Grid filtering, shift-click range selection.
- [x] **#35 Log location & BitLocker key logging** — Timestamped `DOM_*.log` audit log; recovery keys logged with `SENSITIVE` prefix.
- [x] **#37 DisplayName search 400 on empty Autopilot displayName** — Device-name search pre-fetches Autopilot devices and matches client-side; no server-side displayName filter.
- [x] **#38 Crash after copying BitLocker key** — Clipboard errors are caught inside the copy handler; confirmation/summary dialogs null-guard `ShowDialog`.
- [x] **#40 Dashboard platform filtering** — Platform ComboBox filters dashboard statistics.
- [x] **#41 Corporate Device Identifier (stale) support** — Corporate identifier stale report playbook.
- [x] **#46 Dashboard blank / device-name search fails** — Dashboard `$count` batch rework with full-fetch fallback; device-name search rebuilt (see #37). *Needs live-tenant confirmation from reporter.*
- [x] **#47 / #49 / #50 / #56 Wrong device deleted** — Strict identity matching: Graph-ID/exact-serial correlation, deletes operate on resolved object IDs only, and the confirmation dialog shows the exact Entra/Intune/Autopilot IDs before anything is deleted.
- [x] **#48 Automate JSON import** — Saved authentication config auto-loads from `%LocalAppData%/DeviceOffboardingManager`.
- [x] **#54 Offboard by Entra Device ID** — "Device ID" search option (object ID and `deviceId`).
- [x] **#55 / #59 Interactive login localhost redirect / WAM failures** — Device Code login option avoids the browser redirect entirely.
- [x] **#58 Multi-Admin Approval query** — MAA-style protected-operation responses are detected and summarized in the results.

### Remaining for 0.3

- [ ] **#52 / #53 Forbidden (403) errors during offboarding** — Root cause is tenant-side (missing directory role such as Cloud Device Administrator / Intune Administrator, or MAA). Surface an actionable message for 403 responses (which role/permission is likely missing) instead of raw JSON, and document required roles in the README.
- [ ] **README refresh** — Still describes 0.2.x usage; update for 0.3 (Defender opt-in, device-code login, saved auth config, playbook bundling, required roles), and state the v0.4 WinUI plan / final-script-release status.
- [ ] **Release prep** — Bump `.VERSION` to 0.3.0, set changelog date, tag, `Publish-Script` via the existing `publish-script.yml` workflow.
- [ ] **Dead code / dashboard-card background jobs** — Optional polish carried over from the old plan; not release-blocking (cards now show a loading indicator).

### Deferred to v0.4 (WinUI app)

- **#60 Win32/Store application** — This *is* v0.4 (`codex/v0.4-winui-app`; MSIX packaging + signing pipeline already in place).
- **#14 Localization / French** — Needs a string-resource architecture; planned on the WinUI app, not the script.
- **#62 AD, ConfigMgr & MDE offboarding** — MDE part shipped in 0.3 (see #11). On-prem AD/ConfigMgr offboarding needs a local agent/connector story; evaluate for the WinUI app backlog.
- **#3 Restructure codebase** — Superseded: the script stays monolithic for its final release; the WinUI app is the restructured codebase.

## Notes

- All Graph calls use `beta` endpoints.
- The embedded playbook block in `DeviceOffboardingManager.ps1` must be kept in sync with
  the `Playbooks/` directory (regenerate the here-string block when a playbook changes).
- Keep the `NOTE:` comments at the Intune search filters: `serialNumber` only supports `eq`,
  and combining `contains(deviceName)` with `contains(serialNumber)` poisons the query.
