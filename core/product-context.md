---
inclusion: always
---

# Product Context

## What It Is

A **no-code / low-code workflow automation platform** for business users — in the same category as Zapier and n8n, but differentiated by a stronger focus on usability for non-technical users.

> *Powerful automation made approachable for real business users.*

---

## Who It's For

**Primary:** Business analysts, ops teams, back-office staff, project coordinators, support leads — people who understand the process but don't want to depend on engineers to automate it.

**Secondary:** Technical consultants, IT admins, and developers who build or maintain more advanced workflows.

---

## Core Principles

- **End-user-first** — assume users don't know APIs, IDs, or auth scopes. Abstract that away.
- **Simple surface, powerful core** — hide complexity, don't remove capability.
- **Guided experience** — human-readable labels, smart defaults, progressive disclosure, clear errors.
- **Business process clarity** — a workflow should be explainable by the person who owns the process.
- **Safe by default** — validate inputs, warn before destructive actions, make errors actionable.

---

## Key Capabilities

| Area | What It Supports |
|---|---|
| Workflow Design | Visual drag-and-drop builder, triggers, actions, branching, data mapping |
| Execution | Reliable automation engine, retry/error handling, execution tracking |
| Integrations | CRM, ERP, email, databases, APIs, third-party services |
| Deployment | Multi-tenant SaaS + self-hosted; separate DB per customer |
| Admin | User access control, execution history, monitoring |

---

## Typical Use Cases

- Sync data between CRM and internal systems
- Automate approval and routing flows
- Notify teams when events occur
- Manage onboarding/offboarding tasks
- Handle back-office and repetitive admin processes

---

## Implications for Writing Requirements & Tickets

1. **Don't assume technical users** — no raw API details, IDs, or schema names exposed by default.
2. **Prefer pickers and dropdowns** over free-text technical inputs.
3. **Always cover both config-time and runtime behavior.**
4. **Output matters** — every action must return useful data for downstream steps.
5. **Errors must be actionable** — plain-language messages explaining what failed and why.
6. **Advanced options are optional** — keep the default path simple and safe.
