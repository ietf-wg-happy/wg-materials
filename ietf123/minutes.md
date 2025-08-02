# HAPPY Working Group - IETF 123

### Time and Date

* Thursday, July 24, 09:30 - 11:30 CEST ([See this time in your local timezone](https://www.timeanddate.com/worldclock/fixedtime.html?msg=HAPPY+at+IETF+123&iso=20250724T0930&p1=141&ah=2))
* Location: [Hidalgo](https://datatracker.ietf.org/meeting/123/floor-plan?room=hidalgo)
* [IETF Agenda Link](https://datatracker.ietf.org/meeting/123/agenda/?show=happy)

### Links

* [Onsite Tool](https://meetings.conf.meetecho.com/onsite123/?group=happy&short=happy&item=1)
* [Meetecho Room](https://meetings.conf.meetecho.com/ietf123/?group=happy&short=happy&item=1)
* [Meeting chat](https://zulip.ietf.org/#narrow/stream/happy)
* [Notes](https://notes.ietf.org/notes-ietf-123-happy)
* [Minutes](https://datatracker.ietf.org/doc/minutes-123-happy/)

## Administrivia

* Blue sheets / scribe selection / [NOTE WELL](https://www.ietf.org/about/note-well.html) / [Code of Conduct](https://www.rfc-editor.org/rfc/rfc7154.html)
* Agenda revision

# Topics

- **Introduction**, _Chairs_, 5 minutes

## **[draft-ietf-happy-happyeyeballs-v3-01](https://datatracker.ietf.org/doc/draft-ietf-happy-happyeyeballs-v3//)**, _Tommy Pauly, Nidhi Jaju_, 70 minutes

Jen Linkova - be careful defining what v4 only or v6 only means.  what about w/ vpn? Do you really need default?
Tommy - distinction between default route

Ondřej - what is your preffered address family? it should be totally safe to do AAAA query on v4 only network.  if you don't have v6 default gateway then v6 should not be your preffered family? Trying to avoid penaly for not finishing the query. Can you define what v4 and v6 only mean? Can you send AAAA on v4 only but not wait for it unless you have a default route?

Tommy - the other bit that this interacts with is waiting for dns responses - if I don't have a default v6 route then I don't want to block or delay it was more of a hail mary - more about when we move on from the resolution step

Alessandro G - Relationship with SVCB RFC? How much should be mentioned for like how you follow svcb aliases. you may use happy eyeballs v2 to select the address?

Tommy P - text refers to SVCB RFC. We may not reference the specific section you're referring to, but we can make more specific. Please file issue if you see discrepancy. Hoping nothing needs to be updated in SVCB RFC

Alessandro G - SVCB doesn't work on all platforms or you might not have your dns client library working with werid querys - 
Tommy - if your thing supports it then make sure you get it if you don't support it immediately its a negative response

Erik Nygren - what do you do when you move on? Do you ignore further responses?

Tommy - you always respect answers that keep coming in up until the point you have have a connection. your dns recursive resolver is sending you back the answers and we don't want to do silly things - give a bit of a grace period for the recursive.

Geoff Huston - 50ms assumes that either there is no dns response or there is no data - it seems unnecessary - to what extent does no answer really feature? as di

David Schinazi - the idea here is you send both your queries - you get the response for the A and you're still waiting for the AAAA, what do you do? you would just wait indefinitely.  So after 50ms you just say I'm going to start the TCP sin on v4 and you still leave it open but lets not wait a full second when we have an address

Geoff - 50ms jsut seems long, I would have been more aggressive.

Tommy - maybe 50 is too long, we can tune that with more data collection

Jen Linkova - do we have any data for normal cases not including packet loss or jitter which condition does it matter ?

Tommy - you trigger the case on the right because something has happend and you don't want to wait a full second

Dragana Damjanovic - is this from when you send the request or after you get the first response?

Tommy - this is after you get the first response

Jordi - the 50ms is to make sure we give some preference to v6 - its somewhat dependent on the implementation 

Tommy - it is a configurable value that has a recommendation in the spec

Eric Kinnear - P50 is somewhere around 20-40ms P50 of how long it takes to receive all of the records

Tim Chown - section 4.2 - svcb address hints count

Benjamin Schwartz - we have consensus on what we want the behavior to be, but I don't think the text actually says that it doesn't read like you keep waiting in case less preferred options come in

Tommy - some other parts of the text definitively say you keep this open 

Erik Nygren - What if we only have aliases for SVCB/HTTPS records?

Tommy - If you get a proper service record, but it doesn't have address hints and you don't have any address corresponding to it, then it doesn't satisfy this condition (SVCB/HTTPS service info) - we should clarify this in the text - its an edge case that we should call out - if you only have an alias, you keep waiting and maybe push out the timer - you have to wait for a rtt to the resolver again


Lorenzo Colitti - we should have this hierarchy of what's important to sort by, but one might rely on data that you don't have like the A or AAAA record its difficult to separate the sorting from the delays 

Tommy - oh they are definitely related there's a trade-off b etween security vs latency - this is what you do when you have the set of answers

Alessandro G - is there an edge case where you can connect to the wrong server? do you merge svcb answer hints with normal aaaa or a answers?

Tommy - no, with multi-cdn scenarios it would be incorrect to have your svcb response... this is probably a misconfiguration (check transcript)

Yaroslav Rosomakho - some protocols support migration or multipath is it something we should take into account

Tommy - we think this is interesting, but its out of scope - its different from initial establishment

Ben Schwartz - to alessandro's point - that certaintly isn't why I thought we addded ipv4 and ipv6 in, I'm glad I didn't know about when we designed SVCB

I disagree with the drafts track on this business of ECH priority - there is no sense in the client trying to override the priority of the server - you can't improve security by encrypting the client hello when you weren't supposed to

Tommy - I agree we should not be in this position - but this about what do you do when you hit different scenarios - we may want a consensus call with the group because there are differing opion on what a client is allowed to do for its own security purposes


Erik Nygren - kthe address records are couple with that particular service binding and you  never have that mix and match problem 

on second point, I agree with Ben I think it simplifies things to obey the priorities that come in the https record

Mike Bishop - [not as AD] that is mostly what the hints are for and really shouldn't be deployed that way the drat does ahve to content with the fact thats a should and not an enforced model - your algrothim should wait a little longer if the higher priority bucket is empty. how much does the client care about ech versus speed

Ben Schwartz - I think its worth looking at the text again because it also says that the clients preferrence for different alpn values overrides the server preference which operationally is a mistake - it makes the priority only usable between identically configured endpoints which is limiting

the simpliest case - two servers fleets primary and secondary, I run the primary myself and outsource the secondary and the secondary activites quic, does everyone bypass primary and go to secondary

David Schinazi - if your backup doesn't have feature set of primary its not a backup. maybe we can massage the text to handle those cases separately

Daniel Gillmore - I'm not sure I understand Ben's point either.  The client should be able to choose its preferred security and app params. no matter what the text says, the client will have their own priorities.  its unrealistic to expect the client to not have its priorities.  

Lorenzo Colitti - the client will do what it wants no matter what we write here

Jen Linkova - does happy eyeballs know if the address is synthesized or not? how can you tell? what happens when we start seeing cases with multiple interfaces? how do we deal with multiple interfaces (vpn uses cases, etc). is this inside your PVD? why do we want another layer of translation even if its stateless? 

David Schinazi - its a requirement to know what the prefix is? is that fair? 

Ondřej - as a host I don't care if the network is ipv6 or dual stack. the worst case is that the native will go through nat44 and the synthesized v6 will go through nat64

Tommy - 

Mike Bishop - if you're on a network that has dns64 then the dns is going to generate aaaa records for you and if you're being sent through a v4 nat its a detail of the ntwork that the client doesn't have any visiblity to. 

its more simple than this conversation is making it. but there exist v6 addresses that are less preferred

Jordi -

Lorenzo C - if you're dual stack, you do what you're being told.  if you're v6 mostly then you do the needful 

we need to reduce the cases to two: v6 only and dual-stack

Erik Nygren - (see transcript - not sure if there was an actionable comment)

Lorenzo C - clarification: the attack is an on-patch attacker that is able to figure out which of your packets are asking for the https record - the network is able to pick out that one packet and drop only that one without dropping the others? 

Stephen F -

Ben Schwartz - the attack is possible with a probability of 1 out of 3

lorenzo c - if we're not using DoQ is there a way to defeat the attack or mostly mitigate it?

Ben Schwartz - rfc9460 this behavior is specified as mandatory

Daniel G - mitigation mechanisms - you need either coalescing all of the queries and all of the responses - I don't think you can guarantee that DoH is going to do that

Lorenzo - 

Tommy - this is a case where we should do some targeted studying to see practically what's happening (retransmissions and next connection attempt timing)


David S - there's no guarantee and that's fine its just a recommendation

Tommy - I feel strongly about keeping current approach that you keep falling back but don't wait for some dramatically longer time or wait until failures because we're talking about tcp timeouts and that's like 90s - falling aback through like 8 addresses and none of them are responding then its probablly time to try the backup

Ben Schwartz - we should have a target for the upper bound on the false positive backup fallback rate - any system with a false postivie fallback rate less than 5% is good enough - clear and strong bias for higher priority options


Tommy - it would be bad if one historical thing made you sort things different - what if you do have historical data that says all 8 address fail and you always fallback to the fallback and I'm just going to fallback to the backup pre-emptively

Ben Schwartz - your goals is to minimize the false positive fallback rate and theres a tuning question of finding heuristics that balance those two forces - whats important is you don't get stuck because of a small number of bad measurements - we want to be able to go back to option 1 if option 1 is working

Target stickiness question: thumbs up around the room

Lorenzo C - do we want to try harder to make the https/svcb query survive?

Eric K - Lorenzo please add a new issue for this question ^


## **Happy Eyeballs Testing**, _Dave Plonka_, 15 minutes

David Schinazi - The tooling is critical to the work we do, thank you very much for doing this work

## **Reporting Network Errors**, _Jordi Palet_, 25 minutes

David Schinazi - HE is implemented in the network stack; between your http api and your kernel sockets api. you could use it for non-http apps, conceptually it lives in the http stack

Jordi - should we recommend something for the implementers

David S - a recommendation isn't going to help because of Conways Law - we all ship our org charts

Tommy - for the record its not specific to http, there are other things like dns itself that use HE, when you're talking about reporting you have different solutions for different app types

Erik Nygren - two dimensions - older clients and things that are running on the server - shouldn't rely on browser client to collect that data

David Schinazi - on the topic of privacy - if you want a reporting solution, then you need to figure out ones that the implementers are comfortable with - need to give examples of breakage - what are the failure modes and what you want to fix


## **Closing and Next Steps**, _Chairs_, 5 minutes

Eric K - would anyone be interested in an interim on the main document - 8ish thumbs up in the room none on chat

An interim on reporting - 6ish in the room thumbs up

