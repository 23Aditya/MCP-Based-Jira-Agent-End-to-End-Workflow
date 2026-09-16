# MCP-Based-Jira-Agent-End-to-End-Workflow

### Working

The workflow starts when the user submits a request in natural language, such as “Create a Jira ticket for the login issue and assign it to Rahul.” The AI application understands the request, detects the intent, and extracts the required information such as the project, issue type, description, and assignee. The AI then uses the MCP client to discover and select the appropriate Jira MCP tool, such as `create_issue()` or `assign_issue()`. The MCP request passes through the MCP Gateway, where authentication, authorization, RBAC, validation, rate limiting, and audit logging can be applied. The request then reaches the Jira MCP Server, which exposes Jira operations as standardized MCP tools and internally communicates with Jira through the Jira REST API. Jira creates the ticket, assigns it to Rahul, and the ticket proceeds through its lifecycle—such as Open, Assigned, In Progress, Testing, Approval or Rejection, Resolved, and finally Closed. During the process, the MCP tools can perform actions such as updating the issue, adding comments, changing status, or checking the current ticket state. Once the workflow is complete, the system uses `get_issue()` to verify the final ticket status, assignee, and resolution. The result then travels back from Jira through the Jira MCP Server, MCP Gateway, and MCP Client to the AI application, which converts the structured result into a natural-language response and informs the user that the ticket has been completed.

    ┌──────────────────────────────────────────────────────────────┐
    │ 1. USER                                                      │
    │                                                              │
    │ "Create a Jira ticket for the login issue and assign it      │
    │  to Rahul."                                                  │
    └────────────────────────────┬─────────────────────────────────┘
                                 │
                                 ▼
    ┌──────────────────────────────────────────────────────────────┐
    │ 2. AI APPLICATION / AGENT                                    │
    │                                                              │
    │ • Understand natural-language request                        │
    │ • Detect intent                                              │
    │ • Extract project, issue type, summary, description, assignee│
    └────────────────────────────┬─────────────────────────────────┘
                                 │
                                 ▼
    ┌──────────────────────────────────────────────────────────────┐
    │ 3. MCP CLIENT                                                │
    │                                                              │
    │ • Discover Jira MCP tools                                    │
    │ • Select required tool                                       │
    │                                                              │
    │ Example:                                                     │
    │   create_issue()                                             │
    │   assign_issue()                                             │
    │   update_issue()                                             │
    │   change_status()                                            │
    │   add_comment()                                              │
    │   get_issue()                                                │
    └────────────────────────────┬─────────────────────────────────┘
                                 │
                           MCP Request
                                 │
                                 ▼
    ┌──────────────────────────────────────────────────────────────┐
    │ 4. MCP GATEWAY                                               │
    │                                                              │
    │ • Authentication                                             │
    │ • Authorization / RBAC                                       │
    │ • Request validation                                         │
    │ • Rate limiting                                              │
    │ • Audit logging                                              │
    └────────────────────────────┬─────────────────────────────────┘
                                 │
                                 ▼
    ┌──────────────────────────────────────────────────────────────┐
    │ 5. JIRA MCP SERVER                                           │
    │                                                              │
    │ Exposes Jira operations as MCP tools                         │
    │                                                              │
    │ create_issue()                                               │
    │ assign_issue()                                               │
    │ update_issue()                                               │
    │ change_status()                                              │
    │ add_comment()                                                │
    │ get_issue()                                                  │
    └────────────────────────────┬─────────────────────────────────┘
                                 │
                          Jira REST API
                                 │
                                 ▼
    ┌──────────────────────────────────────────────────────────────┐
    │ 6. JIRA                                                      │
    │                                                              │
    │ Create Ticket                                                │
    │       ↓                                                      │
    │ PROJ-123                                                     │
    │       ↓                                                      │
    │ Assign to Rahul                                              │
    └────────────────────────────┬─────────────────────────────────┘
                                 │
                                 ▼
                      ┌─────────────────────┐
                      │   TICKET LIFECYCLE  │
                      └─────────┬───────────┘
                                │
                                ▼
                        ┌───────────────┐
                        │     OPEN      │
                        └───────┬───────┘
                                │
                                ▼
                        ┌───────────────┐
                        │    ASSIGNED   │
                        │     Rahul     │
                        └───────┬───────┘
                                │
                                ▼
                        ┌───────────────┐
                        │  IN PROGRESS  │
                        │ Developer     │
                        │ works on fix  │
                        └───────┬───────┘
                                │
                                ▼
                        ┌───────────────┐
                        │    TESTING    │
                        │ QA validation │
                        └───────┬───────┘
                                │
                          ┌─────┴─────┐
                          │           │
                          ▼           ▼
                    ┌──────────┐  ┌──────────┐
                    │  FAILED  │  │  PASSED  │
                    └────┬─────┘  └────┬─────┘
                         │             │
                         │             ▼
                         │      ┌──────────────┐
                         │      │   APPROVAL   │
                         │      │ User/Manager │
                         │      └──────┬───────┘
                         │             │
                         │       ┌─────┴─────┐
                         │       │           │
                         │       ▼           ▼
                         │   ┌────────┐  ┌─────────┐
                         │   │ REJECT │  │ APPROVE │
                         │   └───┬────┘  └────┬────┘
                         │       │             │
                         │       │             ▼
                         │       │      ┌────────────┐
                         │       │      │  RESOLVED  │
                         │       │      └─────┬──────┘
                         │       │            │
                         └───────┘            ▼
                                      ┌────────────────┐
                                      │     CLOSED     │
                                      └───────┬────────┘
                                              │
                                              ▼
    ┌──────────────────────────────────────────────────────────────┐
    │ 7. FINAL JIRA STATUS CHECK                                   │
    │                                                              │
    │ MCP Client → MCP Server → get_issue() → Jira                 │
    │                                                              │
    │ Verify:                                                      │
    │ • Ticket ID                                                  │
    │ • Final status                                               │
    │ • Assignee                                                   │
    │ • Resolution                                                 │
    └────────────────────────────┬─────────────────────────────────┘
                                 │
                                 ▼
    ┌──────────────────────────────────────────────────────────────┐
    │ 8. RESULT RETURNS THROUGH MCP                                │
    │                                                              │
    │ Jira → MCP Server → MCP Gateway → MCP Client → AI            │
    └────────────────────────────┬─────────────────────────────────┘
                                 │
                                 ▼
    ┌──────────────────────────────────────────────────────────────┐
    │ 9. FINAL AI RESPONSE                                         │
    │                                                              │
    │ "Ticket PROJ-123 was created, assigned to Rahul,             │
    │  successfully resolved, approved, and closed."               │
    └──────────────────────────────────────────────────────────────┘
