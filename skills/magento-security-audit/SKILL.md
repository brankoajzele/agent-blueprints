---
name: magento-security-audit
description: Audit Magento security, plan hardening, or investigate suspected compromise. Use for explicit security assessments or incident work, rather than routine implementation and general code review.
---

# Magento security audit

## Establish scope and evidence

Identify the environment, edition/version, patch level, exposed interfaces, and requested assessment. Use current Adobe advisories for vulnerability claims and identify affected packages/versions. Run only scans covered by the authorization; a code audit does not imply permission for live probing or hardening changes.

## Assess the relevant surface

- Check dependency advisories with available tooling such as `composer audit`, and patch/core integrity against trusted release artifacts. State scanner coverage and limitations; a clean dependency scan does not exclude malicious code or application flaws.
- Trace authentication, ACLs, object ownership, request method/CSRF protection, output escaping, and parameter binding. Anonymous API resources can be intentional; assess exposed behavior and data rather than rejecting them categorically.
- Inspect secret handling, log contents, backups, runtime permissions, and document-root exposure. PHP must be able to read required configuration, including `app/etc/env.php`; restrict access to intended runtime/owner identities and prevent public HTTP access. Avoid blanket chmod recipes.
- Assess Admin MFA and access controls, integration credential scope, supported TLS configuration, and transport headers in the actual deployment. A custom Admin path is defense in depth, not a replacement for authentication or evidence of a critical vulnerability by itself.
- Configure application CSP through supported module configuration, `csp_whitelist.xml`, and rendering/nonce mechanisms as appropriate to the release. Test payment and third-party integrations before tightening enforcement.

## Report or respond

For each finding, give evidence, affected scope, exploit conditions, impact-based severity, and a concrete remediation with verification. Separate confirmed vulnerabilities from hardening suggestions. Do not certify PCI/GDPR compliance from automated checks.

For suspected compromise, preserve relevant evidence and coordinate containment before changing files, rotating keys, or stopping jobs. Plan credential and encryption-key changes separately, accounting for dependent data and integrations. Carry out only the authorized response actions and record their outcome.

Consult Adobe's [security advisories](https://www.adobe.com/trust/security/bulletins-and-advisories.html), [authorization guidance](https://developer.adobe.com/commerce/php/development/security/authorization), and [CSP documentation](https://developer.adobe.com/commerce/php/development/security/content-security-policies) for the affected release.
