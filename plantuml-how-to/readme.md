# PlantUML with ArchiMate: Complete Guide

A comprehensive guide for using PlantUML with ArchiMate extensions for enterprise architecture modeling.

&gt; 📚 **Explore the author's publications:** [mohammed-brueckner.com/publications](https://mohammed-brueckner.com/publications) — featuring [*IT's not magic, it's architecture*](https://www.amazon.com/dp/B0CVZ1BWPN) (IT leadership & enterprise architecture) and [*Machine Learning Operations (MLOps) with Databricks on Azure End-to-End*](https://www.amazon.com/dp/B0FTSY78DR) (production-grade MLOps systems).

---

## Table of Contents

- [Introduction](#introduction)
- [Quick Start Examples](#quick-start-examples)
- [Local Setup Guide](#local-setup-guide)
- [Usage and Export](#usage-and-export)
- [Advanced Features](#advanced-features)
- [Online Alternatives](#online-alternatives)
- [Integration with Archi](#integration-with-archi)
- [Styling and Customization](#styling-and-customization)
- [Business Domain Views](#business-domain-views)

---

## Introduction

PlantUML is a powerful tool for creating diagrams from plain text descriptions. When combined with ArchiMate extensions, it becomes an effective solution for enterprise architecture modeling. This guide covers both local and online usage scenarios.

If you want to master not only your diagrams but IT Architecture overall, check out:
[“IT’s not magic, it’s architecture”](https://www.amazon.com/dp/B0CVZ1BWPN) *today*!

For the full collection of architecture and MLOps resources, visit [mohammed-brueckner.com/publications](https://mohammed-brueckner.com/publications).

---

## Quick Start Examples

&gt; 💡 The cloud and ML pipeline patterns below align with concepts explored in [*Machine Learning Operations (MLOps) with Databricks on Azure End-to-End*](https://www.amazon.com/dp/B0FTSY78DR).

### Simple Example

```plantuml
@startuml
!include &lt;archimate/Archimate&gt;
left to right direction
!theme plain
!global $ARCH_SPECIAL_SHAPES = %true()
skinparam linetype ortho

title AWS E-Commerce Infrastructure Architecture (ArchiMate-Aligned)

' AWS Cloud Boundary - Encompassing cloud regions and ML
Grouping(awsCloud, "AWS Cloud") {
  ' Regions as Sub-Groupings
  Grouping(euWest1, "eu-west-1 (Ireland)") {
    Technology_Node(ec2, "Amazon EC2")
    Technology_Node(ecs, "Amazon ECS")
    Technology_Service(sqs, "Amazon SQS")
    Technology_Node(dynamodb, "Amazon DynamoDB")
    Rel_Assignment(ec2, ecs, "Hosts")
    Rel_Flow(ecs, sqs, "Queues to")
    Rel_Association(dynamodb, sqs, "Persists")
  }

  Grouping(euNorth1, "eu-north-1 (Stockholm)") {
    Technology_Node(lambda, "AWS Lambda")
    Technology_Node(serverless, "Serverless Compute")
    Technology_Node(nosql, "NoSQL Database Service")
    Technology_Service(collateralHandler, "Collateral Handler")
    Rel_Realization(lambda, serverless, "Executes")
    Rel_Serving(serverless, nosql, "Serves")
    Rel_Triggering(lambda, collateralHandler, "Triggers")
  }

  ' ML Pipeline Grouping
  Grouping(mlPipeline, "Machine Learning Pipeline") {
    Technology_Node(xgboost, "XGBoost Model")
    Technology_Artifact(modelTrainingData, "Model Training Data")
    Technology_Node(s3, "Amazon S3")
    Technology_Node(sagemaker, "Amazon SageMaker")
    Technology_Artifact(distributedBreakout, "Distributed Breakout.jar")
    Rel_Flow(modelTrainingData, s3, "Stores in")
    Rel_Realization(sagemaker, xgboost, "Trains")
    Rel_Assignment(sagemaker, distributedBreakout, "Deploys")
  }

  ' Inter-Region Flows within Cloud
  Rel_Flow(euWest1, euNorth1, "Cross-Region Sync")
  Rel_Flow(euNorth1, mlPipeline, "Integrates with ML")
}

' On-Prem / Local Development Boundary
Grouping(onPrem, "On-Prem (Local Development)") {
  Technology_Device(pc, "My PC (Windows 10 Pro)")
  Technology_SystemSoftware(oracleJDK, "Oracle JDK 11")
  Technology_SystemSoftware(javaSE, "Java SE")
  Rel_Assignment(oracleJDK, pc, "Installed on")
  Rel_Realization(javaSE, oracleJDK, "Uses")
}

' Cloud to On-Prem Flows
Rel_Flow(awsCloud, onPrem, "Deploys to Local")

' External Internet
Technology_CommunicationNetwork(internet, "Internet")
Rel_Flow(internet, awsCloud, "Exposes")

@enduml
```

### Complex Example with Annotations

**Note:** This example requires local PlantUML installation or [Kroki.io](https://kroki.io/) as it exceeds online service limits.

```plantuml
@startuml
!include &lt;archimate/Archimate&gt;
left to right direction
!theme plain
!global $ARCH_SPECIAL_SHAPES = %true()
skinparam linetype ortho
skinparam dpi 300
skinparam backgroundColor white
skinparam noteBackgroundColor #lightblue
skinparam noteBorderColor #blue

title AWS E-Commerce Infrastructure Architecture (ArchiMate-Aligned)

' AWS Cloud Boundary
Grouping(awsCloud, "AWS Cloud") {
  Grouping(euWest1, "eu-west-1\n(Ireland)") {
    Technology_Node(ec2, "Amazon EC2")
    Technology_Node(ecs, "Amazon ECS")
    Technology_Service(sqs, "Amazon SQS")
    Technology_Node(dynamodb, "Amazon DynamoDB")
    Rel_Assignment(ec2, ecs, "Hosts")
    Rel_Flow(ecs, sqs, "Queues to")
    Rel_Association(dynamodb, sqs, "Persists")
    note right of ec2
      Best Practice:
      - Use EC2 for
        scalable compute.
      - Auto-scale based
        on demand.
    end note
  }

  Grouping(euNorth1, "eu-north-1\n(Stockholm)") {
    Technology_Node(lambda, "AWS Lambda")
    Technology_Node(serverless, "Serverless Compute")
    Technology_Node(nosql, "NoSQL Database Service")
    Technology_Service(collateralHandler, "Collateral Handler")
    Rel_Realization(lambda, serverless, "Executes")
    Rel_Serving(serverless, nosql, "Serves")
    Rel_Triggering(lambda, collateralHandler, "Triggers")
    note right of lambda
      Best Practice:
      - Serverless for
        event-driven tasks.
      - Monitor cold starts.
    end note
  }

  Grouping(mlPipeline, "Machine Learning\nPipeline") {
    Technology_Artifact(modelTrainingData, "Model Training Data")
    Technology_Node(s3, "Amazon S3")
    Technology_Node(sagemaker, "Amazon SageMaker")
    Technology_Node(xgboost, "XGBoost Model")
    Technology_Artifact(distributedBreakout, "Distributed Breakout.jar")
    Rel_Flow(modelTrainingData, s3, "Stores in")
    Rel_Realization(sagemaker, xgboost, "Trains")
    Rel_Assignment(sagemaker, distributedBreakout, "Deploys")
    note right of sagemaker
      Best Practice:
      - Version models in
        S3.
      - Use SageMaker for
        managed training.
    end note
  }

  Rel_Flow(euWest1, euNorth1, "Cross-Region Sync")
  Rel_Flow(euNorth1, mlPipeline, "Integrates with ML")
  note right of awsCloud
    Overall:
    - Hybrid cloud setup
      ensures redundancy
      across regions.
  end note
}

Grouping(onPrem, "On-Prem\n(Local Development)") {
  Technology_Device(pc, "My PC\n(Windows 10 Pro)")
  Technology_SystemSoftware(oracleJDK, "Oracle JDK 11")
  Technology_SystemSoftware(javaSE, "Java SE")
  Rel_Assignment(oracleJDK, pc, "Installed on")
  Rel_Realization(javaSE, oracleJDK, "Uses")
  note left of pc
    Best Practice:
    - Secure local dev
      with VPN.
    - Sync code via Git.
  end note
}

Rel_Flow(awsCloud, onPrem, "Deploys to Local")

Technology_CommunicationNetwork(internet, "Internet")
Rel_Flow(internet, awsCloud, "Exposes")

@enduml
```

---

## Local Setup Guide

### Prerequisites

#### 1. Install Java (JRE) ☕

PlantUML requires Java Runtime Environment to function.

**Windows:**
- Download from [adoptium.net](https://adoptium.net) or [oracle.com](https://www.oracle.com/java/technologies/downloads/)

**Mac:**
```bash
brew install openjdk
```

**Linux:**
```bash
sudo apt install default-jre
```

**Verify Installation:**
```bash
java -version
```

#### 2. Download PlantUML JAR 📦

1. Visit [plantuml.com/download](https://plantuml.com/download)
2. Download the latest `.jar` file
3. Store in a permanent location with a short path (no spaces)

**Example Location:** `C:\plant\plantuml-1.2025.8.jar`

#### 3. Install Graphviz (Optional but Recommended) 📊

Required for certain diagram types (Class, State, Component diagrams).

- Download from [graphviz.org](https://graphviz.org)
- Verify: `dot -version`

---

## Visual Studio Code Setup

### Install Extension 🖥️

1. Open VS Code
2. Go to Extensions (`Ctrl+Shift+X` or `Cmd+Shift+X`)
3. Search for "PlantUML" by jebbs
4. Install the extension

### Configure Settings ⚡

Open Command Palette (`Ctrl+Shift+P`) → **Preferences: Open Settings (JSON)**

Add the following configuration:

```json
{
    "security.workspace.trust.untrustedFiles": "open",
    "workbench.colorTheme": "Monokai Dimmed",
    "workbench.editor.empty.hint": "hidden",

    "plantuml.jarPath": "C:\\plant\\plantuml-1.2025.8.jar",
    "plantuml.includepaths": "C:\\plant",
    "plantuml.render": "Local",
    "plantuml.exportFormat": "png",
    "plantuml.exportOutDir": "exports",
    "plantuml.commandArgs": [
        "-charset", "UTF-8",
        "-DPLANTUML_LIMIT_SIZE=32768"
    ]
}
```

**Note:** Update paths to match your local setup.

### Environment Variables ⚙️

For large diagrams, set these environment variables:

```bash
PLANTUML_LIMIT_SIZE=32768
PLANTUML_SECURITY_PROFILE=DEFAULT
```

---

## Usage and Export

### Quick Test ✅

1. Create `test.puml`:

```plantuml
@startuml
Alice -&gt; Bob: Hello
@enduml
```

2. Export the diagram
3. Verify PNG appears in `exports` directory

### Exporting Diagrams 📤

1. Open `.puml` file in VS Code
2. Preview with `Alt+D` (optional)
3. Command Palette (`Ctrl+Shift+P`)
4. Run: **PlantUML: Export Current Diagram**
5. Output saved to configured `exportOutDir`

### Auto-Export 🔄

VS Code extension doesn't auto-export on save. Options:

**VS Code Task:** Create custom task to run export command

**External File-Watcher:** Use file-watcher to execute:
```bash
java -jar plantuml.jar file.puml
```

---

## Advanced Features

&gt; 🏛️ Deepening your ArchiMate and enterprise architecture practice? [*IT's not magic, it's architecture*](https://www.amazon.com/dp/B0CVZ1BWPN) covers the principles and patterns that complement these modeling techniques.

### ArchiMate Library 🏛️

**Preferred Method (Built-in stdlib):**
```plantuml
!include &lt;archimate/Archimate&gt;
!theme archimate-standard from &lt;archimate/themes&gt;
```

**If Built-in Fails:**

1. Extract stdlib:
```bash
java -jar plantuml.jar -extractstdlib
```

2. Manual inclusion:
```plantuml
!include path/to/Archimate.puml
!theme archimate-standard from path/to/themes
```

### Syntax Rules ✍️

**Quotes:** Always use straight ASCII quotes (`"Customer"`), not curly quotes (`"Customer"`)

**Macros (CamelCase):**
```plantuml
Business_Actor(alias, "Label")
Application_Component(alias, "Label")
Technology_Interface(alias, "Label")
```

**Relationships:**
```plantuml
' Full Macro
Rel_Serving(a, b, "label")

' Shorthand
a -[serving]-&gt; b : label
```

### Diagram Layout & Styling 🎨

**Line Routing:**
```plantuml
skinparam linetype ortho
skinparam linetype polyline
```

**Direction:**
```plantuml
left to right direction
```

**Themes:**
```plantuml
!theme archimate-standard
!theme archimate-saturated
!theme archimate-lowsaturation
```

**Scaling:**
```plantuml
scale max 2000x1200
```

**Image Resolution:**
```plantuml
skinparam dpi 300
```

### Debugging 🔍

**Verbose Mode:**
```bash
java -jar plantuml.jar -v file.puml
```

**Preprocessor Output:**
```bash
java -jar plantuml.jar -preproc file.puml &gt; file.preproc
```

**Check Stdlib:**
```bash
jar tf plantuml.jar | findstr archimate
```

### Common Pitfalls ⚠️

- ✅ Normalize all quotes to straight ASCII
- ✅ Use local includes if built-in stdlib missing
- ✅ Remote includes may be blocked by security profiles
- ⚠️ If macros fail, PlantUML defaults to basic sequence diagram

---

## Online Alternatives

### Kroki.io

[Kroki.io](https://kroki.io/) is more reliable than PlantUML's online service, especially for complex diagrams.

**Basic Example:**
```plantuml
@startuml
archimate #Business "Customer" as customer
archimate #Business "Order Service" as service
archimate #Application "Order App" as app
service -up-&gt; customer : serves
app .up.&gt; service : realizes
@enduml
```

**Complex Example:**
```plantuml
@startuml
archimate #Business "Data & AI Strategy" as biz_strategy
archimate #Business "Analytics & BI" as biz_bi
archimate #Business "Data Engineering" as biz_de
archimate #Application "Databricks Workspace" as ws
archimate #Application "SQL Warehouses" as sqlwh
archimate #Application "Notebooks & Jobs" as apps_jobs
archimate #Application "Delta Live Tables" as dlt
archimate #Application "MLflow Tracking" as mlflow
archimate #Application "Unity Catalog" as uc
archimate #Application "Delta Lake (Bronze/Silver/Gold)" as delta &lt;&lt;DataObject&gt;&gt;

biz_bi -down-&gt; sqlwh : consumes insights
biz_de -down-&gt; dlt : defines pipelines
biz_strategy -down-&gt; ws : governs platform use
ws .down.&gt; apps_jobs : hosts
ws .down.&gt; sqlwh : provides
ws .down.&gt; uc : governs data access
apps_jobs .down.&gt; dlt : orchestrates
mlflow .down.&gt; apps_jobs : integrates
uc .down.&gt; delta : catalogs and policies
dlt .down.&gt; delta : writes curated tables
sqlwh .down.&gt; delta : queries
@enduml
```

**Note:** For large diagrams, local installation recommended as online services have size limits.

---

## Integration with Archi

&gt; 🔗 For more on enterprise architecture tooling, strategy, and IT leadership, see the [author's publications](https://mohammed-brueckner.com/publications).

### Archi vs. Enterprise Tools

**Archi:**
- ✅ Free, open-source
- ✅ Lightweight
- ✅ Ideal for personal/small-scale modeling
- ❌ Limited collaboration features
- ❌ No enterprise integrations

**Sparx EA (Commercial):**
- ✅ Scalable, feature-rich
- ✅ Repositories, versioning, team support
- ✅ Built for enterprise use
- ❌ Requires paid licenses

**Interoperability:**
- Models exchangeable via `.archimate` files
- Complex elements may not transfer perfectly
- Manual cleanup often needed

**Use Case:**
- **Archi:** Learning, pilots, individual work
- **Sparx EA/BiZZdesign/LeanIX:** Professional, large-scale architecture

### From PlantUML to Archi Model

#### Prerequisites

- Archi desktop tool (v4.9+)
- [jArchi Scripting Plugin](https://github.com/archimatetool/archi-scripting-plugin)
- PlantUML diagram with ArchiMate syntax

#### Step 1: Sample PlantUML Diagram

```plantuml
@startuml
!include &lt;archimate/Archimate&gt;
Technology_Device(firewall, "Firewall")
Technology_Node(loadBalancer, "Load Balancer")
Technology_Node(webServer, "Web Server")
Rel_Assignment(loadBalancer, webServer, "Balances")
Rel_Triggering(firewall, loadBalancer, "Protects")
@enduml
```

#### Step 2: Converted JSON Structure

```json
{
  "elements": [
    {
      "type": "TechnologyDevice",
      "id": "firewall",
      "name": "Firewall"
    },
    {
      "type": "TechnologyNode",
      "id": "loadBalancer",
      "name": "Load Balancer"
    },
    {
      "type": "TechnologyNode",
      "id": "webServer",
      "name": "Web Server"
    }
  ],
  "relationships": [
    {
      "type": "AssignmentRelationship",
      "source": "loadBalancer",
      "target": "webServer",
      "label": "Balances"
    },
    {
      "type": "TriggeringRelationship",
      "source": "firewall",
      "target": "loadBalancer",
      "label": "Protects"
    }
  ]
}
```

#### Step 3: jArchi Script

Paste this into Archi's scripting console:

```javascript
// Setup
var model = repository.getModels()[0];
var view = model.createView("archimate-diagram", "Imported View");

// JSON Data
var data = {
  "elements": [
    { "type": "TechnologyDevice", "id": "firewall", "name": "Firewall" },
    { "type": "TechnologyNode", "id": "loadBalancer", "name": "Load Balancer" },
    { "type": "TechnologyNode", "id": "webServer", "name": "Web Server" }
  ],
  "relationships": [
    { "type": "AssignmentRelationship", "source": "loadBalancer", 
      "target": "webServer", "label": "Balances" },
    { "type": "TriggeringRelationship", "source": "firewall", 
      "target": "loadBalancer", "label": "Protects" }
  ]
};

// Create Elements
var elementsById = {};
data.elements.forEach(function(e, i) {
  var element = model.createElement(e.type, e.name);
  elementsById[e.id] = element;
  view.add(element, 100 + i * 200, 100);
});

// Create Relationships
data.relationships.forEach(function(r) {
  var source = elementsById[r.source];
  var target = elementsById[r.target];
  if (source && target) {
    var rel = model.createRelationship(r.type, source, target, r.label || "");
    view.add(rel);
  }
});

console.log("Model created from PlantUML JSON.");
```

**Result:** Fully editable ArchiMate view in Archi with real elements and relationships.

---

## Miro Plugin

The Miro PlantUML plugin renders diagrams as images (not editable models).

### Installation

1. Go to Miro Marketplace
2. Search "PlantUML"
3. Install and authorize

### Usage

1. Open Miro board
2. Access "More apps" (+ icon)
3. Select PlantUML app
4. Enter code in editor
5. Preview updates in real-time
6. Click "Add to board"

**Note:** Creates static images, not interactive Miro objects.

---

## Styling and Customization

### Modern Style Sheets

**Note:** Requires latest PlantUML version (Nov 2025+). Older versions use deprecated `skinparam`.

```plantuml
@startuml
!include &lt;archimate/Archimate&gt;
left to right direction
!global $ARCH_SPECIAL_SHAPES = %true()
skinparam linetype ortho

&lt;style&gt;
  diagram {
    BackgroundColor #2E2E2E
  }
  archimate {
    FontColor black
    FontSize 10
    FontStyle bold
    Shadowing false
  }
  
  /* Style by ArchiMate element type */
  archimate.Technology_CommunicationNetwork {
    BackGroundColor #FF8C00 !important
    LineColor #FFFFFF !important
  }
  archimate.Technology_Device {
    BackGroundColor #DC143C !important
    LineColor #FFFFFF !important
  }
  archimate.Technology_Node {
    BackGroundColor #4169E1 !important
    LineColor #FFFFFF !important
  }
  archimate.Technology_SystemSoftware {
    BackGroundColor #3428deff !important
    LineColor #000000 !important
  }
  archimate.Technology_Artifact {
    BackGroundColor #FFD700 !important
    LineColor #000000 !important
  }
  archimate.Technology_Service {
    BackGroundColor #9370DB !important
    LineColor #FFFFFF !important
  }
  archimate.Technology_Path {
    BackGroundColor #00CED1 !important
    LineColor #000000 !important
  }
  
  grouping {
    BackGroundColor #eebe39b8
    LineColor #BBBBBB
    FontColor #1d13ebff
    FontStyle bold
    RoundCorner 15
    Shadowing true
  }
  
  relationship {
    LineColor #FF1493
    LineThickness 2
    FontColor #2600ffcd
    FontStyle bold
  }
  
  /* Fallback: Style by alias for guaranteed results */
  firewall, loadBalancer, webServer, vpnGateway {
    BackGroundColor #DC143C !important
  }
  dmzLan, secureNet, privateLan, internet {
    BackGroundColor #FF8C00 !important
  }
  appServer, database, backupServer {
    BackGroundColor #4169E1 !important
  }
&lt;/style&gt;

title E-Commerce Infrastructure-Focused Architecture

Technology_CommunicationNetwork(internet, "Internet")

Grouping(publicDMZ, "Public DMZ") {
  Technology_CommunicationNetwork(dmzLan, "DMZ LAN")
  Technology_Device(firewall, "Firewall")
  Technology_Node(loadBalancer, "Load Balancer")
  Technology_Device(webServer, "Web Server")
  Technology_SystemSoftware(waf, "WAF Software")
  Technology_Artifact(sslCert, "SSL/TLS Cert")
  Technology_Service(sslService, "SSL Service")
  
  Rel_Aggregation(dmzLan, firewall)
  Rel_Aggregation(dmzLan, loadBalancer)
  Rel_Aggregation(dmzLan, webServer)
  Rel_Triggering(firewall, loadBalancer, "Protects")
  Rel_Assignment(loadBalancer, webServer, "Balances")
  Rel_Assignment(waf, webServer, "Secures")
  Rel_Realization(sslService, sslCert, "Uses")
  Rel_Assignment(sslCert, webServer, "Configures")
}

Grouping(secureInterZone, "Secure Inter-Zone Network") {
  Technology_CommunicationNetwork(secureNet, "Secure Network")
  Technology_Device(vpnGateway, "VPN Gateway")
  Technology_Path(securePath, "Encrypted Path")
  
  Rel_Aggregation(secureNet, vpnGateway)
  Rel_Realization(securePath, secureNet)
  Rel_Association(securePath, vpnGateway, "Via")
}

Grouping(privateVNET, "Private VNET") {
  Technology_CommunicationNetwork(privateLan, "Private LAN")
  Technology_Node(appServer, "App Server")
  Technology_Node(database, "Database Server")
  Technology_Node(backupServer, "Backup Server")
  Technology_SystemSoftware(os, "OS (Linux)")
  Technology_SystemSoftware(dbms, "DBMS (PostgreSQL)")
  Technology_Artifact(backupPolicy, "Backup Policy")
  Technology_Service(monitoringService, "Monitoring Service")
  
  Rel_Aggregation(privateLan, appServer)
  Rel_Aggregation(privateLan, database)
  Rel_Aggregation(privateLan, backupServer)
  Rel_Assignment(os, appServer, "Hosts")
  Rel_Assignment(dbms, database, "Runs on")
  Rel_Assignment(backupServer, backupPolicy, "Applies")
  Rel_Serving(monitoringService, database, "Monitors")
  Rel_Association(appServer, database, "Connects to")
}

Rel_Flow(internet, publicDMZ, "Exposes")
Rel_Flow(publicDMZ, secureInterZone, "Routes through")
Rel_Flow(secureInterZone, privateVNET, "Secures")

@enduml
```

---

## Business Domain Views

&gt; 📊 Business architecture, capability mapping, and IT service design are core themes in [*IT's not magic, it's architecture*](https://www.amazon.com/dp/B0CVZ1BWPN).

When standard ArchiMate business components aren't available, use stereotype rectangles:

```plantuml
@startuml
left to right direction

skinparam rectangle {
  StereotypeFontColor #FFFFFF
  StereotypeFontSize 12
}
skinparam actor {
  StereotypeFontColor #FFFFFF
  StereotypeFontSize 12
}

skinparam rectangle&lt;&lt;Business_Capability&gt;&gt; {
  BackgroundColor #99FF99
  BorderColor #006600
  shadowing false
}
skinparam rectangle&lt;&lt;Business_Process&gt;&gt; {
  BackgroundColor #99CCFF
  BorderColor #003366
  shadowing false
}
skinparam rectangle&lt;&lt;Business_Service&gt;&gt; {
  BackgroundColor #FFD700
  BorderColor #B8860B
  shadowing false
}
skinparam actor&lt;&lt;Business_Role&gt;&gt; {
  BackgroundColor #FFB6C1
  BorderColor #8B008B
  shadowing false
}

title ServiceNow Incident Management Business Architecture

rectangle "IT Service Domain" as itServiceDomain {
  rectangle "User Verification Capability" as userVerification &lt;&lt;Business_Capability&gt;&gt;
  rectangle "Incident Retrieval Capability" as incidentRetrieval &lt;&lt;Business_Capability&gt;&gt;
  rectangle "Escalation Alerting Capability" as escalationAlerting &lt;&lt;Business_Capability&gt;&gt;
  
  rectangle "OAuth Authentication Process" as authenticationProcess &lt;&lt;Business_Process&gt;&gt;
  rectangle "User Details Fetch Process" as userFetchProcess &lt;&lt;Business_Process&gt;&gt;
  rectangle "Open Incidents Query Process" as incidentQueryProcess &lt;&lt;Business_Process&gt;&gt;
  rectangle "Alert Generation Process" as alertGenerationProcess &lt;&lt;Business_Process&gt;&gt;
  
  rectangle "API Orchestration Service" as apiOrchestrationService &lt;&lt;Business_Service&gt;&gt;
  
  actor "Incident Manager Role" as incidentManager &lt;&lt;Business_Role&gt;&gt;
}

actor "External User (Caller)" as externalUser
actor "ServiceNow System" as serviceNowSystem

' Capability to Process relationships
userVerification ..&gt; authenticationProcess : Supports
userVerification ..&gt; userFetchProcess : Supports
incidentRetrieval ..&gt; incidentQueryProcess : Supports
escalationAlerting ..&gt; alertGenerationProcess : Supports

' Role performing processes
incidentManager ..&gt; authenticationProcess : Performs
incidentManager ..&gt; userFetchProcess : Performs
incidentManager ..&gt; incidentQueryProcess : Performs
incidentManager ..&gt; alertGenerationProcess : Performs

' Service orchestration
apiOrchestrationService ..&gt; authenticationProcess : Orchestrates
apiOrchestrationService ..&gt; userFetchProcess : Orchestrates
apiOrchestrationService ..&gt; incidentQueryProcess : Orchestrates
apiOrchestrationService ..&gt; alertGenerationProcess : Orchestrates

' Process flow
externalUser --&gt; authenticationProcess : Initiates
authenticationProcess --&gt; userFetchProcess : Triggers
userFetchProcess --&gt; incidentQueryProcess : Triggers
incidentQueryProcess --&gt; alertGenerationProcess : Triggers

' Capability dependencies
userVerification ..&gt; incidentRetrieval : Enables
incidentRetrieval ..&gt; escalationAlerting : Informs

' External system
serviceNowSystem ..&gt; apiOrchestrationService : Provides Data To

@enduml
```

---

## Resources

- [Author Publications — Mohammed Brückner](https://mohammed-brueckner.com/publications) — IT architecture, MLOps, and enterprise integration resources
- [PlantUML Official Site](https://plantuml.com)
- [PlantUML Download](https://plantuml.com/download)
- [Graphviz](https://graphviz.org)
- [Kroki.io](https://kroki.io/)
- [Archi Tool](https://www.archimatetool.com/)
- [jArchi Plugin](https://github.com/archimatetool/archi-scripting-plugin)
- [VS Code PlantUML Extension](https://marketplace.visualstudio.com/items?itemName=jebbs.plantuml)

---

**Last Updated:** November 2025
