# v0.3 Release Plan

Branch: `feature/v0.3`
Previous version: 0.2.2

---

## Phase 1: Critical Bug Fixes

These are broken today and must be fixed before any new features.

- [x] **Fix export functions** — Replaced non-existent properties with actual `DeviceObject` class properties.
- [x] **Fix playbook download path** — Load from local `$PSScriptRoot/Playbooks/` instead of downloading from GitHub.
- [x] **Fix BitLocker key retrieval** — Use `$keyIdResponse[0].id` to access first element.
- [x] **Fix Playbook_1 field name** — `lastContactDateTime` -> `lastContactedDateTime`.
- [x] **Fix `$deviceSuccess++` in PSCustomObject** — Extracted counter logic before PSCustomObject.
- [x] **Fix permission scopes** — `Device.ReadWrite.All`, added `BitlockerKey.Read.All`. Kept unused scopes for future phases.
- [x] **Fix stale `$script:parsedDevices`** — Clear on cancel button handler.
- [x] **Fix playbook result columns** — Dynamic column generation from playbook output schema.

---

## Phase 2: Safety & Identity (Core v0.3 Theme)

Addresses wrong-device deletion (Issues #47, #49) and audit requirements (#35).

- [ ] **Strict device identity matching** — Replace `-like "*$deviceName*"` partial matching with exact match for Autopilot client-side filtering. Use serial number as primary correlation key. When multiple matches exist, show all and require explicit selection.
- [ ] **Dry run / preview mode** — Before any delete, resolve all Graph object IDs and display them in the confirmation dialog. Show exact Entra ID object ID, Intune device ID, and Autopilot identity ID for each device being removed.
- [ ] **Search by Entra Device ID** — Add "Device ID" as a third option in the search dropdown (Issue #54). Use IDs internally for all delete operations.
- [ ] **Disable-before-delete option** — Add "Disable in Entra ID" action (`PATCH /devices/{id}` with `accountEnabled: false`) as an alternative to immediate deletion.
- [ ] **Persistent audit logging** — Timestamped log filenames (e.g., `DOM_20260313_143022.log`). Log admin UPN, device identifiers, Graph object IDs, action taken, and success/failure. Log recovery keys before deletion (with sensitivity warning).

---

## Phase 3: Graph API Improvements

- [x] **Add retry logic with backoff** — `Invoke-GraphRequestWithRetry` handles 429 throttling (reads Retry-After header), 5xx transient errors (exponential backoff), and network-level failures.
- [x] **Update `Get-GraphPagedResults`** — Uses retry wrapper, accepts optional `$Headers` parameter for `ConsistencyLevel: eventual`.
- [x] **Implement batch requests** — `Invoke-GraphBatchRequest` helper: auto-chunks >20 requests, retries failed sub-requests. Used for search (Entra+Intune), offboarding (Entra+Intune+Autopilot per device), and dashboard `$count`.
- [x] **Migrate all v1.0 endpoints to beta** — Zero v1.0 references remaining across main script and all 5 playbooks.
- [x] **Add `$select` to all GET calls** — Every GET endpoint now specifies only the properties accessed downstream.
- [x] **Add `$count` for dashboard statistics** — Single `$batch` call with 13 `$count` sub-requests replaces 3 full-collection fetches + client-side counting. Includes per-OS counts for pie chart. Falls back to full-fetch if `$count` fails.
- [x] **Batch search queries** — Devicename search batches Entra+Intune; serial search batches Intune+Autopilot. Autopilot full-fetch for devicename search hoisted out of per-term loop.
- [x] **Batch offboarding operations** — Per-device batch combines Entra+Intune+Autopilot operations into single `$batch` call.
- [x] **Implement bulk Autopilot deletion** — When 2+ devices selected, uses `deleteDevices` bulk endpoint. Falls back to individual deletion on failure.

---

## Phase 4: New Features

### Offboarding Actions
- [ ] **Retire/Wipe before delete** — Add optional pre-offboarding actions: Retire (`POST .../retire`), Wipe (`POST .../wipe`), or Delete-only. Enforce correct ordering.
- [ ] **Defender for Endpoint offboarding** — Add MDE as a fourth service checkbox (`POST /api/machines/{id}/offboard`). Requires `Machine.Offboard` permission. (Issue #11)
- [ ] **LAPS password retrieval** — Display LAPS password in confirmation dialog alongside BitLocker/FileVault. `GET /deviceLocalCredentials/{id}` with `DeviceLocalCredential.Read.All`. (Issue #13)
- [ ] **Multi-Admin Approval awareness** — Detect MAA pending state in API responses. Show notification when action requires second admin approval. (Issue #58)

### Search & Display
- [ ] **Partial/wildcard search** — Add "Contains" search mode using `startsWith()` / `contains()` OData filters where supported. (Issue #9)
- [ ] **Device group membership display** — Show Entra ID group memberships via `GET /devices/{id}/memberOf` for impact assessment before offboarding.
- [ ] **Device compliance state** — Display `complianceState` from managed device properties in results grid.

### Playbooks
- [ ] **Implement Playbook 6: OS-Specific devices** — Filter by specific OS platform.
- [ ] **Implement Playbook 7: Outdated OS devices** — Devices not running latest OS version.
- [ ] **Implement Playbook 8: EOL OS devices** — Devices running end-of-life OS versions.
- [ ] **Implement Playbook 9: BitLocker Key Report** — Audit report of all BitLocker recovery keys.
- [ ] **Implement Playbook 10: FileVault Key Report** — Audit report of all FileVault recovery keys.

---

## Phase 5: Code Quality & UX

- [ ] **Load playbooks from local filesystem** — Use `$PSScriptRoot/Playbooks/` instead of downloading from GitHub at runtime. Eliminates security risk of remote code execution without integrity verification.
- [ ] **Fix dashboard threading** — Primary `$count` path no longer uses thread jobs (fixed in Phase 3). Fallback full-fetch path still uses thread jobs without Graph auth context. Either pass token explicitly or use `-InitializationScript` to re-authenticate in worker threads.
- [ ] **Fix dashboard card UI blocking** — Card click handlers run synchronous Graph calls. Move to background jobs with loading indicators.
- [ ] **Fix search box Enter key** — Remove `AcceptsReturn="True"` and add `KeyDown` handler to submit on Enter.
- [ ] **Make window resizable** — Change `ResizeMode="NoResize"` to `CanResize` with minimum size constraints. Current 1200x700 clips on many laptops.
- [ ] **Remove dead code** — Empty `GotFocus`/`LostFocus` handlers, unused `ToastNotificationStyle`, non-functional `RemoveHandler` calls.
- [ ] **Deduplicate utility functions** — Extract `Get-GraphPagedResults` and `ConvertTo-SafeDateTime` into a shared module or dot-sourced file. Currently duplicated 9 and 6 times respectively.
- [ ] **Clear client secret after use** — Zero out `$AuthDetails.Secret` after `Connect-ToGraph` completes.

---

## Phase 6: Polish (If Time Permits)

- [ ] Advanced grid filtering and shift-click range selection (Issue #33)
- [ ] Offboarding report generation — HTML/PDF audit artifact
- [ ] Saved authentication config for service principals (Issue #48)
- [ ] Platform filtering on dashboard (Issue #40)
- [ ] Autopilot group tag management (Issue #21)
- [ ] Localization / multi-language support (Issue #14)
- [ ] Co-management awareness — Detect and warn about co-managed devices

---

## Notes

- All Graph API calls should use `beta` endpoints (not v1.0) for richer response data
- Run `feature-dev:code-reviewer` before marking any phase complete
- Each phase should be a separate PR or commit group for clean history
