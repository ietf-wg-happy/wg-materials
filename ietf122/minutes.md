# HAPPY Working Group Minutes
IETF 122, Bangkok

Tuesday, March 18, 17:00 - 18:30 ICT

Location: Boromphimarn 1/2

Session recording: https://www.youtube.com/watch?v=WpuWzn2nSc4

## Chair's introduction (10 minutes)
Eric Kinnear (Apple), Tim Chown (Jisc)

Chairs' email: happy-chairs@ietf.org 

Happy WG datatracker page: https://datatracker.ietf.org/wg/happy/about/

Chairs' slides: https://datatracker.ietf.org/meeting/122/materials/slides-122-happy-chair-01

Eric presented the agreed charter.

The main focus of the WG is updating the core happy eyeballs algorithm.

A secondary informational document on reporting connectivity problems may also be published.

Eric confirmed the github mode of working with editors having merge permissions for both editorial and design changes, under the Chair's oversight, at least until we get closer to last call. 

If anyone believes issues are closed prematurely they can email the chairs at happy-chairs@ietf.org.


## HEv3 Draft Discussion
draft-pauly-happy-happyeyeballs-v3-00

Tommy Pauly, Nidhi Jaju, 35 minutes

Slides: https://datatracker.ietf.org/meeting/122/materials/slides-122-happy-happy-eyeballs-v3-01

Datatracker for draft: https://datatracker.ietf.org/doc/draft-pauly-happy-happyeyeballs-v3/


* Why do we need HEv3? Lots of changes in DNS and transport area since the earlier two versions including QUIC, SVCB, and ECH. It's no longer just about IPv4/IPv6.
* Two main goals of doc: incorporate new technologies since RFC 8305 and review and update configurable values in the algorithm based on operational experience.
* Scope is client side algorihtm for connection establishment - going from FQDN to a single established connection to the server.
* Detecting and reporting broken networks is out of scope of this doucment, as are specific mechanisms for scenarios with multiple network links.
* The updated algorithem was described, including racing through the full handshake including TLS/QUIC, and the configurable values.
    * Q: How to select the resolver, e.g., to use DoH 
        * Draft currently silent on encrypted DNS selection
        * IPv6-Mostly has implications wrt encryoted resolvers and DNS64
        * Tommy will expand the draft to include considerations for non-default resolvers
    * Q: Waiting on SVCB response
        * Used to be that if you got AAAA first you went, otherwise if you got A first you waited a little, now you go if you get AAAA or an IPv6 address hint from SVCB you go, otherwise you wait a little for SVCB should that be required to make decisions like ECH.
        * Lorenzo - consider caching behavior. 
* Open Issues and discussion:
    * Impacts of integrating SVCB
        * Generally including description of what client does
        * Thorny bit is how to deal with sorting given prioritizations, or ALPN preference, etc. The "juicy bits"!
        * Lorenzo C: 6724 assumes you get all addresses and sort, but here you get them over time, and sort, and then may get another one - need clarity. You gather everything you have, sort, then if more comes in the existing connection attempts keep going, and you add extra addresses in order.
        * Q: What about historical sorting data - is it kept for ever? Should we consider keeping it for less long? Previous text on this was ambiguous as it's platform specific (may be part of the routing table). Agree should not be kept for long. 
        * Arrows on slide 14 are misleading. Any address with no route to it won't be in the list.
    * Sorting based on priorities
        * Prioritizing server priorities (SVCB) over family sorting
        * Should SVCB priority published by servers override local preferences around ALPN/ECH? Client choices contain security preferences, should they be valued over server? Historically successful connections need to be accounted for in priority too
        * The ECH-SVCB draft conflicts with what's currently in the HEv3 draft
        * Should SVCB priority be able to override preferred address family sorting (original HE)? Currently keeps reordering from original HEv2.
        * Comment: should try higher priority options first and give those some time to succeed before trying others.
        * Martin T: rule should be client preferences win. Server-based priority should override address family. There's also likely to be a preference for going for something that worked better in the past - but that should be lower than server priority to avoid potentially fixating on a poorer choice.
        * Open question of who is responsible for the security properties of the connection - a fun discussion!
    * IPv6-only/-Mostly
        * Should we rank NAT64 synthesized address lower?
        * How to handle DNS64 vs PREF64 vs both?
        * The hackathon IPv6 setup via Dave Plonka was very useful for testing.
        * Jen L: Goal is to eventually stop using DNS64 and have clients syntethize locally.
        * Comment: we can just pretend DNS64 doesn't exist.
        * Note difference in how to configure DNS vs how to handle addresses learnt.
        * Lorenzo C: think about the incentive on the servers.
    * Optimistic DNS (skimmed over due to time - need to consider whether/how to include it)
    * Do we need interim(s) or testing/hackathon setups?
    * Anything else?
        * Some discussion on client/host capabilities/functionalities - "low effort" network libraries may not be capable
        * Chairs asked Lorenzo to open an issue on this topic
* Next steps
    * The chairs solicited opinion on adopting the draft
    * Poll: I have read draft or prior version -- 26 yes, 15 no, 1 no opinion
    * Poll: Adopt? 42 yes, 0 no, 4 no opinion
    * Eric reminded attendees that the vote will be confirmed via the mail list.


## Chromium Implementation Status Update
Kenichi Ishibashi, 10 minutes

Slides: https://datatracker.ietf.org/meeting/122/materials/slides-122-happy-ietf122-happy-chromium-update-01-00

Brief overview of current behavior of Chromium and updates for the HEv3 implementation - noted different behaviour for TCP and QUIC in the previous version.

The HEv3 rework rewrites the connection attempt logic, sharing DNS resolutions for TCP and QUIC, and attempting connections without waiting for all DNS responses.

Early field experiment coming soon. 
* Slide 6, bullet 2 clarification: Race TCP and QUIC _until_ known QUIC will work
* Don't support non-default target names yet but do plan to support it later

Chairs will add a wiki area for implementor notes and any field measurements in the git repo.

Tommy should add a brief implementation section to his draft to point to the wiki (which is removed on final publication).



## Reporting Network Errors to Origins
Nic Jansma, Utkarsh Goel, 15 minutes

Slides: https://datatracker.ietf.org/meeting/122/materials/slides-122-happy-reporting-network-errors-to-origins-01

Examples of potential issues/problems with IPv6/DNS/QUIC/etc were presented.

HE may mask a variety of network issues - a connection may work, but the experience could be improved.

Looking for collaborators/co-authors for a draft on network error logging (NEL).

* Mic:
    * All of this gives you a lot of network topology info on the client, whereas before the server got less information. Hard to give enough info that doesn't cross privacy line. Don't see how this circle can be squared.
    * Tommy P: Where are the mtreics being reported and how?  To the server? The network operator? Aggregated? How real-time does this need to be vs delayed and aggregated? Given this would be an improvement over no feedback currently. Documenting the dimensions around metrics could be useful - what's interesting to measure?  Then we can figure out how to report them separately.
    * Lars E: Could we limit this? What about server says this is my preferred method and give feedback just on if that worked. Step back from all that's being proposed here.
    * Jordi: HE is good but we should have implemented reporting since day 1. It causes problems when deploying IPv6. We need to find a reporting protool that is already implemented by hosting services or ISPs or both, to make it easier to adopt. Not sure who should receive it - probably both the server hoster and the ISP - problems are usually in the ISP. 
    * Gorry F: This is good, even if no conclusion yet.
The chairs will seek AD advice on the privacy considerations for NEL that will need to be included in the proposed draft. 


## Summary and Areas for Coordination
Chairs, 10 minutes

* Adoption call sent to mailing list
* Thoughts on virtual interim(s)?
    * Eric suggested some possible topics - DNS server selection, server priority vs client priority, how to insta-fail, ECH/mixed-ECH support/policies and IPv6-only/Mostly networks. 
    * What other topics would be of interest?
        * We can make some progress, but we could aait a little bit until we can see what we're getting stuck on with specific github issues?
        * Two motivations for having one: take some leaps ahead; crossing over into other IETF groups (invite those people to interims, and/or invite them to github)
        * Avoid interims being editor meetings.
        * Suggestion to have any interim singularly focused, e.g., on v6ops/mostly/only, and a separate on one ECH/DNS.
        * Need to bear in mind not all platforms can do what's discussed here - getting the folks responsible for such systems into a room could be useful.
    * Concensus to give it a few weeks on github and then decide from there if interim needed to hash things out.
    * Areas of coordination - v6ops, 6man, tsvwg, quic, tls, httpbis and dnsop. If any are nissing, please shout to the chairs. 

Meeting closed.