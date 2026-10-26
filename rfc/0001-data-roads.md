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

## 4. Architecture

Data Roads is a layer, not a replacement, for its two dependencies:

- **`open-agent-graph`** provides the signed, append-only event log that
  Data Roads uses to anchor Ownership claims and record Evidence of
  generation. An Owner's claim to a piece of data is itself an OAG
  Assertion, backed by Evidence, corroborated the same way any other OAG
  claim is — not a bare declaration.
- **`oag-mesh`** provides the actual cross-cluster replication and
  federated corroboration that makes "distributed across many peers
  globally" a real property rather than an aspiration. Data Roads does not
  reimplement replication; it relies on `oag-mesh`'s existing
  gossip/pull-replication model.

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
