# Data Roads

An RFC and a short companion paper proposing a public data storage and
distribution scheme: data is encrypted and redundantly distributed across
many peers globally, with no single custodian. The generator of a piece of
data is its owner by default. Scientists and enterprises get access only
through an active, renewable agreement — never a permanent grant — in
exchange for some form of consideration ("payment") that is deliberately
left undefined in this early draft rather than resolved prematurely.

Data Roads doesn't invent a replication or claims layer from scratch — it's
designed to sit on top of two existing projects:

- [`open-agent-graph`](https://github.com/webdawg/open-agent-graph) — the
  signed, append-only event substrate (no global consensus, no blockchain)
  that Data Roads uses to anchor ownership claims and evidence.
- [`oag-mesh`](https://github.com/webdawg/oag-mesh) — the cluster-bridging
  and federated corroboration layer that Data Roads relies on for the actual
  redundant, global distribution.

## Status

Early draft. Expect frequent, substantial corrections — this is not a
stable spec yet.

## Contents

- [`rfc/0001-data-roads.md`](rfc/0001-data-roads.md) — the technical spec.
- [`paper/data-roads.md`](paper/data-roads.md) — a short companion paper
  summarizing the RFC's motivation and model for a reader who isn't
  implementing it directly.

## License

AGPL-3.0-or-later. See [`LICENSE`](LICENSE).
