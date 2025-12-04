# OpenHands Prompts for SharePoint Modernization

## Background Context

These prompts are designed for evaluating OpenHands on legacy SharePoint modernization projects. These prompts are only suggestions. Feel free to adapt or change anything about them to further meet the needs of the project. If output is incomplete on the first run, try breaking into smaller focused prompts or providing additional context about the specific repository structure.

## Prompt Summaries

**Prompt 1: Repository Mapping & Architecture Discovery**
- Scans for technology stack (JS, C#, PowerShell, etc.)
- Identifies SharePoint-specific artifacts (AppManifest.xml, feature.xml, CSOM patterns)
- Generates an ARCHITECTURE_MAP.md with Mermaid diagrams showing component relationships and data flow

**Prompt 2: Deprecated SharePoint Connector Analysis**
- Comprehensive pattern detection for CSOM, Add-In model, legacy SP.js APIs, and ACS authentication
- Generates MIGRATION_PLAN.md with phased implementation (14-week timeline)
- Includes before/after code transformation examples (CSOM → PnP Framework)
- Risk register and testing strategy

**Prompt 3: Combined Single-Pass Analysis**
- Produces 5 deliverable files in one run: repo analysis, architecture diagrams, deprecated inventory, migration plan, and quick reference guide
- Includes actual grep/find commands for pattern matching
- Designed as a template that can be batch-run across 40+ repositories

## Usage Notes

**Batch Processing:**
For analyzing multiple repositories, consider running Prompt 3 (combined) on each repo individually with the CLI to not overwhelm the AI model.

**Output Review:**
- Review generated Mermaid diagrams in a markdown previewer
- Validate deprecated component counts against manual spot-checks
- Use findings to prioritize repositories by migration complexity

**Customization:**
These prompts can be adapted for:
- Different SharePoint versions (2013/2016/2019/Online)
- Specific migration targets (pure SPFx vs hybrid)
- Different timeline constraints
- Varying team expertise levels

---

## Prompt 1: Repository Mapping & Architecture Discovery

```
You are analyzing a legacy SharePoint intranet codebase that must be modernized before the SharePoint Add-In model retirement on April 2, 2026. Your task is to perform comprehensive repository mapping and generate architecture documentation.

### Primary Objectives

1. **Repository Structure Analysis**
   - Scan all directories and identify the technology stack (JavaScript, TypeScript, C#, PowerShell, etc.)
   - Identify build systems (webpack, MSBuild, npm, NuGet, etc.)
   - Map all configuration files and their purposes
   - Document the folder structure and module organization

2. **SharePoint-Specific Discovery**
   - Locate all SharePoint Add-In manifest files (AppManifest.xml, elements.xml, feature.xml)
   - Identify CSOM (Client-Side Object Model) usage patterns
   - Find all SharePoint REST API calls
   - Document SharePoint feature definitions and web part configurations
   - Identify provider-hosted vs SharePoint-hosted components

3. **Dependency Mapping**
   - Extract all package.json dependencies and their versions
   - Extract all NuGet package references
   - Identify internal module dependencies
   - Map external API integrations and service connections
   - Flag any deprecated or end-of-life dependencies

4. **Generate Architecture Documentation**

Create a file called ARCHITECTURE_MAP.md containing:

#### Section 1: Executive Summary
- Brief overview of the repository purpose
- Technology stack summary
- Estimated migration complexity (LOW/MEDIUM/HIGH/CRITICAL)

#### Section 2: Component Diagram (Mermaid)
Generate a Mermaid diagram showing:
- Major modules/components
- Data flow between components
- External service integrations
- SharePoint touchpoints

Example structure:

[Mermaid flowchart showing:]
- Client Layer: Web Parts, JavaScript Modules
- SharePoint Integration: CSOM Calls, REST API, Add-In Model
- Backend Services: Custom APIs, Data Storage
- Arrows showing data flow between layers

#### Section 3: File Tree with Annotations
List key directories and files with brief descriptions of their purpose.

#### Section 4: Deprecated Component Inventory
Create a table listing:
| File Path | Deprecated Component | Type | Migration Priority |

Types include: CSOM, Add-In Manifest, Legacy Web Part, Deprecated API, etc.

#### Section 5: Integration Points
Document all external touchpoints:
- SharePoint Online APIs
- Microsoft Graph API usage (if any)
- Azure services
- Third-party APIs
- CDN references

#### Section 6: Security Observations
Flag any security concerns found:
- Hardcoded credentials or API keys
- Insecure authentication patterns
- Exposed secrets in config files

### Output Requirements
- Save all findings to ARCHITECTURE_MAP.md in the repository root
- Use clear, scannable formatting
- Include line numbers or file paths for all findings
- Ensure Mermaid diagrams render correctly in GitHub/GitLab

### Analysis Approach
1. Start with package.json, *.csproj, and config files to understand the stack
2. Search for SharePoint-specific patterns: "ClientContext", "SP.", "SPFx", "AppManifest"
3. Trace data flow from UI components to backend services
4. Document as you discover - don't wait until the end
```

---

## Prompt 2: Deprecated SharePoint Connector Analysis & Migration Plan

```
You are a SharePoint modernization specialist analyzing a legacy codebase for migration from the SharePoint Add-In model to SPFx (SharePoint Framework). Generate a comprehensive implementation plan for modernizing this repository.

### Background

Microsoft is retiring the SharePoint Add-In model on April 2, 2026. All components using this model must be migrated to:
- SPFx (SharePoint Framework) for client-side components
- PnP Framework + Microsoft Graph API for backend operations (replacing CSOM)
- Microsoft Entra ID for authentication (replacing legacy SharePoint auth)

### Analysis Tasks

#### Phase 1: Deprecated Component Detection

Search the codebase for these patterns and document ALL occurrences:

**CSOM (Client-Side Object Model) - Must migrate to PnP/Graph:**
- ClientContext
- Microsoft.SharePoint.Client
- SP.ClientContext
- .ExecuteQuery() / .ExecuteQueryAsync()
- Web.Lists / Web.Webs / Web.SiteUsers

**SharePoint Add-In Model - Must migrate to SPFx:**
- AppManifest.xml
- SharePointProjectItem.spdata
- elements.xml with Add-In references
- AppEventReceiver
- RemoteEventReceiver
- TokenHelper / SharePointContext

**Legacy JavaScript APIs - Must migrate to SPFx patterns:**
- SP.js / SP.Runtime.js / SP.Core.js
- _spPageContextInfo
- SP.SOD (Script on Demand)
- ExecuteOrDelayUntilScriptLoaded
- SP.UI.ModalDialog
- SP.UI.Notify

**Legacy Web Parts:**
- .webpart files
- .dwp files
- <AllUsersWebPart> references

**Legacy Authentication:**
- SPHostUrl / SPAppWebUrl patterns
- SharePointContextToken
- ACS (Access Control Service) tokens

#### Phase 2: Generate Migration Plan

Create a file called MIGRATION_PLAN.md containing:

### Section 1: Migration Scope Summary

Provide counts and complexity assessment:
- Total files requiring modification
- CSOM call sites to migrate
- Add-In components to replace
- Web parts requiring SPFx conversion
- Estimated person-hours (rough order of magnitude)

### Section 2: Migration Dependency Graph (Mermaid)

Create a Mermaid diagram showing migration paths:
- Current State: CSOM Calls, Add-In Model, Legacy SP.js, ACS Auth
- Target State: PnP Framework, Microsoft Graph, SPFx Components, Entra ID Auth
- Arrows labeled with migration action (Migrate/Rewrite/Replace)

### Section 3: Detailed Migration Tasks

For each deprecated component found, create an entry:

#### Task [ID]: [Component Name]
- **File(s):** [list of affected files with line numbers]
- **Current Implementation:** [brief description]
- **Migration Target:** [SPFx / PnP / Graph / Entra ID]
- **Migration Steps:**
  1. [Step 1]
  2. [Step 2]
  3. [etc.]
- **Dependencies:** [other tasks that must complete first]
- **Risk Level:** [LOW/MEDIUM/HIGH]
- **Estimated Effort:** [hours]

### Section 4: Migration Sequence

Create a phased approach:

**Phase 1: Foundation (Weeks 1-2)**
- Set up SPFx development environment
- Configure Microsoft Entra ID app registration
- Establish PnP Framework integration

**Phase 2: Authentication Migration (Weeks 3-4)**
- Replace ACS tokens with Entra ID
- Update all authentication flows
- Test token acquisition

**Phase 3: Backend Migration (Weeks 5-8)**
- Migrate CSOM calls to PnP Framework
- Replace direct SharePoint calls with Graph API where appropriate
- Update error handling for new APIs

**Phase 4: UI Component Migration (Weeks 9-12)**
- Convert legacy web parts to SPFx
- Replace SP.js dependencies
- Implement modern UI patterns

**Phase 5: Testing & Validation (Weeks 13-14)**
- Integration testing
- Performance validation
- Security review

### Section 5: Code Transformation Examples

For the top 5 most common patterns found, provide before/after code examples.

Example - CSOM to PnP Framework:

Before (CSOM):
using (var context = new ClientContext(siteUrl))
{
    context.Credentials = credentials;
    var list = context.Web.Lists.GetByTitle("Documents");
    context.Load(list);
    context.ExecuteQuery();
}

After (PnP Framework):
using (var context = await PnPContext.CreateAsync(siteUrl))
{
    var list = await context.Web.Lists.GetByTitleAsync("Documents");
}

### Section 6: Risk Register

| Risk ID | Description | Likelihood | Impact | Mitigation |

### Section 7: Testing Strategy

- Unit test requirements for migrated components
- Integration test scenarios
- SharePoint Online tenant test plan
- Rollback procedures

### Output Requirements
- Save to MIGRATION_PLAN.md in repository root
- Include specific file paths and line numbers for all findings
- Provide actionable, step-by-step guidance
- Ensure estimates are conservative (add 30% buffer)
```

---

## Prompt 3: Combined Analysis (Single Comprehensive Pass)

```
You are performing a comprehensive modernization assessment of a legacy SharePoint intranet codebase.

Your goal is to create a complete analysis package that can be used as a template for similar modernization projects.

### Deliverables

Create the following files in the repository root:

#### 1. REPO_ANALYSIS.md - Complete Repository Map

Include:
- Technology stack inventory
- Directory structure with annotations
- Build system documentation
- Configuration file summary
- External dependency list with versions

#### 2. ARCHITECTURE_DIAGRAM.md - Visual Architecture

Include Mermaid diagrams for:
- Component relationships
- Data flow
- SharePoint integration points
- Service dependencies

Use red highlighting (style fill:#ff6b6b) for deprecated components requiring migration.

#### 3. DEPRECATED_INVENTORY.md - Deprecated Component Catalog

Create a searchable inventory with:

Summary Statistics:
- Total deprecated patterns found: [X]
- Files affected: [Y]
- Estimated migration effort: [Z] hours

Tables for each category:
- CSOM Usage (File, Line, Pattern, Migration Target)
- Add-In Model Components (File, Component Type, Description, Migration Target)
- Legacy JavaScript (File, Line, Pattern, Migration Target)
- Legacy Web Parts (File, Web Part Type, Description, Migration Target)

#### 4. MIGRATION_PLAN.md - Implementation Roadmap

Include:
- Phased migration approach
- Task breakdown with dependencies
- Effort estimates
- Risk assessment
- Code transformation examples
- Testing requirements

#### 5. QUICK_REFERENCE.md - Pattern Matching Guide

Create a reference for developers showing:
- Common deprecated patterns and their modern replacements
- Copy-paste code snippets for migration
- Links to relevant Microsoft documentation

### Search Patterns to Use

# CSOM patterns
grep -rn "ClientContext\|Microsoft\.SharePoint\.Client\|ExecuteQuery" --include="*.cs"

# Add-In patterns  
find . -name "AppManifest.xml" -o -name "*.spdata"
grep -rn "TokenHelper\|SharePointContext\|AppEventReceiver" --include="*.cs"

# Legacy JS patterns
grep -rn "_spPageContextInfo\|SP\.SOD\|ExecuteOrDelayUntilScriptLoaded" --include="*.js" --include="*.ts"

# Web part files
find . -name "*.webpart" -o -name "*.dwp"

# Authentication patterns
grep -rn "SPHostUrl\|SPAppWebUrl\|SharePointContextToken" --include="*.cs" --include="*.config"

### Quality Requirements

1. **Completeness:** Document every deprecated component found
2. **Accuracy:** Include exact file paths and line numbers
3. **Actionability:** Every finding must have a clear migration path
4. **Reusability:** Structure output to serve as template for similar projects
5. **Readability:** Use consistent formatting, clear headings, working Mermaid diagrams

### Process

1. Perform initial reconnaissance of repository structure
2. Execute pattern searches for deprecated components
3. Analyze results and categorize findings
4. Generate architecture diagrams
5. Create migration task breakdown
6. Estimate effort and identify risks
7. Produce all deliverable documents
8. Validate Mermaid diagrams render correctly
```
