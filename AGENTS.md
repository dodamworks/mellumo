# Mellumo development

Mellumo is an AGPL-3.0 derivative of Postiz. Keep LICENSE, NOTICE and upstream attribution intact. Use pnpm and preserve existing application features and internal identifiers.

The user selected `dodamworks/mellumo` and repository-local author `Dodam <269674695+dodamworks@users.noreply.github.com>` on 2026-11-05, explicitly replacing the initial private-email choice after GitHub GH007 rejected publication. Preserve that decision. This directory is outside the company-root routing paths: use an explicitly verified GitHub identity and target for each external operation; never infer identity from a shared active login or change global account settings.

Read docs/UPSTREAM.md and docs/DEVELOPMENT.md. Follow applicable upstream CLAUDE.md implementation patterns; its old Vite/NX references are superseded by actual package scripts. Mellumo's CONTRIBUTING.md and SECURITY.md replace the archived upstream community policies for this derivative.

Do not modify historical Temporal workflow/activity contracts in place. Avoid new features while establishing the foundation. Use checks proportionate to the change; do not rebuild a broad test program for unchanged upstream behavior.

`evaluation/`, `.local/`, credentials, keys, uploads and databases are local-only. Never commit them. Product source belongs at repository root. Public docs must use repository-relative links and must not contain machine-specific paths or local credentials.
