# Third-party boundaries

The repository as a whole continues the Postiz AGPL-3.0 foundation. Root licensing does not grant access to every dependency, media asset, hosted API or trademark.

| Area | Current boundary |
|---|---|
| Postiz code | Preserve LICENSE, copyright notices, provenance and original ancestry. |
| Package dependencies | Preserve package/lock metadata and package-specific notices. A full dependency/asset license inventory has not been completed. |
| Polotno editor | Optional commercial SDK; its production license is separate. Keep its key unset unless separately licensed. [Official configuration](https://docs.postiz.com/self-host/configuration/polotno). |
| AI services/models | Operator supplies credentials and complies with provider/model terms and charges. No keys or access rights are included. |
| SNS APIs | Each operator needs the appropriate app registration, permissions and platform approvals. Source publication does not supply these permissions. |
| Logos, media and screenshots | Existing assets retain upstream provenance. Avoid representing Postiz branding as Mellumo's own mark or upstream endorsement. Review assets used in public marketing separately. |
| Historical CLA documents | Archived for provenance. They are not Mellumo contribution requirements. |

The upstream .gitmodules file references a public-api plugin path, but the imported baseline does not contain a tracked gitlink at that path. No claim is made that a separate optional plugin repository was imported or licensed here.
