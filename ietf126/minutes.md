# HAPPY Working Group Agenda - IETF 126

## Time and Date

* Monday, July 20, 14:00 - 16:00 CEST ([See this time in your local timezone](https://www.timeanddate.com/worldclock/fixedtime.html?msg=HAPPY+at+IETF+126&iso=20260720T14&p1=259&ah=2))
* Location: [Grand Klimt Hall 1](https://datatracker.ietf.org/meeting/126/floor-plan?room=grand-klimt-hall-1)
* [IETF Agenda Link](https://datatracker.ietf.org/meeting/126/agenda/?show=happy)

## Links

* [Onsite Tool](https://meetings.conf.meetecho.com/onsite126/?group=happy&short=happy&item=1)
* [Meetecho Room](https://meetings.conf.meetecho.com/ietf126/?group=happy&short=happy&item=1)
* [Meeting chat](https://zulip.ietf.org/#narrow/stream/happy)
* [Notes](https://notes.ietf.org/notes-ietf-126-happy)
* [Minutes](https://datatracker.ietf.org/doc/minutes-126-happy/)

## Administrivia

* Blue sheets / scribe selection / [NOTE WELL](https://www.ietf.org/about/note-well.html) / [Code of Conduct](https://www.rfc-editor.org/rfc/rfc7154.html)
* Agenda revision

## Topics

### **[draft-ietf-happy-happyeyeballs-v3](https://datatracker.ietf.org/doc/draft-ietf-happy-happyeyeballs-v3/)**, _Tommy Pauly, Nidhi Jaju_, 25 minutes

#### Document status / completed PRs
- Editorial updates merged: "address" → "endpoint" where appropriate; "interface" → "PVD" for precision.
- Clarified handling of larger IPv6 header sizes.
- Added concrete examples of DNS answer processing (how the algorithm reacts to combinations of HTTPS, A, and AAAA records).

Ben Schwartz: I thought the MTU issues were fragmentation-related, not header-size-related. Please have a fragmentation person review this.

#### [PR 137](https://github.com/ietf-wg-happy/draft-happy-eyeballs-v3/pull/137)

Establishes §4.1 rules for which queries to send based on PVD routes: query A if global IPv4 is available; query AAAA if global IPv6 is available; query A if a NAT64 prefix is detected. NAT64 handling simplified — clients discover the prefix via RA or ipv4only.arpa; if present, synthesized IPv6 addresses are treated as IPv4 for sorting and grouping. Eliminates the "last resort local synthesis delay" timer.

Jen: if you have synth-v6 and real v4, in case you have more specific routes for v4... this would solve the VPN case. we'll discuss it on the pr

Philipp: vpn case will more or less be the same issue as the pvd... what could go wrong? wrong resolver -- that's a different PVD. PVD context and routes are bound to the VPN. getting PVD right should cover all VPN cases.

Tommy: "pull it up into a broader section" as Erik Nygren said. A purist PVD standpoint would keep the answers completely separate. there are split-tunnel cases where the deployment expects to violate this... implementations do resolve on X and connect on Y, which would make this cause breakages. 

Lorenzo: we didn't add any mechanism to declare absence of info in a PVD, so you can't easily default-in parts of another PVD. that's the VPN case and the challenge here. wonder whether we should fix that in PVD. this doc should give pragmatic guidance. does the PVD doc think a VPN is a PVD? not clear. 

#### [PR 133](https://github.com/ietf-wg-happy/draft-happy-eyeballs-v3/pull/133)

(whether the draft should specify how to sort recursive resolver addresses)

Lorenzo: ...penalizing non-responsive is fine, but keep sorting from the DNS side as to which candidate / protocol to use otherwise. RFC 6724bis already specifies address-family sorting — could just reference it.

Ben Schwartz: just put a long comment on this PR. we should try to stay out of this. dns connection optimization is a different problem space. easier to understand if it's separate. six transports for dns.

Tommy: would like to get a sense of the room as to what direction to go.

(poll: keep and iterate or remove entirely — clear preference in the room to remove detailed DNS resolver selection from the draft)

Andrew Yourtchenko: there has to be some description of how it works when one 

Stuart: i tend to have a preference for *some* explanation as to how this should interface with DNS, for people who pick this doc up cold. ok to be non-normative, just to help in understanding the philosophy.

Lorenzo: do we think happy will want to decide based on whether an answer was encrypted? i concur with stuart. 

Tommy: mainly an ech discussion, no?

Lorenzo: to do ech you have to know the answer was encrypted or not.

#### Exponential Connection Attempt Delay

Max Inden: proposed replacing the constant 250ms connection attempt delay with a 50ms initial delay followed by an exponential (2x) backoff.

Tommy: Apple's implementation doesn't use fixed timers — it relies on historical RTT and retransmission timers.

Nidhi: Chromium is also moving toward using historical RTT.

(group agreed to look at reducing the recommended default timer values in the draft)

### **[Optimistic DNS](https://datatracker.ietf.org/doc/draft-gakiwate-dnsop-optimistic-dns/)**, _Gautam Akiwate_, 10 minutes

(timeboxed, to be discussed in dnsop)

Stub resolvers immediately return expired cached records to the application while firing an asynchronous background refresh query. Async DNS makes optimistic DNS possible; Happy Eyeballs makes it safe (the app can fall back to fresh addresses if the expired ones fail). Shipping on Apple devices via mDNSResponder since 2018; Firefox and Chromium are working on similar integrations.

Erik Nygren: distinguish a short grace period (transient races) from ignoring TTLs for long periods (e.g. 7 days), which has real security/operational implications.

Ben Schwartz: could let domains opt in / tune optimistic TTL extension via a parameter in HTTPS records.

(currently targeted at dnsop, but depends heavily on HE safety guarantees — could potentially progress in HAPPY)

### **A Report on Firefox’s HEv3 Implementation**, _Max Inden_, 15 minutes

- HEv3 enabled by default in Firefox Nightly (~15,000 users); core state machine modularized into a reusable, deterministic library ([mozilla/happy-eyeballs](https://github.com/mozilla/happy-eyeballs)).
- Parameters: 25ms resolution delay (draft suggests 50ms); 50ms initial connection attempt delay with a 2x multiplier (draft suggests 250ms constant).
- First connection attempt succeeds in 84% of cases; only one attempt is made in 86% of sessions.
- Interleaves both IP families and protocols: QUIC v6, QUIC v4, TCP v6, TCP v4.
- HTTPS record positive responses in ~7% of connections.

Lorenzo: if you race the first address with the first address, how would your metrics change? how much of this has to do with you're just hammering the network harder? are we measuring the efficacy of HE or its side-effects? this would give us an upper bound on the utility of this whole working group. if sending more packets makes it work... just send more packets! you interleave protocols. i'm guessing you looked at the draft and it said use 6724, and you said... that looks hard. would be good to get guidance.

Tommy: whenever we have an https record, we use it. Max: only 3% of website operators send an https record. Tommy: is the query going out properly, or are there bugs? my numbers, i see 17%, and it causes an h3 upgrade. 

### **Update on Google/Chromium Implementation, Including Optimistic DNS**, _Kenichi Ishibashi_, 15 minutes

- Refactoring connection management to share DNS resolutions between TCP and QUIC and avoid waiting for all DNS responses.
- TCP: incremental resolution fully implemented and near shipping; 15ms resolution delay when a AAAA response is outstanding.
- QUIC: incremental resolution in development — start handshakes as soon as a usable endpoint resolves, fall back to other-family addresses if the first is slow or fails.
- Optimistic DNS groundwork completed in the resolver; field trials for TCP planned.

Johannes Zirngibl: do you race quic against tcp or tls/tcp; is the signal the tls handshake? 

Kenichi: totally separate codepaths currently

Nidhi: Chromium waits for the TLS handshake to complete.

Ben Schwartz: What happens if QUIC is attempted, and tcp/tls wins but quic works, do you fallback?

Kenichi: no.

Ben Schwartz: we should make sure the hev3 text supports this connection escalation.

Ben Schwartz: what are the stale-cache lifetime limits (optimistic DNS)?

Kenichi: cache is cleared on browser relaunch and on network changes, though on desktop it could persist for weeks.

### **Update on Apple's Happy Eyeballs Implementation**, _Tommy Pauly_, 10 minutes

- Relies on historical RTT data rather than static timers.
- First attempt wins the race in 97% of connections.
- Recent betas ensure HE reordering happens strictly within SVCB priority groups (respecting operator priorities).
- A/AAAA resolution-time delta: P90 17ms, P95 34ms, P99 100ms.

Andrew Yourtchenko: i see a lot of focus on the happy path, can you characterize the failures? would retry solve to?

Tommy: the first thing does keep retransmitting, doesn't tell you whether flooding the network would make it better

Vaibhav: battery life?

Tommy: if you're aligned with the radio is already up, not terrible. probably.

### **Happy Eyeballs Webtester Update and Hackathon Report**, _Johannes Zirngibl and Tim Betzer_, 10 minutes
- Hackathon Activity: https://wiki.ietf.org/en/meeting/126/hackathon#happy-eyeballs-webtester
- Web-based testing framework updated to support HEv3 tests (on a staging site); tests client behavior by delaying server hellos (TLS/QUIC handshake layer) or network-layer packets by IP family and protocol.
- Hackathon additions: server-side packet capture and DNS query arrival logging to reconstruct precise connection traces.
- Added test cases for HTTP/3 detection, ECH fallback, HTTPS alias-mode behavior, and delayed-AAAA handling.

### **HEv3 Event Logging with qlog**, _Mirja Kuehlewind_, 15 minutes

- Standardized qlog-based logging format to help debug failures normally masked by HE fallbacks.
- Defines three data types (attempt, policy, and DNS resolution) and a sequence of key connection events (timers, candidate discovery, attempts, and terminal connection summaries).

Lars: which operator? I can log this, how can we get it where it needs to be?

Mirja: I'm thinking about network operators. For right now, offline. Online, we need a new architecture, but we need the data model first.

Ben: we'd want a lot more info for debugging use cases, maybe there are qlog events that are missing, e.g. for DNS?

Mirja: events are also there. this is just an additional aggregated summary.

Ben: often missing is the way to thread all the details together. HESession seems very useful. I have a HEv3 [PR 125](https://github.com/ietf-wg-happy/draft-happy-eyeballs-v3/pull/125) up to make the state machine much more explicit, which connects with this.

Lucas: this is a -00 but i like it. dns should be its own qlog schema? a standard for these events, threading and session ids super important.

### **[Considerations for Happy Eyeballs Error Reporting](https://datatracker.ietf.org/doc/html/draft-palet-happy-reporting-considerations)**, _Philipp Tiesel_, 15 minutes
- Including discussion of [HAPPY-SAD](https://datatracker.ietf.org/doc/draft-trammell-happy-sad/)

Lars: personas: dev/users, fully want to support. makes sense. managed devices is basically the user, almost equivalent. for almost everything else there i don't like it. NEL: that the server, so maybe, but even there, skeptical. no idea why a browser wants to expose anything to them ever.

Andrew: is there some way to additionally signal expectations / willingness to do something about it?

Mike Blanche: the two ends of the list have a relationship. the middle, no existing relationship. there could be ways to expose that info from the endpoints.

Valentin Goşu: most personas can already get this information... why should apps do yet more work? think about actionability.

Brian Trammell: happy-sad not a promise, something between a joke and a threat. reviving the draft. better to think of it as a thought experiment.

Jordi: nine years ago i started asking for error reporting. my customers are isps. their bad situations with IPv6 were hidden by HE. i'm missing a discussion about which personas we care about.


## As Time Permits

- **[Enhanced Dual Stack and the Need for an HEv3 OS Implementation Profile](https://datatracker.ietf.org/doc/draft-xiao-v6ops-eds/)**, _Xipeng Xiao_
  - Argues the traditional dual-stack model (RFC 6724) is too risky for enterprises: a broken IPv6 deployment degrades user experience before failing back.
  - Proposes "Enhanced Dual Stack": (1) a system/OS-level HEv3 implementation profile so all system applications benefit, not just browsers; (2) enhancing legacy APIs (getaddrinfo, connect) to factor in address performance; (3) clear error reporting so operators can diagnose and fix IPv6 issues.
  - Invited the working group to review and provide feedback.
