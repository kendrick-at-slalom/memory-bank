# Architecture Review Board: 2026-04-15 (excerpt)

Attendees: Priya (chair, platform arch), Marcus (commerce arch), Devon (fulfillment lead), Sarah (notes)

This is a snippet from the middle of the meeting. The full transcript runs ~90 minutes; this excerpt is the part where the order-replay-strategy discussion intersected with a system constraint discovery.

---

**Priya:** Okay, item three. Marcus, you brought the order-replay strategy. Where are we?

**Marcus:** Half-formed. The premise is we've got the event log from ADR 0007, and we now need a snapshot strategy for fast catch-up. The naive thing is hourly snapshots, but we've got services that can't tolerate the load. I want a decision out of this conversation but I might be premature.

**Devon:** Fulfillment definitely can't take hourly. We're already hot on the read path during peak.

**Priya:** Is that a load problem or a network problem?

**Devon:** Network. The fulfillment cluster doesn't have the bandwidth back to the snapshot store during peak. We'd be saturating the link.

**Marcus:** Wait, why is bandwidth the bottleneck and not CPU?

**Devon:** Plant floor network. We inherited a constraint from the OT side: the fulfillment cluster sits behind a network appliance that gives us TCP only, and there's a hard cap on throughput we can't engineer around. UDP isn't on the table at all. I keep meaning to write this down somewhere and never have.

**Priya:** That's a real constraint. Sarah, can we get that captured? It's going to affect at least three more architecture conversations this quarter.

**Sarah:** Got it.

**Marcus:** Okay so if hourly is off the table, options are: less-frequent snapshots, snapshot-on-demand, or a different replay strategy entirely. I don't think we're going to land this today.

**Priya:** Agreed, table it for the next ARB. Devon, the network constraint goes in the bank either way; that's not contingent on the snapshot decision.

**Devon:** Yeah, fair.

**Priya:** While we're here: the broader pattern of "your team has a network constraint nobody else knows about" came up twice last month. Should there be a rule that systems with non-standard network constraints have to surface them to ARB before architecture decisions land?

**Marcus:** I'd be for it but I don't want to invent a rule on the fly. Floor it for next time.

**Priya:** Fair. Sarah, take it as an open item. Moving on.

---

[End of excerpt. Meeting continues with item four.]
