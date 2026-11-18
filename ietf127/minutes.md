# HAPPY Working Group Agenda - IETF 127

## Time and Date

* Monday, November 16, 16:30 - 18:30 PST ([See this time in your local timezone](https://www.timeanddate.com/worldclock/fixedtime.html?msg=HAPPY+at+IETF+127&iso=20261116T1630&p1=224&ah=2))
* Location: [Golden Gate 7-8](https://datatracker.ietf.org/meeting/127/floor-plan?room=golden-gate-7-8)
* [IETF Agenda Link](https://datatracker.ietf.org/meeting/127/agenda/?show=happy)

## Links

* [Onsite Tool](https://meetings.conf.meetecho.com/onsite127/?group=happy&short=happy&item=1)
* [Meetecho Room](https://meetings.conf.meetecho.com/ietf127/?group=happy&short=happy&item=1)
* [Meeting chat](https://zulip.ietf.org/#narrow/stream/happy)
* [Notes](https://notes.ietf.org/notes-ietf-127-happy)
* [Minutes](https://datatracker.ietf.org/doc/minutes-127-happy/)

## Administrivia

* Blue sheets / scribe selection / [NOTE WELL](https://www.ietf.org/about/note-well.html) / [Code of Conduct](https://www.rfc-editor.org/rfc/rfc7154.html)
* Agenda revision
* Scribes: David Plonka, Keely Podosin

## Topics

### **Introduction and Agenda Bashing**, _Chairs_, 5 minutes

### **[draft-ietf-happy-happyeyeballs-v3](https://datatracker.ietf.org/doc/draft-ietf-happy-happyeyeballs-v3/)**, _Tommy Pauly, Nidhi Jaju_, 30 minutes

Tommy and Nidhi reviewed updates to the draft (see slides).

From Tim via chat, what changed from -04 to -05 is here: https://author-tools.ietf.org/iddiff?url2=draft-ietf-happy-happyeyeballs-v3-05

The PvD concept is from RFC 7556 (not the different IPv6-specific PvD from RFC 8801).

There was a lot of discussion in the queue that how to use a pool of connections is out of scope, so the draft should keep the details about canceling/abandoning connections that "lose" to a minimum.

The issues list is clean; Tommy asked if we can go to WGLC.

Ben Schwartz asked that reviewers read it as if they were a new reader, to see if the structure makes sense or should be improved with a major editorial pass.

Eric suggested readers identify what is confusing and drive changes from that. He suggested an extended last call (before the next meeting) collecting last call issues.

Eric said he took the discussion to mean the group will start last call(s).

### **Firefox Partial Rollout Report**, _Max Inden (remote)_, 20 minutes

Max said Firefox has started a rollout and A/B experimentation.
(See details in slides, e.g., reported that about 23% of page loads involved new connections, thus HEv3 is relevant. HEv3 benefits all users in time to "first contentful paint.")

Questions from Philipp Tiesel (see recording, hard to hear in room), Esteban Carisimo.

Lorenzo Colitti: Interesting that DNS answers come at the same time; there is a lot of uniformity.

Max: HTTPS responses are usually faster than A and AAAA.

Lorenzo: What is the 600 second timer?

Max: Previously we would only do optimistic DNS for positive responses; now we do it for negative as well. That is treated as stale after 600 sec. I am optimistic to increase that.

Ben: on optimistic DNS (based on dnsop WG): it causes more problems than you might be able to see on the client side. For example, (1) trying to turn off a server cluster... the traffic doesn't leave; (2) with automatic DNS load-balancing, slow-moving clients can result in overloaded servers in the cluster. Don't jump in too hard on optimistic DNS.

Ben: optimistic HTTP/3 makes it hard to test it with a small load.

Stuart Cheshire: q (to Ben): why not just take the server down after the TTL? The clients must be using optimistic DNS and HE, so they will fail over to another.

Ben: it would be good if the DNS query indicated it was optimistic DNS.

Tim Chown: Are there significant deviations in your implementation from the draft?

Max: all of mine are resolved in the draft.

### **Update on Chromium Implementation**, _Kenichi Ishibashi_, 20 minutes

See slides of HEv3 implementation in stages.
Noted that it's important to evaluate connection latency overall (with HEv3), not per-protocol alone.

Alessandro Ghedini: is there a timeline for the field testing?

Kenichi: no.

Mirja Kuehlewind: Do you know anything about these slow mobile networks?

Kenichi: not yet.

Mirja: We should be sure not to conceal the problem, but try to fix those slow networks.

### **Happy Eyeballs v3 Implementation in Swift**, _Gautam Akiwate_, 10 minutes

Open source: https://github.com/apple/swift-network-evolution

Q from Gautam (to group): preferring endpoints with established use; perhaps [we should] consider including endpoints with existing connections.
Currently this is a place where they slightly diverged from the existing draft.

Ben Schwartz: help me understand the difference between prioritizing endpoints with an existing connection vs. including it — is it having a "fresh" RTT estimate?

Gautam: either use HEv3 only after considering the existing pool, or something else... (see recording for details)

Ben: suggest being conceptually clear by looking at the existing pool, and only using HEv3 if a new connection is required. HE can be informed by the fact that there is an existing connection in the pool. "The clearest thing is for HE to be a new connection creation machine."

Ben: this might defeat some DNS-based load balancing.

Stuart Cheshire: The fastest connection setup does not predict the fastest connection for what you're trying to do. The original motivation [for HE] was when IPv6 was broken; it was about getting a connection that works *at all*. (This idea is not going to do that; we can't predict that the connection will perform well in the future; only multipath protocols can do that.)

### **[Considerations for Happy Eyeballs Error Reporting](https://datatracker.ietf.org/doc/html/draft-palet-happy-reporting-considerations)**, _Philipp Tiesel_, 15 minutes

See slides.

Lars Eggert: [see recording/transcript]

Mirja Kuehlewind: You want a better experience for users, and you want to find problems and fix them. [implication that otherwise HEv3 might hide them]

Altanai Bisht: [see recording/transcript]

Erik Nygren: IETF is interested in making v4 perform well in deployment, and it's hard to track down the corner cases... [implication that it can't be easily measured/debugged outside the application]

Tommy: we've talked several times about this; perhaps a new thought: can we get more progress, for the network case, from just passive observation? HE changes the behavior; this is visible to the network and the operator.

Ben Schwartz: advocating for application "qlogs" that could be shared.

Tim Chown: suggested narrowing down the use cases.

Mirja: happy to keep working on the qlog stuff.

### **Wrap-up and Next Steps**, _Chairs_, 10 minutes

The chairs indicated they would move to last call(s), as discussed above.
