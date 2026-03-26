# Fork Notes — JayCodesX/paperclip

This is a maintained fork of [paperclipai/paperclip](https://github.com/paperclipai/paperclip).

## Why This Fork Exists

The upstream Paperclip project is the foundation. This fork adds:
- Features not yet in upstream that are needed for production use
- Bug fixes waiting to be PRed upstream
- Integration points for the [paperclip-openrouter-adapter](https://github.com/JayCodesX/paperclip-openrouter-adapter)

## Branch Strategy

| Branch | Purpose |
|---|---|
| `master` | Production base — upstream master + all our changes merged |
| `feat/*` | In-progress work |

**Never commit directly to `master`.** Always use a feature branch and merge in.

## Changes From Upstream

### feat: delete issues, comments, and runs ([feat/delete-issue-and-comments](https://github.com/JayCodesX/paperclip/tree/feat/delete-issue-and-comments))

**Why:** Agents occasionally produce bad output that gets saved as comments or runs on issues. Without the ability to delete, bad data accumulates and poisons future agent runs (agents read all comments on an issue as context).

**What changed:**
- `server/src/services/issues.ts` — cascade delete activity log, read states, comments before deleting issue
- `server/src/routes/issues.ts` — new `DELETE /issues/:id/comments/:commentId` endpoint
- `server/src/routes/agents.ts` — new `DELETE /heartbeat-runs/:runId` endpoint
- `server/src/services/heartbeat.ts` — `removeRun()` now cleans up activity log entries first (FK fix — was silently failing)
- `ui/src/components/CommentThread.tsx` — trash icon on comments (hover to reveal, confirmation popover)
- `ui/src/components/IssuesList.tsx` — trash icon on issue rows, `Set<string>` for concurrent delete tracking, error toasts
- `ui/src/pages/IssueDetail.tsx` — "Delete Issue" in more-menu, `onError` handlers on all delete mutations
- `ui/src/pages/AgentDetail.tsx` — trash icon on each run in the Runs tab, Reset Sessions removed from agent menu

**Upstream PR candidate:** Yes — the delete issue/comment/run features and the activity log FK fix are general enough for upstream.

**Status:** Tested locally, ready to test on production instance.

---

## Syncing With Upstream

When upstream releases a new version:

```bash
git fetch upstream
git checkout master
git merge upstream/master
# Resolve any conflicts (most likely in files we've touched — see Changes above)
git push origin master
```

### Files Most Likely to Conflict

These are the files we've changed that upstream is also likely to touch:

- `server/src/services/issues.ts`
- `server/src/routes/issues.ts`
- `server/src/routes/agents.ts`
- `ui/src/pages/IssueDetail.tsx`
- `ui/src/pages/AgentDetail.tsx`
- `ui/src/components/IssuesList.tsx`

When a conflict happens: keep our additions (the delete blocks) and accept upstream's other changes. Our changes are additive so conflicts are usually straightforward.

---

## Related Repos

| Repo | Purpose |
|---|---|
| [paperclip-openrouter-adapter](https://github.com/JayCodesX/paperclip-openrouter-adapter) | OpenRouter/DeepSeek adapter, installed into Paperclip via `install.sh` |
| [orager](https://github.com/JayCodesX/orager) | CLI agent runner with session management and MCP server for model chaining |

The adapter and orager are **not** committed into this fork — they are installed separately via `install.sh`. This keeps them cleanly separable and easier to PR upstream independently.
