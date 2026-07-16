## Research: Hermes Project Deletion Support — Findings & Proposed Flow

### Phase 1 — What Hermes provides natively

**No `hermes project delete` exists.** The project verbs are: `create`, `list/ls`, `show`, `add-folder`, `remove-folder`, `rename`, `set-primary`, `use`, `archive`, `restore`, `bind-board`. `archive` is a soft-delete (sets `archived=1`); `restore` brings it back.

**However, `delete_project()` exists in the DB layer** at `hermes_cli/projects_db.py:586-590`:

```python
def delete_project(conn: sqlite3.Connection, project_id: str) -> bool:
    """Hard-delete a project and its folders (cascade)."""
    with write_txn(conn):
        cur = conn.execute("DELETE FROM projects WHERE id = ?", (project_id,))
    return cur.rowcount > 0
```

It is **not wired into the CLI** — no subcommand calls it. The schema uses `ON DELETE CASCADE` on `project_folders`, so folders auto-cleanup. But the CLI has no verb to invoke it.

**Skill deletion:** No `hermes skill delete`. Skill cleanup routes are:
- `hermes skills uninstall <name>` — removes hub-installed skills
- `hermes curator archive <name>` — moves agent-created skill dir to `~/.hermes/skills/.archive/`
- `hermes curator prune` — bulk-archive idle skills (>=90d by default)
- Skills are **archived, never truly deleted** by design

**Session deletion:** `hermes sessions delete <id>` and `hermes sessions prune` exist. `prune` supports `--cwd <path>` for filtering by working directory — useful for deleting sessions tied to a workspace path.

**Sessions DON'T have direct project_id foreign keys.** They resolve projects via `project_for_path()` — longest-prefix match of session cwd against project folders. Deleting a project leaves sessions intact; they just no longer resolve to any project.

**Current state on this machine:**
- Projects: `the-long-reign`, `chaos-dashboad`, `audhd-toolkit` (active)
- ws-* skills: `ws-chaos-dashboard-v1`, `ws-chaosgoblin`, `ws-drey`, `ws-job-search`, `ws-ndd-and-mental-health`, `ws-neon-blackout`, `ws-neon-whiteout`, `ws-the-long-reign`
- `projects.db` at `~/.hermes/projects.db` (SQLite, per-profile), `state.db` holds sessions

---

### Phase 2 — Proposed deletion flow for `/workspace delete`

#### 1. Cleanup scope — what gets deleted

| Artifact | Location | Method |
|----------|----------|--------|
| **Native Hermes project** | `~/.hermes/projects.db` | Call `pdb.delete_project()` directly (or raw SQL if CLI access needed) |
| **ws-* skill** | `~/.hermes/skills/ws-<slug>/` | `hermes curator archive ws-<slug>` — moves to `.archive/` |
| **Workspace directory** | `~/.hermes/workspaces/<name>/` | `rm -rf` (config.yaml + snapshots/) |
| **Sessions (optional)** | `~/.hermes/state.db` | `hermes sessions prune --cwd <primary_path> --dry-run` then with `--yes` |

**Not deleted:**
- The actual workspace repo/folder (e.g., `~/Projects/audhd-toolkit/`) — that is user data, not plugin-managed
- Kanban board bindings — the board itself lives in `kanban.db` (root-anchored, shared across profiles). The project->board binding dies with the project row.

#### 2. Cleanup order (least -> most recoverable)

```
1. BACKUP: Optionally snapshot workspace dir (snapshots/ -> a temp location)
2. ARCHIVE SKILL: `hermes curator archive ws-<slug>`
3. DELETE WORKSPACE DIR: `rm -rf ~/.hermes/workspaces/<name>/`
4. DELETE PROJECT: Call `pdb.delete_project(conn, project_id)` or raw SQL
5. PRUNE SESSIONS: `hermes sessions prune --cwd <primary_path>` (optional, user-prompted)
```

**Rationale:** If step 2 succeeds but 3 fails, the skill is recoverable from `.archive/`. If 4 succeeds but 2 failed, we have an orphan project — easy to detect. If 4 fails, we still have the project row and can retry. Sessions are last because they are the most ephemeral and least impactful.

#### 3. Safety — confirmation prompt

Before deletion, display:

```
⚠️  About to delete workspace "audhd-toolkit":

  Project:     audhd-toolkit (p_abc12345)
  Path:        ~/Projects/audhd-toolkit/
  Skill:       ws-audhd-toolkit -> ~/.hermes/skills/.archive/
  Workspace:   ~/.hermes/workspaces/audhd-toolkit/ (N snapshots, created YYYY-MM-DD)
  Sessions:    M sessions with cwd under this path (--dry-run to review)

  The repo at ~/Projects/audhd-toolkit/ will NOT be deleted.

  Type the workspace name to confirm: _
```

- Require typing the **exact workspace name** (not "yes" or "y")
- No `--force` / `--yes` flag for delete
- `--dry-run` prints what would happen without acting

#### 4. Partial failure recovery

| Failure state | Symptoms | Recovery |
|--------------|----------|----------|
| Project deleted, skill still in `~/.hermes/skills/` | Orphan ws-* skill; `hermes project list` clean | `hermes curator archive ws-<slug>` or delete manually |
| Skill archived, project still in `projects.db` | Project visible in `hermes project list`; no ws-* skill | Call `pdb.delete_project()` directly, or let user `hermes project archive` as soft-delete |
| Workspace dir deleted, project + skill intact | Lost snapshots/config; project still functional | Re-create `~/.hermes/workspaces/<name>/` with empty config.yaml (snapshots are gone) |
| Project deleted, workspace dir still exists | Orphan directory at `~/.hermes/workspaces/<name>/` | `rm -rf` the directory manually |
| Sessions not pruned | Old sessions still queryable; no project resolution | Sessions are harmless — `hermes sessions prune --cwd <path>` anytime |

**Implementation recommendation:** The `/workspace delete` handler should:
1. Run each step, collecting results
2. If any step fails, report which succeeded, which failed, and exact recovery commands
3. Never leave the state in an un-reportable mess — always list what is where

#### 5. Since Hermes has no native CLI delete, here is the manual equivalent

```bash
# Step 1: Archive the ws-* skill
hermes curator archive ws-audhd-toolkit

# Step 2: Remove workspace directory
rm -rf ~/.hermes/workspaces/audhd-toolkit/

# Step 3: Delete the project from projects.db (SQLite)
sqlite3 ~/.hermes/projects.db "DELETE FROM projects WHERE slug='audhd-toolkit';"

# Step 4 (optional): Prune old sessions
hermes sessions prune --cwd ~/Projects/audhd-toolkit --dry-run
# Review, then:
hermes sessions prune --cwd ~/Projects/audhd-toolkit --yes
```

**Code-level approach for the plugin:** Import `hermes_cli.projects_db` directly and call `delete_project()` — no CLI wrapping needed. This keeps the plugin self-contained while using the existing DB function that already handles cascading folder cleanup.

---

### Summary

| Question | Answer |
|----------|--------|
| `hermes project delete` exists? | **No.** Only `archive`/`restore` |
| `delete_project()` in DB layer? | **Yes** — `projects_db.py:586`, but not wired to CLI |
| Skill deletion? | `hermes curator archive` (move to `.archive/`, never true delete) |
| Session deletion? | `hermes sessions delete` / `prune` with `--cwd` filter |
| Session->project link? | No FK; resolved via `project_for_path()` on cwd |
| Recommended approach for plugin? | Call `pdb.delete_project()` directly; archive skill via curator; `rm -rf` workspace dir; optional session prune |
