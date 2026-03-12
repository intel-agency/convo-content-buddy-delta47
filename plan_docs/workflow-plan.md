# Workflow Execution Plan: Project Setup

## 1. Overview

| Field | Value |
|-------|-------|
| **Workflow Name** | `project-setup` |
| **Workflow File** | `ai_instruction_modules/ai-workflow-assignments/dynamic-workflows/project-setup.md` |
| **Project Name** | ConvoContentBuddy |
| **Repository** | `intel-agency/convo-content-buddy-delta47` |
| **Total Main Assignments** | 6 |
| **Event Assignments** | 3 (create-workflow-plan, validate-assignment-completion, report-progress) |

### High-Level Summary

This workflow initializes a new project repository for development. It transforms planning documents into actionable development artifacts: a formal application plan, complete project scaffolding, AI-focused documentation, and a comprehensive debrief. The workflow ensures quality through validation checkpoints after each step.

---

## 2. Project Context Summary

### Application Overview

**ConvoContentBuddy** is an autonomous, real-time semantic assistant designed to listen to technical conversations (e.g., coding interviews) and proactively display relevant algorithmic problem solutions. It operates as a background listener using a hybrid intelligence system to understand context, identify specific algorithmic problems, and retrieve optimal solutions via Google Search Grounding.

### Technology Stack

| Category | Technology |
|----------|------------|
| **Runtime** | .NET 10 (C# 14) |
| **Orchestration** | .NET Aspire 10 |
| **Backend** | ASP.NET Core 10 Web API |
| **Frontend** | Blazor WebAssembly |
| **Real-time** | SignalR with Redis Backplane |
| **Vector Database** | Qdrant (gRPC client) |
| **Relational/Graph DB** | PostgreSQL with pgvector |
| **AI/LLM** | Microsoft.SemanticKernel + Gemini 2.5 Flash + text-embedding-004 |
| **Resilience** | Polly (Retry, Circuit Breaker, Fallback) |
| **Observability** | OpenTelemetry (OTLP) |
| **Styling** | Tailwind CSS |

### Key Architecture Decisions

1. **Triple Modular Redundancy (TMR):** API.Brain service runs with `withReplicas(3)` for high availability
2. **N+2 Failover Strategy:** 3-tier fallback (Gemini + Search → Alternative Model → Safe Mode)
3. **Hybrid Retrieval Pipeline:** Vector search → Graph traversal → LLM verification
4. **Zero-Interaction UI:** Ambient dashboard that reacts to speech without user clicks

### Project Structure

```
ConvoContentBuddy.sln
├── ConvoContentBuddy.AppHost          # Aspire Orchestrator
├── ConvoContentBuddy.ServiceDefaults  # OTLP, Health Checks, Resilience
├── ConvoContentBuddy.API.Brain        # ASP.NET Core Web API, Semantic Kernel Hub
├── ConvoContentBuddy.UI.Web           # Blazor WASM, SignalR Client, Speech Interop
├── ConvoContentBuddy.DataSeeder       # Worker Service for LeetCode ingestion
└── ConvoContentBuddy.Core             # Shared DTOs, Event Models, Interfaces
```

### Special Constraints

- **Performance:** Semantic vector matching must complete in under 500ms; End-to-end processing under 2 seconds
- **Resilience:** Application must survive loss of any single container without dropping user session
- **Accuracy:** Hybrid Retriever must identify correct coding problem 95%+ of the time

### Repository Details

- **Template Repository:** `intel-agency/convo-content-buddy-delta47` (GitHub template)
- **Devcontainer:** Pre-built GHCR image with .NET 10 SDK, Bun, uv, opencode CLI
- **CI/CD:** GitHub Actions workflows for validation, Docker publishing, devcontainer prebuilds

---

## 3. Assignment Execution Plan

### Pre-Script-Begin Event

| Field | Content |
|-------|---------|
| **Assignment** | `create-workflow-plan`: Create Workflow Plan |
| **Goal** | Create comprehensive workflow execution plan before any assignments begin |
| **Key Acceptance Criteria** | • Dynamic workflow file read and understood<br>• All assignments traced and read<br>• All plan_docs/ files read<br>• Plan presented to stakeholder and approved<br>• Plan committed as `plan_docs/workflow-plan.md` |
| **Project-Specific Notes** | Extensive planning docs exist with detailed architecture. Must synthesize into clear execution roadmap. |
| **Prerequisites** | None (first task) |
| **Dependencies** | None |
| **Risks / Challenges** | None significant - straightforward planning task |
| **Events** | None |

---

### Main Assignment 1

| Field | Content |
|-------|---------|
| **Assignment** | `init-existing-repository`: Initiate Existing Repository |
| **Goal** | Initialize repository settings, create GitHub Project, import labels, rename workspace files |
| **Key Acceptance Criteria** | • PR and new branch created (`dynamic-workflow-project-setup`)<br>• GitHub Project created for issue tracking<br>• Project linked to repository<br>• Project columns created (Not Started, In Progress, In Review, Done)<br>• Labels imported<br>• Workspace/devcontainer files renamed to match project name |
| **Project-Specific Notes** | - Labels already exist in `.github/.labels.json`<br>- Workspace file already named `convo-content-buddy-delta47.code-workspace`<br>- Devcontainer name in `.devcontainer/devcontainer.json` may need verification<br>- Branch should be `dynamic-workflow-project-setup` |
| **Prerequisites** | GitHub authentication with scopes: `repo`, `project`, `read:project`, `read:user`, `user:email` |
| **Dependencies** | None (first main assignment) |
| **Risks / Challenges** | - GitHub Project (V2) API may have rate limits<br>- Permission issues if auth scopes insufficient<br>- Run `./scripts/test-github-permissions.ps1` to verify |
| **Events** | **post-assignment-complete:** `validate-assignment-completion`, `report-progress` |

---

### Main Assignment 2

| Field | Content |
|-------|---------|
| **Assignment** | `create-app-plan`: Create Application Plan |
| **Goal** | Create comprehensive application plan documented as a GitHub Issue, based on plan_docs/ |
| **Key Acceptance Criteria** | • Application template analyzed and understood<br>• Project structure documented<br>• All phases and steps detailed<br>• Technology stack and design principles followed<br>• Mandatory requirements addressed (testing, docs, containerization)<br>• Plan documented in GitHub Issue using template<br>• Milestones created and issue linked<br>• Issue added to GitHub Project<br>• Labels applied (planning, documentation) |
| **Project-Specific Notes** | - Extensive plan_docs/ already exist with 5 detailed documents<br>- Use existing docs as source material<br>- Create milestones based on 5 phases from roadmap<br>- No code implementation - planning only<br>- Document tech-stack.md and architecture.md in plan_docs/ |
| **Prerequisites** | `init-existing-repository` complete (GitHub Project exists) |
| **Dependencies** | GitHub Project from Assignment 1 |
| **Risks / Challenges** | - Complex architecture may require clarification with stakeholder<br>- .NET 10 Aspire is new - ensure plan reflects current best practices<br>- Must balance detail with clarity |
| **Events** | **pre-assignment-begin:** `gather-context`<br>**on-assignment-failure:** `recover-from-error`<br>**post-assignment-complete:** `validate-assignment-completion`, `report-progress` |

---

### Main Assignment 3

| Field | Content |
|-------|---------|
| **Assignment** | `create-project-structure`: Create Project Structure |
| **Goal** | Create actual project scaffolding: solution, projects, directories, configuration files |
| **Key Acceptance Criteria** | • Solution structure created following guidelines<br>• All required project files and directories established<br>• Initial configuration files created (global.json, Docker, etc.)<br>• Basic CI/CD pipeline structure established<br>• Documentation structure created<br>• Development environment validated<br>• Initial commit made<br>• Repository summary document created |
| **Project-Specific Notes** | - Create .NET 10 solution with 6 projects (AppHost, ServiceDefaults, API.Brain, UI.Web, DataSeeder, Core)<br>- Configure global.json with .NET 10.0 and rollForward<br>- Set up Aspire orchestration in AppHost<br>- Configure TMR with `withReplicas(3)` for API.Brain<br>- Add Qdrant, PostgreSQL, Redis containers to AppHost<br>- Create Dockerfile for each service<br>- Create docker-compose.yml<br>- Treat warnings as errors, create XML documentation |
| **Prerequisites** | `create-app-plan` complete (approved plan exists) |
| **Dependencies** | Approved application plan from Assignment 2 |
| **Risks / Challenges** | - .NET 10 SDK may have breaking changes from .NET 9<br>- Aspire hosting APIs evolving rapidly<br>- Must verify solution builds successfully before proceeding<br>- Docker configurations must be valid |
| **Events** | **post-assignment-complete:** `validate-assignment-completion`, `report-progress` |

---

### Main Assignment 4

| Field | Content |
|-------|---------|
| **Assignment** | `create-repository-summary`: Create Repository Summary |
| **Goal** | Create `.ai-repository-summary.md` file with AI agent-focused context and instructions |
| **Key Acceptance Criteria** | • File exists at repository root<br>• Contains project overview, tech stack<br>• Contains verified build/test commands<br>• Contains project structure / directory layout<br>• Commands validated by running them<br>• Linked from README.md |
| **Project-Specific Notes** | - Document Aspire startup sequence (`dotnet run --project AppHost`)<br>- Document .NET 10 build commands<br>- Document test execution<br>- Document Docker/Aspire container management<br>- Include verification steps for TMR and failover |
| **Prerequisites** | `create-project-structure` complete (actual structure exists to document) |
| **Dependencies** | Project structure from Assignment 3 |
| **Risks / Challenges** | - All documented commands must be verified to work<br>- Must keep under 32K tokens (preferably 8-16K)<br>- Commands must be in correct order |
| **Events** | **post-assignment-complete:** `validate-assignment-completion`, `report-progress` |

---

### Main Assignment 5

| Field | Content |
|-------|---------|
| **Assignment** | `create-agents-md-file`: Create AGENTS.md File |
| **Goal** | Create comprehensive AGENTS.md file following agents.md specification |
| **Key Acceptance Criteria** | • File exists at repository root<br>• Contains project overview section<br>• Contains verified setup/build/test commands<br>• Contains code style and conventions<br>• Contains project structure section<br>• Contains testing instructions<br>• Contains PR/commit guidelines<br>• Commands validated by running<br>• Stakeholder approval obtained |
| **Project-Specific Notes** | - Follow open AGENTS.md specification (agents.md)<br>• Complement README.md, don't duplicate<br>• Cross-reference with .ai-repository-summary.md<br>• Include .NET 10 specific conventions<br>• Document Aspire-specific commands<br>• Include Tailwind CSS conventions |
| **Prerequisites** | `create-repository-summary` complete |
| **Dependencies** | Repository summary from Assignment 4 |
| **Risks / Challenges** | - Must not conflict with existing AGENTS.md (if present)<br>- All commands must be validated<br>- Must be agent-focused, not human-focused like README |
| **Events** | **post-assignment-complete:** `validate-assignment-completion`, `report-progress` |

---

### Main Assignment 6

| Field | Content |
|-------|---------|
| **Assignment** | `debrief-and-document`: Debrief and Document Learnings |
| **Goal** | Create comprehensive debriefing report capturing learnings, insights, and improvement areas |
| **Key Acceptance Criteria** | • Detailed report created using structured template<br>• Report in .md format<br>• All 12 required sections complete<br>• Reviewed and approved by stakeholders<br>• Committed and pushed to repo<br>• Execution trace documented in `debrief-and-document/trace.md` |
| **Project-Specific Notes** | - Document any .NET 10 specific issues encountered<br>- Capture Aspire orchestration learnings<br>- Note any Gemini API integration challenges<br>- Record TMR verification results<br>- Document failover testing outcomes |
| **Prerequisites** | All previous assignments complete |
| **Dependencies** | All previous assignments |
| **Risks / Challenges** | - Must capture all learnings accurately<br>- Execution trace must be complete<br>- May surface issues requiring follow-up |
| **Events** | **post-assignment-complete:** `validate-assignment-completion`, `report-progress` |

---

### Event Assignment: validate-assignment-completion

| Field | Content |
|-------|---------|
| **Assignment** | `validate-assignment-completion`: Validate Assignment Completion |
| **Goal** | Validate completed assignment meets all acceptance criteria; outputs are functional |
| **Key Acceptance Criteria** | • All required files from assignment exist<br>• All verification commands pass (build, test, lint)<br>• Validation report created<br>• Pass/fail status determined<br>• Remediation steps provided if failed |
| **Project-Specific Notes** | - Run .NET build, test, format commands<br>- Verify Docker configurations<br>- Check GitHub state (issues, PRs, projects)<br>- Delegate to independent qa-test-engineer agent |
| **Prerequisites** | An assignment has just completed |
| **Dependencies** | Just-completed assignment outputs |
| **Risks / Challenges** | - Must be executed by independent QA agent<br>- GitHub API state verification required<br>- Must block progression if failed |
| **When Executed** | After each main assignment completes |

---

### Event Assignment: report-progress

| Field | Content |
|-------|---------|
| **Assignment** | `report-progress`: Report Progress After Workflow Step Completion |
| **Goal** | Provide progress reporting, output capture, and validation checkpoints after each step |
| **Key Acceptance Criteria** | • Structured progress report generated<br>• Step outputs captured and recorded<br>• Acceptance criteria validated<br>• Workflow state checkpointed<br>• User notification provided (optional) |
| **Project-Specific Notes** | - Track overall workflow progress (X/6 assignments)<br>- Capture key outputs (issue numbers, file paths)<br>- Enable resume-from-checkpoint capability |
| **Prerequisites** | Workflow step completed successfully |
| **Dependencies** | Completed step outputs |
| **Risks / Challenges** | - Should execute quickly to minimize overhead<br>- All outputs must be properly tagged |
| **When Executed** | After each main assignment completes |

---

## 4. Sequencing Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        PROJECT-SETUP WORKFLOW                                │
└─────────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────┐
│ PRE-SCRIPT-BEGIN EVENT   │
│                          │
│  create-workflow-plan    │ ◄── Current Task
│  (this plan document)    │
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐     ┌─────────────────────────┐
│ ASSIGNMENT 1             │     │ POST-COMPLETE EVENTS    │
│                          │     │                         │
│  init-existing-repository│────►│  validate-assignment-   │
│  • Create GitHub Project │     │    completion           │
│  • Import labels         │     │  report-progress        │
│  • Create branch         │     └─────────────────────────┘
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐     ┌─────────────────────────┐
│ ASSIGNMENT 2             │     │ POST-COMPLETE EVENTS    │
│                          │     │                         │
│  create-app-plan         │────►│  validate-assignment-   │
│  • Analyze plan_docs/    │     │    completion           │
│  • Create plan issue     │     │  report-progress        │
│  • Create milestones     │     └─────────────────────────┘
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐     ┌─────────────────────────┐
│ ASSIGNMENT 3             │     │ POST-COMPLETE EVENTS    │
│                          │     │                         │
│  create-project-structure│────►│  validate-assignment-   │
│  • Create .NET solution  │     │    completion           │
│  • Scaffold 6 projects   │     │  report-progress        │
│  • Configure Aspire      │     └─────────────────────────┘
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐     ┌─────────────────────────┐
│ ASSIGNMENT 4             │     │ POST-COMPLETE EVENTS    │
│                          │     │                         │
│  create-repository-summary│───►│  validate-assignment-   │
│  • Create .ai-repo-      │     │    completion           │
│    summary.md            │     │  report-progress        │
└────────────┬─────────────┘     └─────────────────────────┘
             │
             ▼
┌──────────────────────────┐     ┌─────────────────────────┐
│ ASSIGNMENT 5             │     │ POST-COMPLETE EVENTS    │
│                          │     │                         │
│  create-agents-md-file   │────►│  validate-assignment-   │
│  • Create AGENTS.md      │     │    completion           │
│  • Validate commands     │     │  report-progress        │
└────────────┬─────────────┘     └─────────────────────────┘
             │
             ▼
┌──────────────────────────┐     ┌─────────────────────────┐
│ ASSIGNMENT 6             │     │ POST-COMPLETE EVENTS    │
│                          │     │                         │
│  debrief-and-document    │────►│  validate-assignment-   │
│  • Create debrief report │     │    completion           │
│  • Document learnings    │     │  report-progress        │
│  • Create execution trace│     └─────────────────────────┘
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│ WORKFLOW COMPLETE        │
│                          │
│  • All assignments done  │
│  • PR ready for merge    │
└──────────────────────────┘
```

---

## 5. Open Questions

The following questions may need stakeholder input before or during workflow execution:

### Pre-Execution Questions

1. **Gemini API Keys:** Are Gemini API keys (`GEMINI_API_KEY`) already configured in repository secrets? The application requires Gemini 2.5 Flash and text-embedding-004 access.

2. **GitHub Project Permissions:** Does the authenticated user have permissions to create GitHub Projects (V2) linked to this repository?

3. **Alternative LLM Provider:** For Tier 2 failover, should Azure OpenAI be configured, or a secondary Gemini key? This affects the failover implementation in Phase 4.

### During-Execution Questions

4. **LeetCode Data Source:** For the DataSeeder utility, what is the preferred source for LeetCode problem data? (Official API, scraped JSON, community datasets?)

5. **Milestone Dates:** Should milestones have target dates assigned, or remain date-free for flexibility?

6. **Branch Protection:** Should the `main` branch have protection rules requiring PR reviews before the workflow PR can be merged?

### Post-Execution Questions

7. **Deployment Target:** Is Azure (via `azd` and Aspire) the intended deployment target, or should alternative deployment manifests be prepared?

8. **Monitoring Integration:** Should Application Insights or another APM solution be configured as part of the OpenTelemetry setup?

---

## 6. Critical Path Summary

The critical path through this workflow is:

```
create-workflow-plan → init-existing-repository → create-app-plan → 
create-project-structure → create-repository-summary → create-agents-md-file → 
debrief-and-document
```

**Estimated Total Assignments:** 6 main + 1 pre-event + 12 post-events = 19 assignment executions

**Key Dependencies:**
- `create-app-plan` depends on GitHub Project from `init-existing-repository`
- `create-project-structure` depends on approved plan from `create-app-plan`
- Documentation assignments depend on actual project structure existing

**Critical Success Factors:**
1. GitHub authentication with correct scopes
2. .NET 10 SDK availability in devcontainer
3. Stakeholder availability for plan approval
4. All build/test commands passing after structure creation

---

*Plan created: 2026-03-12*
*Workflow: project-setup*
*Status: Pending Stakeholder Approval*
