---
permalink: /plantuml-how-to/templates/
title: "PlantUML ArchiMate Templates: Copy-Paste Starting Points (2026)"
description: "Ready-made PlantUML ArchiMate templates: application landscape, technology layer, business view, migration view and system architecture diagrams. Copy, paste, adapt."
---

# PlantUML ArchiMate Templates: Copy-Paste Starting Points

Five templates covering the views I build most often. Every one renders with a current PlantUML out of the box — the ArchiMate standard library is bundled, no downloads needed. Adapt the element names, delete what you do not need, and keep the skeleton.

BY THE WAY:
**Platform Economies** — out now.  
[**Buy on Amazon**](https://www.amazon.com/dp/B0H3VVNPZ3) — Kindle and paperback. Also available in [🇬🇧 UK](https://www.amazon.co.uk/dp/B0H3VVNPZ3), [🇩🇪 DE](https://www.amazon.de/dp/B0H3VVNPZ3), [🇯🇵 JP](https://www.amazon.co.jp/dp/B0H3VVNPZ3), and [🇨🇦 CA](https://www.amazon.ca/dp/B0H3VVNPZ3).  
Book page: **[mohammed-brueckner.com/platform-economies](https://mohammed-brueckner.com/platform-economies/)**.

> 📚 New here? Start with the [PlantUML with ArchiMate Complete Guide](https://mohammed-brueckner.com/plantuml-how-to/) — setup, export, styling, and full annotated examples.

---

## Table of Contents

- [Template 1: Application Landscape](#template-1-application-landscape)
- [Template 2: Technology Layer](#template-2-technology-layer)
- [Template 3: Business View](#template-3-business-view)
- [Template 4: Migration View (Current → Target)](#template-4-migration-view-current--target)
- [Template 5: System Architecture Diagram](#template-5-system-architecture-diagram)
- [Usage Notes](#usage-notes)

---

## Template 1: Application Landscape

The classic "what applications do we have and who uses them" view. Application components, their services, and the business actors they serve.

```plantuml
@startuml
!include <archimate/Archimate>
left to right direction
!theme plain
!global $ARCH_SPECIAL_SHAPES = %true()
skinparam linetype ortho

title Application Landscape — Template

Business_Actor(customer, "Customer")
Business_Actor(agent, "Service Agent")

Grouping(core, "Core Applications") {
  Application_Component(portal, "Customer Portal")
  Application_Component(crm, "CRM Suite")
  Application_Component(billing, "Billing Engine")
}

Application_Service(portalApi, "Portal API")
Application_Service(crmSvc, "Customer Data Service")
Application_Service(billingSvc, "Invoicing Service")

Rel_Serving(portalApi, portal, "Serves")
Rel_Realization(portal, portalApi, "Realizes")
Rel_Realization(crm, crmSvc, "Realizes")
Rel_Realization(billing, billingSvc, "Realizes")

Rel_Serving(portal, customer, "Serves")
Rel_Serving(crm, agent, "Serves")
Rel_Flow(crm, billing, "Customer Billing Data")

@enduml
```

---

## Template 2: Technology Layer

Infrastructure reality: nodes, system software, networks. The view that reveals end-of-life databases.

```plantuml
@startuml
!include <archimate/Archimate>
left to right direction
!theme plain
!global $ARCH_SPECIAL_SHAPES = %true()
skinparam linetype ortho

title Technology Layer — Template

Grouping(dc, "Data Center / Cloud Region") {
  Technology_Node(appServer, "Application Server")
  Technology_Node(dbServer, "Database Server")
  Technology_SystemSoftware(jvm, "JVM Runtime")
  Technology_SystemSoftware(rdbms, "PostgreSQL")
  Technology_Artifact(appJar, "application.jar")
  Technology_Artifact(schema, "DB Schema")
}

Technology_CommunicationNetwork(lan, "Internal Network")
Technology_CommunicationNetwork(internet, "Internet")
Technology_Device(userDevice, "User Device")

Rel_Assignment(appServer, jvm, "Runs")
Rel_Assignment(dbServer, rdbms, "Runs")
Rel_Realization(appJar, jvm, "Runs on")
Rel_Realization(schema, rdbms, "Hosted by")
Rel_Flow(appServer, dbServer, "JDBC")
Rel_Association(appServer, lan, "Connected")
Rel_Flow(userDevice, internet, "HTTPS")
Rel_Flow(internet, appServer, "Exposes")

@enduml
```

---

## Template 3: Business View

The slide leadership reads: capabilities, processes, and the roles that own them.

```plantuml
@startuml
!include <archimate/Archimate>
left to right direction
!theme plain
!global $ARCH_SPECIAL_SHAPES = %true()
skinparam linetype ortho

title Business View — Template

Strategy_Capability(sales, "Sales")
Strategy_Capability(fulfillment, "Fulfillment")

Business_Role(accountMgr, "Account Manager")
Business_Role(warehouse, "Warehouse Operator")

Business_Process(quoteToCash, "Quote-to-Cash")
Business_Process(orderHandling, "Order Handling")
Business_Process(shipping, "Shipping")

Business_Object(order, "Customer Order")
Business_Object(invoice, "Invoice")

Rel_Assignment(accountMgr, quoteToCash, "Performs")
Rel_Assignment(warehouse, shipping, "Performs")
Rel_Triggering(quoteToCash, orderHandling, "Triggers")
Rel_Triggering(orderHandling, shipping, "Triggers")
Rel_Access(orderHandling, order, "Reads/Writes")
Rel_Access(quoteToCash, invoice, "Creates")
Rel_Aggregation(sales, quoteToCash, "Contains")
Rel_Aggregation(fulfillment, shipping, "Contains")

@enduml
```

---

## Template 4: Migration View (Current → Target)

One view per migration wave. Triggering and Flow relationships between plateaus make the program plan visible.

```plantuml
@startuml
!include <archimate/Archimate>
left to right direction
!theme plain
!global $ARCH_SPECIAL_SHAPES = %true()
skinparam linetype ortho

title Migration View — Template

Grouping(plateauNow, "Current State") {
  Application_Component(legacyApp, "Legacy Monolith")
  Technology_Node(onPrem, "On-Prem Server")
}

Grouping(plateauWave1, "Wave 1 — Rehost") {
  Application_Component(liftedApp, "Monolith on VM")
  Technology_Node(cloudVm, "Cloud VM")
}

Grouping(plateauTarget, "Target State") {
  Application_Component(serviceA, "Service A")
  Application_Component(serviceB, "Service B")
  Technology_Node(k8s, "Kubernetes Cluster")
}

Rel_Assignment(legacyApp, onPrem, "Runs on")
Rel_Assignment(liftedApp, cloudVm, "Runs on")
Rel_Assignment(serviceA, k8s, "Runs on")
Rel_Assignment(serviceB, k8s, "Runs on")

Rel_Triggering(legacyApp, liftedApp, "Wave 1: Lift & Shift")
Rel_Triggering(liftedApp, serviceA, "Wave 2: Decompose")
Rel_Triggering(liftedApp, serviceB, "Wave 2: Decompose")
Rel_Flow(serviceA, serviceB, "Events")

@enduml
```

---

## Template 5: System Architecture Diagram

For the "plantuml system architecture diagram" searches: a single-system view showing how one application decomposes across layers — business context, application internals, and the technology it runs on.

```plantuml
@startuml
!include <archimate/Archimate>
left to right direction
!theme plain
!global $ARCH_SPECIAL_SHAPES = %true()
skinparam linetype ortho

title System Architecture — Template

Business_Actor(user, "End User")
Business_Process(coreProcess, "Core Business Process")

Grouping(appLayer, "Application Layer") {
  Application_Component(frontend, "Web Frontend")
  Application_Component(backend, "API Backend")
  Application_Component(worker, "Background Worker")
  Application_DataObject(store, "Application Data")
}

Application_Service(api, "Public API")

Grouping(techLayer, "Technology Layer") {
  Technology_Node(container, "Container Host")
  Technology_Node(database, "Database Node")
  Technology_SystemSoftware(engine, "Container Runtime")
}

Rel_Serving(frontend, user, "Serves")
Rel_Serving(api, coreProcess, "Supports")
Rel_Realization(backend, api, "Realizes")
Rel_Flow(frontend, backend, "REST/JSON")
Rel_Flow(backend, worker, "Jobs")
Rel_Access(worker, store, "Reads/Writes")

Rel_Assignment(frontend, container, "Deployed on")
Rel_Assignment(backend, container, "Deployed on")
Rel_Assignment(worker, container, "Deployed on")
Rel_Assignment(container, engine, "Runs")
Rel_Flow(container, database, "SQL")

@enduml
```

---

## Usage Notes

**Start ugly, refine later.** Replace my element names with yours and render. A wrong diagram on screen beats a perfect diagram in your head.

**One view, one question.** If a template starts answering two questions at once, split it. Five small views age better than one cathedral.

**Keep the header block identical everywhere.** The `!theme plain` + `$ARCH_SPECIAL_SHAPES` + `linetype ortho` combination is the baseline that survives PlantUML updates. Style per-view below it, never inside the includes.

**Direction variants are your layout steering wheel.** Every relationship macro accepts `_Up`, `_Down`, `_Left`, `_Right` suffixes (e.g. `Rel_Flow_Right(a, b, "label")`). Use them when the automatic layout crosses lines — but if you need more than three or four, the diagram wants to be two diagrams.

---

## Further Reading

- [PlantUML with ArchiMate: Complete Guide](https://mohammed-brueckner.com/plantuml-how-to/) — the full guide these templates come from
- [PlantUML Troubleshooting](../troubleshooting/) — when a template refuses to render
- [PlantUML vs. Mermaid](../plantuml-vs-mermaid/) — if you are still choosing your tool
- [Architecture as Code: Why Your Diagrams Deserve Better](https://mohammed-brueckner.com/archimate/) — the case for all of this

---

**Last Updated:** August 2026
