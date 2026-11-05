# Contributing to Mellumo

Mellumo builds on Postiz. Prefer reusing existing application patterns and small changes over replacing working features.

## License

Submit contributions under the repository's AGPL-3.0 license and preserve existing notices. You must have the right to submit the work. Mellumo does not require signing the Postiz CLA or assigning copyright. The archived upstream contribution policy is historical, not the policy for this repository.

## Workflow

1. Search existing issues; describe a concrete problem or objective.
2. Create a focused branch and use pnpm 10.6.1 with the Node version in package.json.
3. Follow the existing DTO/controller/service/repository boundaries. Preserve internal package identifiers unless a change needs them.
4. Describe what changed and how you verified the affected path. Documentation/configuration changes need proportionate checks, not a complete product retest.
5. Open a pull request using the repository template. Include reproducible QA steps and any unverified behavior.

AI-assisted contributions are welcome when the contributor understands the change, checks it, and can explain the evidence. Generated claims, fabricated test results and unreviewed bulk submissions are not useful contributions.

Do not change a shipped Temporal workflow/activity contract in place. Preserve running histories by introducing versioned workflows when behavior requires it. Never submit credentials, user content, private logs, databases or paid provider assets.

## Support

Use repository issues for reproducible bugs, installation questions and scoped proposals. Use the private route in SECURITY.md for vulnerabilities. Community support is best effort; no support SLA or bounty is promised.
