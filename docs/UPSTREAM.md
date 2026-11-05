# Upstream provenance and maintenance

- Original project: [Postiz](https://github.com/gitroomhq/postiz-app).
- Imported baseline: `v2.25.3`, commit `a5affe62c74e39c1e7ac960d50d8418b807d532c`.
- Mellumo repository: https://github.com/dodamworks/mellumo.
- Mellumo-specific modifications start on 2026-11-05. Original Git ancestry and author records are preserved.
- The `upstream` remote is for fetching Postiz; its push URL is disabled locally. `origin` is Mellumo.

The baseline is deliberately the release analyzed locally. Newer upstream commits have not been merged automatically. Internal `@gitroom/*` imports, package names, database identifiers, workflow names and application behavior stay intact at this stage.

Original root README, contributor/security policies and CLA files are archived in `docs/upstream/`. Original GitHub workflows are preserved under `.github/upstream-workflows/`, outside the active workflow directory. They contain upstream publishing destinations and integration assumptions and are not Mellumo release automation.

To refresh: fetch upstream, select a reviewed release, create an update branch, merge while preserving Mellumo modifications, then check the affected paths. Never overwrite operational data or rewrite upstream attribution. Review security advisories and migration notes. Source updates and production deployment are separate operations.

No third-party claim in an inherited README is automatically a Mellumo feature, security guarantee, supported version promise or affiliation.
