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
