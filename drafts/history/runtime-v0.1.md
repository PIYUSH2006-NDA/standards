# Open Standards for SI Edge Devices (Draft v0.1)

> **Status: Draft, not for implementation. No certification program exists.**
> **Superseded** by [SI Edge Runtimes v0.3](../runtime-v0.3.md). Kept for history only. Conformance levels and claim strings in this draft are withdrawn. See [TRADEMARKS.md](../../TRADEMARKS.md).

**Status:** Superseded draft · roenu (@roenudev), Bern · 2026-10-09

**Interpretation.** "SI" means superintelligence and advanced AI agents. "Edge devices" means hardware the user owns or controls, such as PCs, VPS instances, phones, home servers, robots, and cars, that run or host AI agents locally. The core idea: **the user's box holds the files, browser, credentials, and memory; inference and orchestration can be remote, but only under rules the device enforces.**

The key words MUST, SHOULD, and MAY are used as defined in RFC 2119.

---

## 1. Scope and Terms

- **Edge Device (ED):** user-controlled hardware that runs an agent runtime or executes tasks for one.
- **Edge Worker:** the process on the ED that runs tools and holds local state.
- **Control Plane (CP):** a remote service that plans, orchestrates, or does inference for the agent.
- **Decision Model ("System One"):** a small local model that classifies tasks and routes them to local or remote models.
- **Egress:** any data that leaves the ED.
- **Consequential Action:** an action that is irreversible, costs money, communicates externally, or changes access rights.

This standard covers the ED↔CP interface and the ED's local guarantees. It does not cover model training or CP internals.

## 2. Device Identity and Attestation

- An ED **MUST** have a stable, cryptographic device identity tied to a key that never leaves the device. *Rationale: a CP has to know which box it is talking to, and the user has to be able to revoke that box.*
- An ED **SHOULD** anchor that key in a hardware root of trust (TPM 2.0, Secure Enclave, or a TEE). *Rationale: a key held only in software can be copied from a compromised OS.*
- The agent runtime **MUST** be signed, and the ED **MUST** verify the signature before starting it. *Rationale: an unsigned runtime cannot be told apart from malware.*
- An ED **MAY** provide remote attestation (measured boot plus runtime measurement) to a CP or to the user. A CP **MUST NOT** require attestation from Level 1 devices. *Rationale: attestation is valuable, but requiring it would lock out DIY hardware.*
- The user, not the CP, **MUST** be able to rotate or revoke a device identity. *Rationale: the owner keeps final control.*

## 3. Data Sovereignty and Locality

- Data **MUST** stay on the ED by default. Egress **MUST** be allowed only by an explicit, user-editable **egress policy**: per destination, per data class, and per purpose. *Rationale: locality is the default and sharing is the exception.*
- The ED **MUST** keep an append-only, local **egress log** recording what left, to where, when, why, and under which policy rule. *Rationale: a sovereignty claim nobody can check is just marketing.*
- An egress policy **SHOULD** support jurisdiction constraints such as "CH only" or "EU/EEA only". Such constraints are an owner policy choice, stricter than the default transfer rules of the Swiss nFADP and the EU GDPR. *Rationale: Swiss and EU users need data residency they can enforce in practice.*
- A CP **MUST NOT** keep ED-originated content beyond the task unless the egress policy allows it explicitly. *Rationale: retention is a separate form of egress and needs its own consent.*

## 4. Remote Attach Protocol

- The ED **MUST** initiate the connection (outbound only). It **MUST NOT** need an open inbound port. *Rationale: this works behind NAT and on home networks, and shrinks the attack surface.*
- Transport **MUST** be mutually authenticated and encrypted: mTLS (TLS 1.3), SSH, or WebSocket over mTLS. *Rationale: both ends have to prove who they are.*
- Tool exposure **SHOULD** be built on the Model Context Protocol (MCP), with the Edge Worker acting as an MCP server toward the CP. *Rationale: reuse an open tool protocol rather than inventing a new one.*
- On connect, the two sides **MUST** negotiate capabilities: tools, filesystem roots, browser, GPU/model availability, the egress policy hash, and the conformance level. *Rationale: the CP should know up front what it may ask for.*
- **Offline behaviour:** when the CP is unreachable, the ED **MUST** keep working on local-capable tasks, **MUST** queue or refuse remote-only tasks, and **MUST NOT** widen its permissions. *Rationale: losing the network should degrade capability, never safety.*
- The user **MUST** be able to detach the CP instantly from the device. *Rationale: the right to leave is part of ownership.*

## 5. Model Routing Transparency

- An ED **SHOULD** run a local decision model that routes tasks by sensitivity, cost, and capability. *Rationale: sensitive tasks stay local, and only hard ones go to bigger models.*
- Every run **MUST** produce a routing record covering model ID and version, where it ran (local/CP/third party plus region), the routing reason, and the data classes sent. *Rationale: the user needs to know which model saw which data.*
- Routing records **MUST** be visible to the user and **SHOULD** be exportable as OpenTelemetry traces. *Rationale: use standard observability tooling, not a proprietary dashboard.*
- Routing rules **MUST** be overridable by the user, for example "never send class X off-device". *Rationale: user policy outranks optimisation.*

## 6. Credentials and Secrets

- Credentials (passwords, API keys, cookies, wallet keys) **MUST** stay on the ED, in an OS keystore or hardware-backed store where one exists. *Rationale: keeping secrets local is the main reason to own the box.*
- Secrets **MUST NOT** be included in prompts or context sent to remote inference. The ED **MUST** redact them before egress. *Rationale: anything a model has seen can leak through logs, training, or output.*
- Tools **SHOULD** use a secret by reference, so the Edge Worker uses it locally and returns only the result. *Rationale: the CP gets the effect of the secret without ever having the secret.*
- Delegated access **SHOULD** use scoped, expiring tokens (OAuth 2.1, an IETF Internet-Draft) instead of long-lived credentials. *Rationale: least privilege and automatic expiry.*

## 7. Permissions and Human Approval

- Permissions **MUST** be explicit and scoped: tool, path, domain, amount, and time window. *Rationale: broad, ambient permission invites misuse.*
- Consequential actions **MUST** require human approval unless the user has pre-authorised a narrowly scoped policy. *Rationale: humans stay in the loop where it matters.*
- Approval requests **MUST** show the exact action, its target, and its origin (which model or CP asked for it). *Rationale: people cannot meaningfully approve what they cannot see.*
- The ED **MUST** provide a local **kill switch** that halts all agent activity and revokes CP sessions, and it **MUST** work without the CP. *Rationale: the off switch belongs to the owner, and it must work offline.*
- For robots, vehicles, and other actuators, a physical or hardware-level stop **SHOULD** be present, alongside any applicable sector safety rules. *Rationale: software controls alone are not enough when actions have physical consequences.*

## 8. Safety and Update Policy

- Runtime, model, and skill updates **MUST** be signed and verified before installation. *Rationale: the supply chain is an attack surface.*
- The ED **MUST** support rollback to the last known-good version. *Rationale: a bad update must never brick the agent or lock out the user.*
- Runtime and skill packages **SHOULD** ship an SBOM (SPDX or CycloneDX). *Rationale: you cannot audit what you cannot list.*
- Tools and skills **MUST** run sandboxed (container, VM, or OS sandbox), with filesystem and network access limited to what was negotiated. *Rationale: limits the blast radius of compromised or confused agents.*
- Providers **SHOULD** document how their deployments map to EU AI Act obligations where those apply. *Rationale: makes compliance easier without this standard claiming to provide it.*

## 9. Interoperability

- The ED **MUST** be model-agnostic, able to run or call any compliant local or remote model. *Rationale: avoids vendor lock-in at the most important layer.*
- Agent memory, skills, policies, and logs **MUST** be exportable and importable in open, documented formats such as JSON/JSONL, Markdown, and SQLite. *Rationale: users own their agent's accumulated state.*
- Skills **SHOULD** be portable packages (manifest, code, permissions, SBOM) that can be installed on any conforming ED. *Rationale: a common skill format lets an ecosystem grow.*

## 10. Energy and Resource Reporting

- The ED **SHOULD** report per-task resource use: CPU/GPU time, memory, and an estimate of energy (Wh) where it can be measured. *Rationale: running locally has real costs, and the user should see them.*
- The user **MAY** set resource budgets (power, thermal, battery, cost) that the router **MUST** respect. *Rationale: phones, cars, and home servers each have different constraints.*

## 11. Conformance Levels

| Level | Name | Requires |
|---|---|---|
| **L1** | Basic | §1, outbound mTLS/SSH attach (§4), local secrets (§6), human approval and kill switch (§7), signed updates and rollback (§8), export (§9) |
| **L2** | Sovereign | L1 + default-local data, egress policy and egress log (§3), jurisdiction constraints, routing records (§5), sandboxing, SBOM |
| **L3** | Attested | L2 + hardware root of trust, verified signed runtime, remote attestation (§2), hardware-backed secrets, resource reporting (§10) |

Claims **MUST** state the level and version, for example "SI-Edge L2 / v0.1", and **SHOULD** come with a public self-assessment.

## 12. Open Questions

1. How should attestation work on hardware without a TPM/TEE (older PCs, some VPS hosts) without excluding them?
2. Can a VPS count as "user-owned" when the hypervisor operator can read memory? Should confidential VMs be required for L3?
3. What is a minimal, shared taxonomy of data classes for egress policies?
4. How should a remote model's context be verifiably deleted, and can it be?
5. Should routing records be signed so they can serve as evidence in audits?
6. How do multiple EDs owned by one user (phone, PC, server) share memory and policy?
7. Which body should maintain this standard: an open working group, a foundation, or an existing standards organisation?

---
*Feedback welcome. This draft is intentionally opinionated and incomplete.*
