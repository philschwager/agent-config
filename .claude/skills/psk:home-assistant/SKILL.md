---
name: psk:home-assistant
description: >
  Manage a live Home Assistant instance and its configuration via the home-assistant (ha-mcp) MCP server.
  Use when creating/editing/debugging automations, scripts, scenes, dashboards, helpers, or YAML config;
  checking system health, logs, or updates; or backing up/restoring HA. Auto-triggers on Home Assistant work
  and also runs explicitly with: status | backup | automate | dashboard | audit.
argument-hint: "[status|backup|automate|dashboard|audit]"
---

# Home Assistant Management

Orchestration entry point for managing the user's Home Assistant instance (configured in `.mcp.json`,
served by the **home-assistant** / ha-mcp MCP server). This skill routes work to the right `ha_*` tools and
enforces a strict safe-change lifecycle. It deliberately does **not** restate Home Assistant domain rules —
those live in the MCP-provided `home-assistant-best-practices` skill, which is maintained upstream and tracks
monthly HA breaking changes. Always consult that skill (Rule 0) before changing config.

## Rule 0 — Consult best practices first (mandatory)

Before creating or modifying **any** automation, script, scene, helper, dashboard, or YAML config, read the
MCP-provided best-practices skill and the specific reference rows that match the task:

1. Read `skill://home-assistant-best-practices/SKILL.md` via the **ReadMcpResourceTool** (server:
   `home-assistant`).
2. That SKILL.md has a **Reference Files** table — read only the `references/*.md` rows that match your task
   (e.g. `automation-patterns.md` for triggers/modes, `safe-refactoring.md` for renames, `helper-selection.md`
   for helpers, `dashboard-guide.md` for Lovelace).
3. Fallback if MCP resources are unavailable: the `ha_get_skill_guide` tool.

Do **not** re-derive or paraphrase these rules here or from memory — HA changes monthly (e.g. `color_temp_kelvin`,
presence state triggers, "Apps" rename, Labs trigger behavior). The upstream skill is the source of truth.

## Safe-change lifecycle (the spine — every mutating command follows this)

```
Discover → Plan → Backup → Change → Validate → Verify
```

1. **Discover** — Ground in real data; never guess entity_ids. Use `ha_get_overview`, `ha_search`,
   `ha_get_entity`, `ha_get_state`, `ha_list_floors_areas`.
2. **Plan** — Summarize the intended change. For batch or destructive edits, state the plan back to the user
   before proceeding.
3. **Backup** — Run `ha_manage_backup` (create) before any destructive or config-file change. Note the backup id.
4. **Change** — Use the config API tools (`ha_config_set_automation`, `ha_config_set_script`,
   `ha_config_set_scene`, `ha_config_set_dashboard`, `ha_config_set_helper`, …). Never hand-write `.storage/`
   files or raw YAML.
5. **Validate** — Reload/validate (`ha_reload_core` or the relevant domain reload) and check for config errors
   via `ha_get_logs`.
6. **Verify** — Re-read state (`ha_get_state` / `ha_get_entity`); for automations, inspect
   `ha_get_automation_traces` to confirm the run path. Report the outcome honestly, including failures.

## Commands

| Command | Purpose | Key tools |
|---|---|---|
| `status` (default) | Health snapshot: overview, system health, pending updates, recent errors | `ha_get_overview`, `ha_get_system_health`, `ha_get_updates`, `ha_get_logs` |
| `backup` | Create or restore a backup | `ha_manage_backup` |
| `automate` | Create/edit/debug automations, scripts, scenes | `ha_config_set_automation` / `_script` / `_scene`, `ha_get_automation_traces`, `ha_eval_template` |
| `dashboard` | Create/edit Lovelace dashboards | `ha_config_set_dashboard`, `ha_config_get_dashboard`, `ha_config_list_dashboard_resources` |
| `audit` | Read-only review flagging anti-patterns & improvements | `ha_get_overview`, `ha_config_get_automation`, `ha_search` + best-practices skill |

When invoked with no argument, run `status`.

### status
Read-only. Report `ha_get_overview`, `ha_get_system_health`, `ha_get_updates`, and recent errors from
`ha_get_logs`. Summarize health, pending updates, and any error/warning worth action. Make no changes.

### backup
Create a backup with `ha_manage_backup` and report the backup id. To restore, confirm the target backup with
the user first, then restore via `ha_manage_backup`. Treat restore as destructive.

### automate
Full lifecycle. Apply **Rule 0** (read `automation-patterns.md`, plus `safe-refactoring.md` when editing
existing config), then run the lifecycle: discover entity_ids → plan → backup → set via `ha_config_set_*` →
reload/validate → verify with `ha_get_automation_traces`. Use `ha_eval_template` to test any template before
committing it.

### dashboard
Apply **Rule 0** (read `dashboard-guide.md` / `dashboard-cards.md`), then lifecycle. Fetch current config with
`ha_config_get_dashboard` before editing; set via `ha_config_set_dashboard`. Verify the dashboard loads.

### audit
**Strictly read-only — make no changes.** Enumerate automations/scripts/helpers (`ha_get_overview`,
`ha_config_get_automation`, `ha_search`), compare against the best-practices skill (Rule 0), and report a
prioritized list of anti-patterns and improvement opportunities. Offer to fix items via `automate` only after
the user chooses.

## Hard rules

- **Never** edit `.storage/` files or hand-write raw YAML — use the config API tools.
- Prefer `entity_id` (or `device_ieee` for ZHA) over `device_id`.
- `audit` is read-only; never mutate during an audit.
- Destructive or batch changes: back up first and state the plan back to the user before proceeding.
- For exact native-construct / automation-mode / refactoring rules → read the MCP skill (Rule 0). Don't guess
  from memory; HA changes monthly.

## Tools by lifecycle phase

- **Best practices (Rule 0):** `ReadMcpResourceTool` (server `home-assistant`, uri
  `skill://home-assistant-best-practices/SKILL.md`); fallback `ha_get_skill_guide`.
- **Discover:** `ha_get_overview`, `ha_search`, `ha_get_entity`, `ha_get_state`, `ha_list_floors_areas`,
  `ha_get_device`.
- **Backup:** `ha_manage_backup`.
- **Change:** `ha_config_set_automation`, `ha_config_set_script`, `ha_config_set_scene`,
  `ha_config_set_dashboard`, `ha_config_set_helper`, `ha_call_service`.
- **Validate:** `ha_reload_core`, `ha_get_logs`, `ha_eval_template`.
- **Verify:** `ha_get_state`, `ha_get_entity`, `ha_get_automation_traces`, `ha_get_history`.

**Related MCP servers** available for device-level help: `shelly-docker-custom` / `shelly-api-docs` (Shelly
devices), `context7` (library/API docs).
