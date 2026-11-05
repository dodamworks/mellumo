# Security policy

Mellumo is in foundation development. There is no supported production release or promised response SLA yet.

Report exploitable vulnerabilities privately to **team@dodam.pet**, with subject `Mellumo security`. GitHub private vulnerability reporting may also be used when enabled on this repository. Do not put vulnerabilities, credentials, customer data or live access tokens in public issues.

Include the affected commit/version, configuration, reproduction steps, observed impact and a minimal proof of concept. Use systems and accounts you control. Do not access other users' data or disrupt services.

Mellumo maintainers handle this derivative; Postiz maintainers do not provide support for Mellumo. Shared upstream issues can be coordinated with Postiz's published security process after triage. Do not assume the upstream CNA, disclosure platform or support timelines apply here.

For deployments, keep secrets outside Git and container build contexts; restrict database/Temporal access; preserve encryption keys and backups; and configure accounts and public URLs deliberately. If a key is exposed, revoke/rotate it: removing a file alone does not revoke a key.
