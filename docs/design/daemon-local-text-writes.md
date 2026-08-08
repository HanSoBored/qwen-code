# Daemon opt-in external text writes

## Decision

Mechanism (b): keep ACP write delegation ON (`writeTextFile: true` unchanged) and
widen the **bridge adapter's** write resolution via `QWEN_SERVE_EXTERNAL_WRITE_ROOTS`
(delimiter-separated absolute paths, default OFF). `createBridgeFileSystemAdapter`
falls back from the bound workspace to `resolveWithinWorkspace` against those roots
only for absolute paths the workspace rejects with `path_outside_workspace` /
`symlink_escape`. Relative inputs never fall back; paths outside workspace+roots
fail closed; in-workspace writes are byte-identical. Widening lives in the adapter,
never the WFS factory / HTTP routes.

## Protections KEPT (for opt-in roots)

Trust gate, symlink rejection / TOCTOU, size caps, atomic temp+rename with mode
preservation, and the WFS write audit — same `wfs.writeTextOverwrite` as
in-workspace writes. Approval (`ask`) still precedes landing.

## Deliberate gaps

No ignore-rule filtering on external roots; audit `relPath` is workspace-relative
(`pathHash` unaffected; see `QWEN_AUDIT_RAW_PATHS`); acp-http `qwen/file/write` /
`qwen/file/edit` unchanged. Per-workspace config (BW-4) deferred; env is global,
validated at boot (non-absolute entry fails the daemon loudly).

- A symlink INSIDE the workspace that points into a configured external root is
  rejected by the workspace resolve (`symlink_escape`) and then ALLOWED via the
  external-root fallback. Within the granted boundary but surprising — the agent
  writes "through" a symlink it thinks is workspace-local. Documented as intended.

- Audit on external writes: a successful external write first emits an `fs.denied`
  (failed workspace resolve) then `recordAccess`; attempts outside every root emit
  only the workspace denial with no explicit fallback-attempt signal.
