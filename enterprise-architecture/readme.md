---
permalink: /enterprise-architecture/
title: "Enterprise Architecture That Survives Delivery"
description: "Practical EA guides on ArchiMate 4, AI agents, architecture as code, APIOps, and governance traceability from Mohammed Brückner."
lang: en
---

<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "CollectionPage",
  "@id": "https://mohammed-brueckner.com/enterprise-architecture/#page",
  "name": "Enterprise Architecture That Survives Contact with Delivery",
  "description": "Practical EA guides on ArchiMate 4, AI agents, architecture as code, APIOps, and governance traceability from Mohammed Brückner.",
  "url": "https://mohammed-brueckner.com/enterprise-architecture/",
  "inLanguage": "en",
  "author": { "@id": "https://mohammed-brueckner.com/#person" },
  "about": [
    { "@type": "Thing", "name": "Enterprise Architecture" },
    { "@type": "Thing", "name": "ArchiMate" },
    { "@type": "Thing", "name": "Architecture Governance" },
    { "@type": "Thing", "name": "Artificial Intelligence" }
  ],
  "mainEntity": {
    "@type": "ItemList",
    "itemListElement": [
      { "@type": "ListItem", "position": 1, "name": "Modeling AI Agents in ArchiMate 4", "url": "https://mohammed-brueckner.com/enterprise-architecture/archimate-ai-agents/" },
      { "@type": "ListItem", "position": 2, "name": "TOGAF, COBIT, ITIL 4, and NIST CSF 2.0", "url": "https://mohammed-brueckner.com/enterprise-architecture/togaf-cobit-itil-nist/" },
      { "@type": "ListItem", "position": 3, "name": "Architecture as Code", "url": "https://mohammed-brueckner.com/archimate/" },
      { "@type": "ListItem", "position": 4, "name": "Archify", "url": "https://mohammed-brueckner.com/archify/" }
    ]
  }
}
</script>

<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "BreadcrumbList",
  "itemListElement": [
    { "@type": "ListItem", "position": 1, "name": "Home", "item": "https://mohammed-brueckner.com/" },
    { "@type": "ListItem", "position": 2, "name": "Enterprise Architecture", "item": "https://mohammed-brueckner.com/enterprise-architecture/" }
  ]
}
</script>

# Enterprise Architecture That Survives Contact with Delivery

Enterprise architecture is useful when a team can use it to make a decision, ship that decision, and show what changed. Everything else is decoration.

I write about architecture that meets that test: models people can review, governance that ends with an owner and evidence, and AI systems whose accountability does not disappear behind an agent-shaped box.

That is also why I test the tools. In my [Archify run](https://mohammed-brueckner.com/archify/), the agent had to attach repository evidence to every node in an architecture diagram. The first 4,241-byte candidate passed all nine validation checks. The interesting part was not the polished HTML. It was the evidence trail, including the four things that broke on the way.

## Start with the problem in front of you

- **I need to model people and AI agents doing work together.** Read [Modeling AI Agents in ArchiMate 4](archimate-ai-agents/). It separates software behavior, assigned roles, and human accountability.
- **I need one line of sight from strategy to operations.** Use the [TOGAF, COBIT, ITIL 4, and NIST CSF 2.0 traceability method](togaf-cobit-itil-nist/). It connects a business objective to architecture, a governance decision, service operation, cybersecurity outcomes, and evidence.
- **I need diagrams that do not become decorative fiction.** Start with [Architecture as Code](https://mohammed-brueckner.com/archimate/), then see how [Archify](https://mohammed-brueckner.com/archify/) adds repository evidence to generated diagrams.

## Model and communicate

**[Architecture as Code](https://mohammed-brueckner.com/archimate/)** makes the case for treating architecture diagrams as reviewable text rather than binary drawings. The win is not prettier output. It is a diff somebody can challenge before the wrong dependency becomes documentation.

**[PlantUML with ArchiMate](https://mohammed-brueckner.com/plantuml-how-to/)** is the practical reference: setup, export, styling, business views, Archi integration, and the errors that otherwise consume an afternoon. Keep [the templates](https://mohammed-brueckner.com/plantuml-how-to/templates/) nearby when a blank page is the obstacle.

**[Archify](https://mohammed-brueckner.com/archify/)** addresses the next failure mode. Text in Git is still only somebody's memory. Archify asks an agent to anchor diagram nodes to source files, then validates the result before rendering a portable artifact.

**[Building jArchi from source](https://mohammed-brueckner.com/jArchi-Build/)** is the narrow, verified procedure for getting the Archi scripting plugin built when the ready-made path is not available.

## Govern and operate

**[TOGAF, COBIT, ITIL 4, and NIST CSF 2.0](togaf-cobit-itil-nist/)** is not an attempt to crown one framework or collapse four vocabularies into a spreadsheet. It is a traceability canvas for following one decision across architecture, governance, service management, and cybersecurity.

**[APIOps as Code](https://mohammed-brueckner.com/apiops/)** applies the same discipline to APIs: versioned specifications, automated checks, publication, operation, and retirement. An API lifecycle that depends on somebody remembering the wiki page is not a lifecycle. It is a hope with endpoints.

**[Integration in a Serverless World](https://mohammed-brueckner.com/integration/)** connects the enterprise view to event-driven and low-code integration choices. Architecture remains necessary when the implementation becomes easier. Especially then.

## AI and accountability

**[Modeling AI Agents in ArchiMate 4](archimate-ai-agents/)** shows why an AI component performing a task is not the same thing as that component owning the outcome. The model needs both the assignment and the accountable human or organization.

**[From Discovery to Deployment with AI](https://mohammed-brueckner.com/how-to-ai/)** looks at the adoption work around the model: discovery, delivery, organizational friction, and the gap between activity and value.

**[The Threshold Test](https://mohammed-brueckner.com/threshold-test/)** gives you five questions for deciding whether an AI proposal clears the bar for investment. Use it before the pilot acquires a logo and a steering committee.

**[What Is a Platform Economy?](https://mohammed-brueckner.com/platform-economy/)** zooms out to the market forces around those choices: platforms, infrastructure, features, and the compression of the middle ground.

## The working rules behind these guides

1. **A model is a claim.** Attach evidence, or label the uncertainty.
2. **A role is not a legal owner.** Record accountability where governance can enforce it.
3. **A diagram is not finished when it renders.** Check the relationships, the reader, and the source of truth.
4. **A control without evidence is a wish.** Decide what will prove it operates before calling it implemented.
5. **A framework is a means, not the deliverable.** Keep the decision and its trace; discard the ceremony that helps neither.

---

**About the author:** Mohammed Brückner is an enterprise architect and author in Munich. He writes about architecture that connects business strategy to delivery, and runs workshops on enterprise architecture, platform economies, and enterprise AI. See [Speaking & Workshops](https://mohammed-brueckner.com/speaking/) or the book *[IT's not magic, it's architecture](https://www.amazon.com/-/de/dp/B0CVZ1BWPN)*.
