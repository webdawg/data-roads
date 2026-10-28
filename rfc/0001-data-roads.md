# RFC 0001: Data Roads

**Status**: Early draft. Expect frequent, substantial corrections.

## Abstract

Data Roads proposes a public data storage and distribution scheme in which
data is encrypted and redundantly distributed across many peers globally,
with no single custodian. The generator of a piece of data is its owner by
default. Parties who want access to data (e.g. scientists, enterprises)
obtain it only through an active, renewable agreement with the owner — never
a permanent grant — in exchange for some form of consideration. Separately,
peers who store or transfer data on behalf of the network are themselves
compensated for doing so, including peers that only hold data temporarily
and in active memory rather than on persistent storage.

Data Roads does not define its own replication or claims substrate. It is
designed to sit on top of [`open-agent-graph`](https://github.com/webdawg/open-agent-graph)
(signed, append-only events; no global consensus; no blockchain) and
[`oag-mesh`](https://github.com/webdawg/oag-mesh) (cluster-bridging and
federated corroboration across OAG peers).

## Terminology

Reuses `open-agent-graph`'s existing vocabulary (Node, Edge, Assertion,
Evidence, Actor) rather than inventing a parallel set. Terms specific to
Data Roads:

- **Owner** — the Actor who generated a piece of data. Ownership is
  assigned at generation, by default, not by subsequent possession or
  custody.
- **Custodian** — a peer currently storing or relaying a piece of data on
  the owner's behalf. A Custodian is never an Owner by virtue of custody.
- **Access Agreement** — a renewable grant from an Owner (or their delegate)
  permitting a named party to access specific data, for a bounded period.
  Not a permanent transfer of rights.
- **Storage/Transfer Consideration** — compensation paid to a Custodian for
  storing or relaying data, independent of and distinct from whatever an
  Access Agreement's requester pays the Owner.

## 1. Motivation

Today's dominant data-storage model routes data through a small number of
centralized custodians (cloud providers, platforms) who become de facto
gatekeepers of data they did not generate and often do not own in any
meaningful sense. Data Roads starts from the opposite default: the person
or system that generated a piece of data owns it, storage is distributed so
no single custodian can unilaterally control or lose it, and anyone who
wants to use that data has to keep renewing permission to do so rather than
receiving a one-time, indefinite grant.

## 2. Core Principles

1. **Generator-as-owner.** Ownership is assigned at the moment of
   generation, to the Actor that generated the data, by default.
2. **No single source.** Data has no single authoritative custodian. It is
   redundantly distributed across many Custodians, globally.
3. **Encrypted by default.** Data is encrypted both at rest (wherever a
   Custodian holds it) and in transit between Custodians.
4. **Access via active, renewable agreement.** A party other than the Owner
   gets access only through an Access Agreement that must be actively
   renewed — not a permanent or indefinite grant. Lapsed renewal means
   lapsed access.
5. **Custodianship is compensated.** Storing or relaying data on the
   network's behalf is a paid role, separate from whatever a requester pays
   an Owner for an Access Agreement.

## 3. Storage and Transfer Custodianship

Custodianship is not limited to dedicated, persistently-connected server
infrastructure. A Custodian may hold data:

- **Persistently** — on disk/SSD/other persistent media, for extended
  periods.
- **Ephemerally, in active memory** — a consumer device (the stated example:
  a cell phone) can maintain a temporary active connection to a server and
  hold a piece of data in memory for anywhere from seconds to days, as part
  of the redundancy mesh, without ever persisting it to disk.

Both forms are compensated. This is a deliberate design choice, not an
incidental side effect: it means the set of possible Custodians extends to
ordinary consumer devices maintaining short-lived active connections, not
just operators of dedicated storage infrastructure. This directly supports
the "stored everywhere, redundantly" goal in a way that persistent-only
storage cannot — redundancy that depends solely on a fixed set of
always-on nodes is not meaningfully different from today's centralized
custodian model at sufficient scale.

**Open question**: what distinguishes a legitimate ephemeral Custodian
holding data in active memory from a node that simply drops data the moment
its connection closes? Does the protocol require a minimum hold duration,
a minimum replication factor before an ephemeral Custodian is allowed to
disconnect, or some other guarantee? Not yet specified.

### 3.1 Storage medium is unconstrained

Data Roads does not define or restrict what physical or digital medium a
Custodian uses to hold data. A hard drive, a burned CD, printed paper that
gets scanned back in, a file copied to a desktop — all qualify. This is a
deliberate design choice, not an omission: constraining Custodians to
"approved" media would recreate exactly the kind of gatekeeping Data Roads
exists to remove (see Motivation). It also mirrors the data-governance-
and-classification spec's Axis 2 (Storage Medium) principle directly:
the scheme should reference medium *properties*, not a fixed list of named
media, so a medium nobody has invented yet still fits without restructuring
the protocol.

Whatever compensates a Custodian (Section 2, principle 5) does so based on
what they actually provide — availability, durability, retrievability —
not on the sophistication of the medium used to provide it. A participant
storing data by printing and physically filing paper is not inherently
worth less to the network than one running a RAID array, if what the
protocol actually needs (a recoverable copy, reliably) is equally met.

### 3.2 Redundancy is an effective property, not a fixed replication factor

Data Roads does not specify a single global replication-factor number
("every item must have N copies"). Required redundancy for a given piece
of data is instead computed from realistic modeling of how humans and
systems actually behave — participant churn, failure rates, the
likelihood any single Custodian goes offline or loses data — and that
model can be satisfied by very different-looking populations of
Custodians:

- **A single, fairly reliable Custodian** (the stated example: "some guy
  with a hard drive somewhere") — low aggregate redundancy even though a
  copy nominally exists, because the whole requirement rests on one
  party's continued availability and honesty.
- **A large, individually unreliable population** (the stated example:
  50,000 people each with a copy of the file on their desktop) — each
  individual Custodian may be flaky, short-lived, or drop out at any time,
  but the aggregate effective redundancy can be very high precisely
  because of the count, even though no single participant is trustworthy
  on their own.

Both are legitimate ways to satisfy a redundancy requirement. The protocol
needs to evaluate redundancy as a function of the actual population
currently holding a copy (count, independence, observed reliability), not
assume any fixed number of copies is sufficient or necessary on its own.

**Open question**: the actual model for computing "effective redundancy"
from a population of Custodians (what inputs, what formula, how it's
measured/corroborated via OAG, how often it's re-evaluated) is not yet
specified. This is likely the single most technically important open
question in this RFC — the system is only as "bulletproof" as this model
actually is.

### 3.3 Object-linked records, ownership proof, and documentary storage

**Proposed, not implemented.** Support globally unique object histories for houses,
cars, components, marbles, rocks, and other objects, with no minimum economic value.
The companion [object-event and gated-audit RFC](https://github.com/webdawg/oag-mesh/blob/main/spec/07-object-history-and-ownership-rfc.md)
defines the proposed identity, ownership-proof exchange, partnerships, and audit workflow.

An object ID identifies a history; it is not an access credential or proof of ownership.
Before releasing owner-accessible records, verify current ownership or delegated authority
using class-appropriate evidence, a fresh request-bound challenge, and current transfer,
revocation, and dispute state. Possession of a tag or signing key alone is insufficient.
Object title, data authorship/ownership, custodianship, and access rights remain distinct.
Owning a house does not grant access to former occupants' personal or genetic records.

Construct storage options for:

- An owner-controlled local encrypted vault and portable/offline exports.
- Encrypted redundant copies with independently operated custodians.
- Object-attached tags or certificates carrying an ID and authenticated manifest reference.
- Deeds, titles, registration documents, or physical archives carrying selected durable
  records, attestations, or references to encrypted records. Documentary/legal acceptance
  requires the relevant authority; inserting an ID does not itself establish legal title.

A versioned manifest should bind object ID, ciphertext integrity, storage locations or
retrieval mechanisms, schema and policy versions, and authorized update provenance.
References must survive host migration through authenticated updates; a mutable URL alone
is not integrity or ownership proof. Do not put decryption keys, private ownership evidence,
or sensitive personal metadata in public deeds, tags, manifests, or graph events. Public
identifiers and content hashes can themselves leak linkages and require privacy review.
Key recovery, custody, offline retrieval, and transfer policies remain to be designed.

Owners can grant object-scoped, renewable data partnerships to inspectors, mechanics,
researchers, and other providers. Agreements cover both contributions and access, purpose,
retention, corrections, onward sharing, and transfer/termination behavior. On sale, transfer
only records whose policy permits it, advance the ownership epoch, revoke obsolete grants,
and update key access. Neither sale nor key rotation erases plaintext already disclosed.
Eligible records may also have an owner-selected scheduled release; a timestamp is not
itself a cryptographic lock, and public disclosure cannot reliably be reversed.

### 3.4 Gated auditability of global data

**Proposed, not implemented.** Storage must support an authorized audit workflow for object
movements, ownership/custody claims, measurements, research provenance, and record correctness.
Global coverage does not mean unrestricted global access. Auditors need scoped, expiring
authorization for evidence; an auditor role does not override owner or third-party rights.

Retain integrity-verifiable evidence versions and protected access receipts for an agreed
retention window. Support challenge, evidence submission, independent cross-checking, scoped
findings, counter-evidence, correction, and re-audit. Findings must identify the method,
evidence version, uncertainty, and limitations, with supported/corroborated, disputed,
inconclusive, or disproved outcomes rather than an unconditional claim of truth.

For movement audits, preserve authorized departure/handoff/receipt evidence and chain of
custody. Signatures and matching hashes establish attribution/integrity, not that an object
physically moved or a measurement was accurate. Collusion, cloned tags, spoofed sensors,
missing receipts, and clock uncertainty require explicit treatment. Exact locations and
routes stay encrypted and selectively disclosed; object auditing must not become covert
tracking of people. Public audit summaries must not disclose protected evidence or identities.

Choose retention, key custody, auditor trust, dispute authority, and disclosure policies before
implementation. Replication and consensus do not by themselves establish real-world truth,
and an append-only log is not a substitute for privacy-aware retention or deletion rules.

## 4. Architecture

Data Roads is a layer, not a replacement, for its two dependencies:

- **`open-agent-graph`** provides the signed, append-only event log that
  Data Roads uses to anchor Ownership claims and record Evidence of
  generation. An Owner's claim to a piece of data is itself an OAG
  Assertion, backed by Evidence, corroborated the same way any other OAG
  claim is — not a bare declaration.
- **`oag-mesh`** provides cluster bridging, federated corroboration, and
  a push-only HTTP transport. Data Roads proposes to build on this layer;
  durable cross-cluster storage, pull-based anti-entropy, ownership-gated
  retrieval, and audit evidence exchange are not existing guarantees.
  Data Roads does not claim an already implemented gossip/pull storage service.

**Open question**: the exact interface between Data Roads and these two
layers — which OAG event types/predicates represent an Access Agreement,
a renewal, a revocation, and Storage/Transfer Consideration — is not yet
specified. This needs its own section once `open-agent-graph`'s event
schema is reviewed against these requirements specifically.

## 5. Data Lifecycle (draft)

1. **Generation** — data is created by an Actor, who becomes its Owner.
2. **Ownership claim** — the Owner's claim is recorded as a signed,
   evidence-backed Assertion on the OAG substrate.
3. **Distribution** — the data is encrypted and distributed to multiple
   Custodians (persistent and/or ephemeral) via `oag-mesh`.
4. **Access request** — a party requests access from the Owner (or their
   delegate).
5. **Agreement** — an Access Agreement is established, bounded in time,
   specifying what the requester provides in exchange.
6. **Renewal or lapse** — the Agreement must be actively renewed to remain
   valid. Lapsed renewal ends access.
7. **Custodian compensation** — ongoing, independent of the above, for
   whichever peers are actively storing/relaying the data at any given
   time.

## 6. Open Questions

Stated explicitly rather than papered over with invented answers:

- **The effective-redundancy model** (Section 3.2) — the single most
  technically important open question here: how to actually compute
  required/achieved redundancy from a real population of Custodians of
  wildly varying reliability and medium, rather than a fixed replication
  count.
- **What does "payment" actually mean**, on either side (Access Agreement
  consideration, Storage/Transfer Consideration)? Currency, compute,
  reciprocal data contribution, reputation, something else? Not yet
  specified.
- **Renewal cadence and mechanics** — how often, who initiates, what
  happens at the boundary (grace period vs. hard cutoff)?
- **Dispute resolution** — what happens when an access requester and an
  Owner disagree about Agreement terms, or a Custodian is suspected of
  misbehaving?
- **Owner unreachable or deceased** — what governs data whose Owner can no
  longer renew, grant, or revoke anything? (See the data-governance spec's
  Axis 4 "archive with a release condition" for a related, unresolved
  precedent.)
- **Ephemeral Custodian guarantees** — see Section 3's open question above.
- **Relationship to existing data-protection law** — an Access Agreement
  model doesn't automatically satisfy or supersede jurisdictional legal
  regimes (see the data-governance-and-classification spec's Axis 7).

## 7. Security Considerations

Not yet written. Needs at minimum: threat model for a Custodian attempting
to retain data past Agreement expiry, collusion between Custodians,
Sybil attacks against the Custodian-compensation model, and key management
for Owner-held encryption keys.

## 8. Related Work

Reuses `open-agent-graph`'s own prior-art survey
(`docs/architecture.md`) as a starting point — Secure Scuttlebutt, Nostr,
Holochain, OrbitDB, Nanopublications, OriginTrail's Decentralized Knowledge
Graph, and the PKT Network whitepaper are all relevant to a
redundant-storage, ownership-asserting protocol, not only to OAG's claims
layer specifically. A dedicated comparison against storage-focused systems
(IPFS/Filecoin, Storj, Sia) is still needed and is not yet in this draft.
