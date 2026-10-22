---
permalink: /enterprise-architecture/togaf-cobit-itil-nist/
title: "TOGAF, COBIT, ITIL 4, and NIST CSF 2.0 Traceability"
description: "Connect business objectives, architecture, governance, IT services, cybersecurity outcomes, and operating evidence without pretending frameworks are interchangeable."
lang: en
---

<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "TechArticle",
  "headline": "TOGAF, COBIT, ITIL 4, and NIST CSF 2.0: A Practical Traceability Method",
  "description": "Connect business objectives, architecture, governance, IT services, cybersecurity outcomes, and operating evidence without pretending frameworks are interchangeable.",
  "url": "https://mohammed-brueckner.com/enterprise-architecture/togaf-cobit-itil-nist/",
  "inLanguage": "en",
  "proficiencyLevel": "Intermediate",
  "datePublished": "2026-10-22",
  "dateModified": "2026-10-22",
  "author": { "@id": "https://mohammed-brueckner.com/#person" },
  "isPartOf": { "@id": "https://mohammed-brueckner.com/enterprise-architecture/#page" },
  "mainEntityOfPage": { "@type": "WebPage", "@id": "https://mohammed-brueckner.com/enterprise-architecture/togaf-cobit-itil-nist/" },
  "about": [
    { "@type": "Thing", "name": "TOGAF" },
    { "@type": "Thing", "name": "COBIT 2019" },
    { "@type": "Thing", "name": "ITIL 4" },
    { "@type": "Thing", "name": "NIST Cybersecurity Framework 2.0" },
    { "@type": "Thing", "name": "Architecture Governance" }
  ],
  "citation": [
    "https://www.opengroup.org/togaf/fundamental-content",
    "https://www.isaca.org/resources/cobit",
    "https://www.peoplecert.org/browse-certifications/it-governance-and-service-management/ITIL-1/itil-4-specialist-plan-implement-and-control-3804",
    "https://www.nist.gov/cyberframework",
    "https://nvlpubs.nist.gov/nistpubs/CSWP/NIST.CSWP.29.pdf"
  ]
}
</script>

<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "BreadcrumbList",
  "itemListElement": [
    { "@type": "ListItem", "position": 1, "name": "Home", "item": "https://mohammed-brueckner.com/" },
    { "@type": "ListItem", "position": 2, "name": "Enterprise Architecture", "item": "https://mohammed-brueckner.com/enterprise-architecture/" },
    { "@type": "ListItem", "position": 3, "name": "Framework Traceability", "item": "https://mohammed-brueckner.com/enterprise-architecture/togaf-cobit-itil-nist/" }
  ]
}
</script>

# TOGAF, COBIT, ITIL 4, and NIST CSF 2.0: A Practical Traceability Method

**TOGAF structures architecture work. COBIT assigns governance and management accountability. ITIL 4 connects the result to service operation. NIST CSF 2.0 supplies cybersecurity outcomes.** Use them together by tracing one decision through their different responsibilities, not by forcing their vocabularies into a one-to-one mapping.

That distinction saves you from the familiar framework spreadsheet: four columns of loosely related terms, no decision owner, and no evidence that anything changed after the workshop.

The method below is my synthesis, not an official crosswalk from The Open Group, ISACA, PeopleCert, or NIST. It starts with a business objective and ends with operating evidence. The frameworks help along the way; none of them owns the whole chain.

## Give each framework one job

| Framework | Use it to answer | Do not ask it to prove |
|---|---|---|
| **TOGAF Standard, 10th Edition** | What architecture work is needed, in which ADM phase, producing which artifact and decision? | That the organization assigned an owner or operates the result well |
| **COBIT 2019** | Which governance or management objective applies, and who is accountable for the decision and its risk? | That a technical control works or a service meets its targets |
| **ITIL 4** | Which service-management practice and service relationship must carry the decision into operation? | That the architecture is strategically sound |
| **NIST CSF 2.0** | Which cybersecurity outcome should the decision support, and what evidence would show progress toward it? | That a COBIT or ITIL activity automatically satisfies the outcome |

These are working boundaries, not claims that the frameworks never overlap. They do overlap. The mistake is treating overlap as equivalence.

ISACA's official *Implementing the NIST Cybersecurity Framework Using COBIT 2019* publication maps COBIT to **CSF 1.1**. It is useful background, but it is not an official CSF 2.0 crosswalk. Record your own CSF 2.0 relationships and the reasoning behind them.

## The six-part traceability canvas

Create one record for a decision that matters. Six fields are enough to expose the usual gaps.

1. **Business objective.** The outcome the organization wants, with a named business owner.
2. **Architecture artifact.** The model, requirement, principle, roadmap item, or architecture decision that makes the change concrete.
3. **Governance decision and owner.** The decision taken, its authority, the relevant COBIT objective, and the person or body accountable for it.
4. **ITIL practice and service.** The practice that carries the decision into operation and the service affected by it.
5. **NIST CSF 2.0 outcome.** The specific outcome or category the decision supports.
6. **Operating evidence.** The artifact, record, test, or measurement that demonstrates operation rather than intention.

Keep the record readable on a phone. A stacked entry beats a six-column table that turns every field into a vertical strip of one-word lines.

The important relationship is directional: **objective → architecture → decision → operation → outcome → evidence**. If a field is empty, do not hide the gap with a framework logo.

## A worked trace: customer onboarding access

This example is illustrative. It is not a client case or a claim about a production system.

A company wants customers to activate onboarding accounts without shared credentials or unnecessary manual access reviews. The architecture introduces an identity service and human review for exceptions.

### 1. Business objective

**Reduce onboarding friction while keeping account access attributable to an individual customer.**

The business owner is the onboarding product owner. “Improve security” would be too vague to guide a decision or produce useful evidence.

### 2. Architecture artifact

An architecture decision record establishes these rules:

- Every account belongs to an identifiable person.
- Shared customer credentials are prohibited.
- The identity service is the authoritative source for authentication decisions.
- Exceptions require human review and an expiry date.

A simple ArchiMate view can connect the onboarding process, application components, application services, and requirements. The [AI-agent modeling guide](../archimate-ai-agents/) covers the related accountability problem when software performs part of the work.

### 3. Governance decision and owner

The governance decision approves the identity policy, the prohibited shared-account pattern, and the residual-risk owner.

Relevant COBIT 2019 objectives could include **EDM03 Ensured Risk Optimization** for governance oversight of risk and **APO12 Managed Risk** for management of the risk response. These are candidate objectives for this scenario, not universal answers and not a one-to-one mapping to the other frameworks.

Record the accountable executive, the management owner, the decision date, and the conditions that require reconsideration. “Security owns it” is not an owner.

### 4. ITIL 4 practice and service

The affected service is customer onboarding. Two ITIL 4 practices are directly useful in this trace:

- **Change Enablement** coordinates assessment and authorization of the service change.
- **Service Configuration Management** maintains the relationships among the identity service, onboarding service, and their configuration items.

Other practices may participate. Add them when they contribute a decision or evidence, not to make the canvas look comprehensive.

### 5. NIST CSF 2.0 outcome

Two relevant CSF 2.0 categories are:

- **GV.RR — Roles and Responsibilities:** cybersecurity roles, responsibilities, and authorities are established, communicated, understood, and enforced.
- **PR.AA — Identity Management, Authentication and Access Control:** access to assets is managed through identity, authentication, and access-control outcomes.

The architecture and governance records should support these outcomes. They do not prove them. That is what the sixth field is for.

### 6. Operating evidence

A reviewer should be able to inspect:

- The approved architecture decision and named accountable owners.
- The authorized change record for the onboarding service.
- A current configuration record connecting onboarding to the identity service.
- Test results showing that shared credentials cannot be created or used.
- Access-review records, including exceptions and their expiry dates.
- Authentication and administrative-action logs tied to individual identities.

The exact evidence depends on the implemented controls. Decide what would falsify the claim “access is attributable” before declaring success. A screenshot of the architecture tool will not do it.

## Where framework integrations fail

**The mapping becomes the deliverable.** A giant matrix is easier to review than an actual service, so the matrix wins. Stop at decisions that need a trace.

**Every overlap becomes an equivalence.** COBIT risk governance, ITIL change work, and CSF outcomes are related. They are not interchangeable units.

**Owners become departments.** A governance record needs a role with authority and a name attached in the operating system of record. “Architecture” and “Security” are not accountable people.

**Evidence means documentation.** A policy proves that somebody wrote a policy. Look for configuration, test, review, and operational records too.

**The version disappears.** This guide deliberately pins its service-management examples to **ITIL 4** and its cybersecurity examples to **NIST CSF 2.0**. Keep versions in your register. Do not silently mix terminology from different releases.

**The canvas expands before it works.** Twenty objectives and thirty outcomes create a reporting program, not traceability. Begin with one material decision.

## A minimal adoption sequence

1. **Pick one live decision.** Choose something with a business owner, an architecture change, and an operational consequence.
2. **Write the objective and evidence first.** This prevents framework terminology from substituting for intent.
3. **Add the architecture artifact.** Link the maintained model or decision record, not an exported slide with no owner.
4. **Record governance explicitly.** Name the decision authority, risk owner, and relevant COBIT objective.
5. **Connect service operation.** Identify the affected service and only the ITIL practices that carry real work.
6. **Select the CSF outcome.** Explain why it applies. Keep the rationale beside the link.
7. **Review the evidence with operations.** Remove claims that the available records cannot support.

One complete trace teaches more than a blank enterprise-wide template. Scale only after the first record survives a review by the people who actually run the service.

## What not to automate yet

Do not generate a universal crosswalk with an LLM and call it governance. Do not infer accountability from a framework label. Do not turn “a document exists” into a passing control check. Do not auto-populate CSF outcomes from keywords without recording a reviewer and rationale.

Automation becomes useful later: checking that required fields are present, detecting expired exceptions, linking to authoritative records, and flagging missing evidence. First make one trace correct enough that automation has something worth preserving.

---

## Official references

- The Open Group, *[TOGAF Fundamental Content](https://www.opengroup.org/togaf/fundamental-content)* — the current TOGAF Standard, 10th Edition entry point.
- ISACA, *[COBIT](https://www.isaca.org/resources/cobit)* — COBIT 2019 framework and implementation resources. The same page identifies the official NIST mapping as CSF 1.1.
- PeopleCert, *[ITIL 4 Specialist: Plan, Implement and Control](https://www.peoplecert.org/browse-certifications/it-governance-and-service-management/ITIL-1/itil-4-specialist-plan-implement-and-control-3804)* — official coverage of Change Enablement and Service Configuration Management.
- NIST, *[Cybersecurity Framework](https://www.nist.gov/cyberframework)* and *[The NIST Cybersecurity Framework 2.0](https://nvlpubs.nist.gov/nistpubs/CSWP/NIST.CSWP.29.pdf)* — the framework and its outcome taxonomy.

## Related guides

- [Enterprise Architecture](..) — the connected EA guides
- [Modeling AI Agents in ArchiMate 4](../archimate-ai-agents/) — execution, roles, and human accountability
- [Architecture as Code](../../archimate/) — reviewable models and diagrams
- [APIOps as Code](../../apiops/) — moving API decisions through a managed lifecycle
- [Archify](../../archify/) — attaching repository evidence to generated diagrams

**About the author:** Mohammed Brückner is an enterprise architect and author in Munich, working on IT strategy, enterprise architecture, and digital transformation. More connected guides live on the [Enterprise Architecture hub](..).
