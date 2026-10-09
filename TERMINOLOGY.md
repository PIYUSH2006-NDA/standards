# Terminology

> **Status: Draft, not for implementation. No certification program exists.**

Shared terms for all SI Edge drafts. Where a draft defines a term normatively, the section is given. Document IDs: **SIE-RT** (runtime), **SIE-COM** (communication), **SIE-PRV** (provider).

| Term | Meaning | Defined in |
|---|---|---|
| **SI** | Shorthand used in these drafts for highly capable AI agent systems. Makes no claim that any system is superintelligent | README |
| **Personal SI** | An SI runtime that acts for one person on hardware they control and serves as their primary interface | README, SIE-RT intro |
| **Edge Device** | Hardware the owner controls that runs an SI Runtime or a shim | SIE-RT §1 |
| **SI Runtime** | On-device software that hosts agents, mediates every action, and presents the interface | SIE-RT §1 |
| **Runtime Core (Core)** | The trusted part of the runtime that enforces policy, permissions, egress, logging, and Trusted UI | SIE-RT §1 |
| **Owner** | The person or organisation that controls a runtime, its policy, and its keys | SIE-RT §1, §19 |
| **User** | A person who interacts with the runtime; may differ from the owner | SIE-RT §1, §19 |
| **Router Model** | Small local decision model that picks models and tools | SIE-RT §1 |
| **Control Plane** | Remote service that plans, orchestrates, or runs inference for the runtime | SIE-RT §1, §12 |
| **Module** | Installable unit with a signed manifest | SIE-RT §3 |
| **Capability** | An action exposed by the OS, an app, a module, or hardware | SIE-RT §1, §6 |
| **Capability descriptor** | Description of a hardware item: type, functions, performance class, safety constraints, attestation | SIE-RT §4 |
| **Trusted UI** | Fixed interface components rendered only by the Core through a channel others cannot imitate | SIE-RT §5 |
| **Adaptation Profile** | User-controlled description of how the runtime communicates and works with them | SIE-RT §7 |
| **Privacy Gateway** | Core component that all outbound cloud calls pass through | SIE-RT §13 |
| **Full gateway mode** | Strictest Gateway setting used for unverified (P0) providers | SIE-RT §13 |
| **Privacy report** | Per-call, user-visible summary of what was sent and what the provider could see | SIE-RT §13 |
| **Egress** | Any data that leaves the device | SIE-RT §1 |
| **Consequential Action** | Irreversible, costs money, communicates externally, changes access rights, or moves something physically | SIE-RT §1 |
| **SI Envelope** | Common signed message structure with versioned schema and canonical encoding | SIE-COM §A5 |
| **Form (F1 to F9)** | Classification of a message by communication pattern | SIE-COM §A1 |
| **Legacy endpoint** | Device, app, or OS that does not itself sign and verify SI Envelopes | SIE-COM Part B |
| **Shim** | Lightweight software that lets an endpoint speak the envelope without a full runtime | SIE-COM §B4 |
| **Proxy** | SI runtime that translates legacy protocols for legacy endpoints (Method 4) | SIE-COM §B5 |
| **Provider** | Any remote service a runtime calls | SIE-PRV §1 |
| **Provider Manifest** | Signed, machine-readable description of a provider's capabilities, policies, and claimed profile | SIE-PRV §8 |
| **Anonymous mode / Identified mode** | Requests without account credentials / with the user's own account | SIE-PRV §1 |
| **Conformance profile** | Set of rules a self-assessment can claim (R1 to R3, C1 to C3, P0 to P3). Not a certification | All drafts |
| **Profile tag** | Marker such as [R2] at the start of a rule giving the lowest profile it applies to | All drafts, Conventions |
| **Self-assessment** | The only form of conformance claim before v1.0 | All drafts |

## Residency Tags

Defined normatively in SIE-RT §11. Residency tags are an **owner policy choice**. They are stricter than the default rules of the Swiss nFADP and the EU GDPR, which allow transfers abroad under conditions such as adequacy decisions or appropriate safeguards.

| Tag | Allowed processing and storage locations |
|---|---|
| `CH` | Switzerland only |
| `EU` | EU and EEA member states only |
| `CH-EU` | Switzerland, or EU and EEA member states |
| other | Owner-defined. Any tag a runtime or provider does not recognise is treated as not allowed |

## Profile Tiers

| Tier | Runtime | Communication | Provider |
|---|---|---|---|
| 0 Unverified | n/a | C0 (legacy hop) | P0 |
| 1 Basic | R1 | C1 | P1 |
| 2 Sovereign | R2 | C2 | P2 |
| 3 Attested | R3 | C3 | P3 |

A session's effective tier is the lowest tier of the runtime, every communication hop, and every provider in its path.

## Labels Shown in Trusted UI

`legacy` (legacy endpoint), `unattested` (no verified attestation), `sideloaded` (module signed outside the owner's registries).
