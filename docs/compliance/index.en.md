# Compliance

Myeline is designed to support your compliance with the main
frameworks that apply to RAG solutions in professional environments.

- [GDPR](gdpr.md) — sub-processor registry, DPA, exercising rights,
  retention periods.
- [Data residency](data-residency.md) — where your data physically
  lives depending on the chosen edition.
- [ISO 27001 mapping](iso-27001-mapping.md) — correspondence between
  Annex A controls and the platform's native features (useful in
  audits).

## Frameworks supported by construction

| Framework               | Edition                 | Coverage                                        |
|-------------------------|-------------------------|-------------------------------------------------|
| **GDPR**                | All                     | Native (encryption, audit, rights, retention)   |
| **HDS (French health)** | Sovereign               | Compatible — since you host, your infra must be HDS-certified |
| **SecNumCloud**         | Sovereign               | Compatible by construction (air-gap)            |
| **NIS2 / French OIV**   | Sovereign               | Compatible — infra isolation matches the requirements |
| **ISO 27001 / 27701**   | All                     | Helped by mapping + audit log                   |
| **EU AI Act**           | All                     | Limited-risk AI system; transparency + usage log|

No edition "certifies" your compliance on its own — certification
applies to your organisation and its infrastructure, not to Myeline.
But Myeline doesn't create technical obstacles to audits.

## Reference documents

Available on request to [hello@myeline.io](mailto:hello@myeline.io):

- DPA (Data Processing Agreement) to sign with Myeline (in pure
  sovereign, the DPA scope is purely technical support — Myeline
  has no access to data)
- Security policy (Myeline PSSI)
- Pen test report (yearly, third-party)
- Article 30 GDPR processing-registry template, pre-filled for
  Myeline functionality
