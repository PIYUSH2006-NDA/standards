# Open Standards for SI Edge Runtimes (Draft v0.2)

> **Status: Draft, not for implementation. No certification program exists.**
> **Superseded** by [SI Edge Runtimes v0.3](../runtime-v0.3.md). Kept for history only. Conformance levels and claim strings in this draft are withdrawn. See [TRADEMARKS.md](../../TRADEMARKS.md).

**Status:** Superseded draft · roenu (@roenudev), Bern · 2026-10-09 · Supersedes v0.1

**Interpretation.** This is an open standard for a native **SI runtime**: an advanced AI agent runtime that runs directly on user-owned edge devices (phones, PCs, VPS instances, home servers, cars, robots) and **is the primary interface**. Users talk to the runtime, and the runtime drives tools, apps, files, and hardware for them. The standard is model-agnostic. Any local or remote model can be plugged in as the decision or reasoning model. Nothing here describes or implies any vendor's actual plans or products.

The key words MUST, SHOULD, and MAY follow RFC 2119.

---

## 1. Scope and Terms

- **SI Runtime:** the on-device software that hosts agents, mediates every action, and presents the interface.
- **Runtime Core:** the trusted part of the runtime. It enforces policy, permissions, egress, and logging.
- **Router Model:** a small local decision model ("System One") that classifies tasks and picks a model and tools.
- **Remote Model / Control Plane (CP):** off-device inference or orchestration.
- **Capability:** an action exposed to the runtime by the OS, an app, or a device.
- **Egress:** any data that leaves the device. **Consequential Action:** an action that is irreversible, costs money, communicates externally, changes access rights, or moves something physically.

## 2. Runtime Architecture

- A conforming runtime **MUST** separate the **Runtime Core** from models. Models propose actions and the Core decides whether they run. *Rationale: a model must never be the security boundary.*
- The runtime **SHOULD** include a local **Router Model** that routes by sensitivity, cost, latency, and capability. *Rationale: sensitive and simple tasks stay local, and only hard ones go out.*
- Remote models **MUST** be pluggable through a documented, vendor-neutral provider interface. *Rationale: no model lock-in at the interface layer.*
- Tools and apps **SHOULD** be reached through the Model Context Protocol (MCP) or a compatible capability API (§4). *Rationale: reuse an open protocol.*
- The runtime **MUST** include a local **memory store** (§5) and a **scheduler** for persistent and background agents, with per-agent budgets. *Rationale: persistent agents need durable state and controlled execution.*
- The runtime **MUST** work in a degraded, local-only mode without network access. *Rationale: losing the network should make the interface less capable, never unusable or unsafe.*

## 3. Interface Layer

- Voice, text, and vision (camera, screen) **MUST** be supported as primary inputs, as far as the hardware allows. *Rationale: the runtime replaces the app grid as the way people interact with the device.*
- The runtime **MAY** generate UI on demand (forms, tables, previews, controls). Generated UI **MUST** be rendered by the Runtime Core from a declarative schema, not as arbitrary model-produced code. *Rationale: generated UI is an attack surface.*
- Legacy apps **MUST** remain available as a fallback, and the user **MUST** be able to bypass the agent. *Rationale: the agent must never be the only way into a person's own device.*
- The runtime **MUST** show at all times what the agent is doing: the active task, tools in use, and data leaving the device. *Rationale: an interface that acts on the user's behalf has to be observable.*
- The interface **MUST** meet established accessibility guidelines (for example WCAG for visual output) and **SHOULD** support switching modalities mid-task. *Rationale: replacing the interface must not exclude people.*

## 4. Device and App Capability API

- Apps and the OS **MUST** expose actions to the runtime through declared capabilities, not by screen scraping, wherever a declared capability exists. *Rationale: structured actions are more reliable and easier to audit.*
- Each app or skill **MUST** ship a **permission manifest** listing its capabilities, data classes touched, network destinations, and which actions are consequential. *Rationale: users and the Core need to know the blast radius before granting access.*
- UI automation (accessibility trees, screen control) **MAY** be used as a fallback, but it **MUST** run with the same permission checks and logging. *Rationale: fallback paths must not become loopholes.*
- Hardware capabilities (camera, mic, location, actuators) **MUST** be gated individually and show a visible indicator while active. *Rationale: sensors and actuators are the most sensitive capabilities.*

## 5. Persistent Agents and Memory

- Agent memory **MUST** be stored locally by default, encrypted at rest, in an open and documented format (for example SQLite plus JSONL/Markdown, with embeddings stored alongside their source text). *Rationale: memory is the user's data and should outlive any vendor.*
- Users **MUST** be able to inspect, edit, delete, and export memory, and import it into another conforming runtime. *Rationale: portability and the right to erasure (nFADP/GDPR).*
- Multiple agents on one device **MUST** have isolated memory and permissions unless the user explicitly grants sharing. *Rationale: a coding agent does not need a finance agent's memory.*
- Persistent agents **MUST** run under scheduler budgets (time, compute, money, actions) and **MUST** be listable and stoppable by the user. *Rationale: background autonomy needs visible limits.*

## 6. Multi-Device Continuity

- A user's devices (phone, PC, VPS, car, home server) **SHOULD** act as one **runtime mesh** with shared identity, policy, and memory. *Rationale: one assistant, many bodies.*
- Sync **MUST** be end-to-end encrypted with keys that only the user's devices hold, and **SHOULD** work peer-to-peer without a mandatory cloud relay. *Rationale: continuity must not require giving up sovereignty.*
- Each device **MUST** keep its own identity and be revocable on its own. *Rationale: losing a phone must not compromise the mesh.*
- Policy **MUST** be able to restrict data by device (for example "health data never syncs to the VPS"). *Rationale: devices differ in trust and jurisdiction.*

## 7. Device Identity and Attestation

- Each device **MUST** have a cryptographic identity whose key never leaves the device, and the user **MUST** be able to rotate or revoke it. *Rationale: the owner controls trust.*
- The key **SHOULD** be anchored in a hardware root of trust (TPM 2.0, Secure Enclave, or a TEE). *Rationale: keys held only in software can be copied.*
- The Runtime Core **MUST** be signed and verified at startup. *Rationale: an unsigned runtime cannot be trusted with every interaction on the device.*
- Devices **MAY** provide remote attestation, and it **MUST NOT** be required below Level 3. *Rationale: keep DIY hardware in scope.*

## 8. Data Sovereignty and Egress

- Data **MUST** stay on the device by default. Egress **MUST** be allowed only by an explicit, user-editable policy: per destination, data class, purpose, and jurisdiction (for example "CH only" or "EU/EEA only"). Such constraints are an owner policy choice, stricter than the default transfer rules of the Swiss nFADP and the EU GDPR. *Rationale: locality is the default.*
- The Core **MUST** keep an append-only local **egress log** of what left, to where, why, and under which rule. *Rationale: a sovereignty claim has to be checkable.*
- Remote parties **MUST NOT** retain device content beyond the task unless the policy explicitly allows it. *Rationale: retention is a separate form of egress.*

## 9. Remote Models and Control Plane

- Connections **MUST** be outbound from the device and mutually authenticated (mTLS with TLS 1.3, or SSH), with no open inbound ports required. *Rationale: works behind NAT and keeps the attack surface small.*
- A CP **SHOULD** reach device tools only through MCP, and the runtime **MUST** apply its local permissions to every CP request. *Rationale: a remote planner is just another untrusted caller.*
- Capabilities, the egress-policy hash, and the conformance level **MUST** be negotiated on connect. *Rationale: the remote side knows its limits up front.*
- The user **MUST** be able to detach any CP or remote model instantly, and the runtime **MUST** then fall back to local mode. *Rationale: being able to leave is part of ownership.*

## 10. Model Routing Transparency

- Every run **MUST** record which model and version ran, where (device, CP, or third party, plus region), the routing reason, and the data classes sent. *Rationale: users have to know which model saw what.*
- Routing records **MUST** be visible to the user and **SHOULD** be exportable as OpenTelemetry traces. *Rationale: use standard tooling.*
- User routing rules (for example "never send class X off-device") **MUST** override router decisions. *Rationale: user policy outranks optimisation.*

## 11. Credentials and Secrets

- Credentials **MUST** stay on the device in an OS or hardware-backed keystore. *Rationale: secrets are the core of local control.*
- Secrets **MUST NOT** be included in any model context, local or remote. Tools **MUST** use secrets by reference, executed by the Core. *Rationale: a model that never sees a secret cannot leak it.*

## 12. Permissions, Approval, and Kill Switch

- Consequential actions **MUST** require human approval unless the user has pre-authorised a narrow policy (scope, amount, time). *Rationale: humans stay in the loop where it matters.*
- Approval prompts **MUST** show the exact action, its target, which agent and model proposed it, and the data involved. *Rationale: people cannot meaningfully approve what they cannot see.*
- A local **kill switch** **MUST** stop all agents, revoke remote sessions, and work offline. Actuated devices (cars, robots) **SHOULD** also have a hardware stop and follow applicable sector safety rules. *Rationale: the off switch belongs to the owner.*

## 13. Safety, Updates, and Supply Chain

- Runtime, model, and skill updates **MUST** be signed, verified, and possible to roll back. *Rationale: the supply chain is an attack surface, and a bad update must not lock the user out.*
- Skills and tools **MUST** run sandboxed, limited to their manifest. *Rationale: limits the blast radius.*
- Packages **SHOULD** ship an SBOM (SPDX or CycloneDX). *Rationale: you cannot audit what you cannot list.*
- Providers **SHOULD** document how their deployments map to EU AI Act obligations where those apply. *Rationale: easier compliance, without this standard claiming to provide it.*

## 14. Energy and Resources

- The runtime **SHOULD** report per-task compute and estimated energy, locally and (where known) remotely, via OpenTelemetry metrics. *Rationale: lets local and remote options be compared fairly.*
- User budgets (battery, thermal, cost) **MUST** be respected by the router and the scheduler. *Rationale: phones and cars have hard limits.*

## 15. Conformance Levels

| Level | Name | Requires |
|---|---|---|
| **R1** | Runtime | Core/model separation, local-only mode (§2), activity visibility and legacy fallback (§3), permission manifests (§4), local exportable memory (§5), local secrets (§11), approvals and kill switch (§12), signed updates (§13) |
| **R2** | Sovereign | R1 + egress policy and log (§8), routing records (§10), outbound mTLS/SSH remote attach (§9), sandboxing and SBOM, E2E-encrypted multi-device sync (§6) |
| **R3** | Attested | R2 + hardware root of trust, verified Core, remote attestation (§7), hardware-backed secrets, resource reporting (§14) |

Claims **MUST** state the level and version (for example "SI-Runtime R2 / v0.2").

## 16. Open Questions

1. How should OS vendors expose system-level capabilities to third-party runtimes, and who governs the capability schema?
2. What is a minimal shared memory format that preserves embeddings across different models?
3. Can a VPS count as user-owned without confidential computing?
4. Approval fatigue: how should approvals scale for agents that take many small actions?
5. Which body should maintain this standard?

---
*v0.2 changes: reframed around a native runtime that acts as the interface; added architecture, interface, capability API, memory, and multi-device sections; folded remote attach into §9; redefined conformance levels as R1 to R3.*
