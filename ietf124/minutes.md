# IETF 124 Montreal
happy WG Meeting
Thursday 6 November 2025, Centre Ville room

happy WG: https://datatracker.ietf.org/wg/happy/about/
Chairs: Eric Kinnear, Tim Chown
Github repo: https://github.com/ietf-wg-happy/


## Introduction, Chairs, 10 minutes
Slides: https://datatracker.ietf.org/meeting/124/materials/slides-124-happy-chair-00

Eric reminded people about the new Note Well and introduced the agenda.


## draft-ietf-happy-happyeyeballs-v3, Tommy Pauly, Nidhi Jaju, 80 minutes

Draft doc: https://datatracker.ietf.org/doc/draft-ietf-happy-happyeyeballs-v3/
Slides: https://datatracker.ietf.org/meeting/124/materials/slides-124-happy-happy-eyeballs-v3-00

(Nidhi speaking)

Tommy (on slide 9, issue #101): 
As coauthor, I think it's useful to highligh this because it's a topic that has come up in discussion and takes an opinionated interpration of SVCB spec. There are a number of cases where it seems unecessary to wait if it doesn't make a difference. There are things that may impact the handshake, but only in performance aspects not in security aspects (like a downgrade).

Eric Nygren: The svcb draft differentiates optional vs required clients. If you are an optional client there's no need to wait. cases of the HTTPS scheme I don't recall the draft suggeting it but I think it's a good idea, and we may want to update the SVCB draft and update it if it doesn't mention it.

(Tommy speaking)

Slide 16-17

Jen Linkova: 
For terminology we should not say v6 mostly. v6 only is a corner case of dual stack with zero v4.

For slide 17, maybe we want to enumerate scenarios and talk about what result we want achieve. The current text seems confusing.

the draft does not say currently if this is per interface or not. If it's per interface, then VPN case disappears as it's just another interface.o

Tommy: this brings us to the complexity of split DNS views. I'm inclided we need to cover the multiple interface case. I like the idea of starting with the goals.

Philipp Tiesel: I think this PR is still too complex. We need to handle ipv4 addresses disguised as v6 addresses as v4. Detect DNS 64 generated IP addresses. The question of if we synthesize or not is an implementation decision. We may want to give guidance.

Tommy: I'm getting the sense we should have a small group work on a PR for this


Lorenzo Colitti: The question of v4 vs v6 socket, arguably this doesn't belong here, it should be in 6724 bisbis. We could solve it here, but it wouldn't solve it for non-happy eyeballs situation. 

Tommy: I think the mechanism for the synthesis should be mentioned but not elaborated upon.

Eric K: Are we thinking design team or bisbis?

Jen Linkova: We can have things defined in 6724, because that's just defaults. Applications can do differently if they think it is a good idea.

Erik Nygren: This came up in DNSop this week and the conclusion ended up with 6724 bis bis. Would be nice to have a common resource.

Tommy: I'd appreciate help in coordinating the common solution

Philipp Tiesel: yes design team and 

Lorenzo: Could we claw back 6724 bis and add some more things? 

Jen Linkova: We need to take this discussion offline or at 6man. I'm not ready to say which option is better, we should talk about it.

Eric K: maybe sneak in as time permits in 6man.

Slide 20

Lorenzo Colitti: "The network"? 

Tommy: If the route goes away the historical data goes away. 

Lorenzo: we should change the words the network.

Jen Linkova: If you reconnect to the same wfii later do you flush the data or do you remember?

Tommy: currently we lose that since we don't remember it and we don't know a reasonable time frame to cache it.

Lars Eggert: We should separate out what needs to happen for privacy and what needs to happen for performance. Reattaching to a recently used interface, we should allow persisting this data. Might get complex fast.

Gorry Fiarhurst: I don't get the connection between the first part and the second part. The SHOULD doesn't make sense to me. The SHOULD is because? I don't see the reasons.

Eric K: Second what Lars says, we should be explicit about the privacy reasons.

Slide 21

Lars Eggert: Why not a MUST? Is there ever a case where we use the TCP connection and the TLS handshake fails?

Tommy: Maybe some case where you don't have this control.

Lars Eggert: Make it explicit as a SHOULD with here's why you may not be able to...

Philip Tiesel: Yes "DEFINITELY SHOULD", there are implementations where it's hard to implement and there are enough practical problems with things breaking TLS.

Lars Eggert: If TLS doesn't work, you can't use it. You don't like Must?

Tommy: "If you can you MUST"

Jonathan Lennox: If you're on a constrained device and want to avoid the cost of multiple handshakes.

Tommy: This gets to the point where you push out timers. 

Philip Tiesel: We don't want to make it hard to implement HEv3.


Slide 22:

Erik Nygren: Might be a bad idea. Maybe a new binding service parameter for? to pass the need for what the application wants?

Tommy: For figuring what the application wants has come up multiple times. Sometimes the client has this context and sometimes the server does.

Erik Nygren: No strong opinion.

Tommy Jensen: what should HEv3 say and what do applications want to do? These seem different. If applications always want to do QUIC then it's not doing HEv3.

Jen Linkova: I think this could be useful. I've seen cases where network breaks udp connection after handshake.

Tommy: This handshake aspect might be addressed by doing the race at the higher layer (e.g. wait for tls, or wait until you have the WebSocket like thing)

Stuart Cheshire: 

As the person who planted the seed of this, at the IETF in dublin 20 years ago. We had a long debate about picking the best connection and that led to many debates. A connection that works is better than one that doesn't. If there was a theoretical faster connection wasn't something we care. We need multipath protocols. Happy Eyeballs doesn't replace multipath.

Lucas Pardue: Who cancels what and when? There are a lot timers that I'm not sure are in the realm of happy eyeballs. 

Eric K (no hat): There are some places where we do this like alt service. We have seen metrics where this is a good idea.

Eric K(chair hat): Not sure if the current framing is within charter. Do we want a way to keep the other connection around as an escape from happy eyeballs.

Tommy: If you really liked QUIC, nothing stops you from doing this the next time around.

Matt Joras: 

We are in the position of don't cancel. Our mobile stack aggressivly prefer quic. They will keep trying until we hit QUIC. The speed to acquire a connection isn't necessarily correllated with how good that connection is.

We've seen in our data that failed initial connection can be pretty flaky if it was the first connection. The thing you started first was correllated to a chaotic app start event.

If you don't do this, we notice double digit changes in the amount we use QUIC. We should be careful about correlating the connection establishment with how it does longer term.

Tommy:

Would be great if you could share data and the algorithm behind the data. The thing that worked in the moment is best as you can't predict the future. In the future nothing says you can't keep trying.

There could be tuning with the app launch that you could do before doing something like never cancelling the quic connection.

Matt:

Our alg is pretty simple. We try QUIC first, we will try tcp if that fails. If a later connection comes along we try quic again if that works, we move everything to QUIC. The experience is not just one connection it's about the whole session experience.

Tommy:

One attempt ends with one connection. Each connection provides you with a new attempt.

Stuart:

I want to make a quick follow up. We should investigate what's going. I would like to investigate that bug. Without happy eyeballs it sounds like QUIC would just fail. Speaking as someone from apple who supports app developers.

Marco:
This fits with the happy eyeballs at a higher layer. For example racing a GET request.

Nidhi Jaju: If Chrome sees Alt-svc we will use QUIC again in the future. We don't cancel the attempt.

Eric K: wants to see data for when QUIC failed the race, but performs better.

Lars Eggert: Answer should be you SHOULD cancel unless you know better.

Lucas Pardue:
Would like an error code for cancelling due to losing race (e.g. QUIC error code).

Gorry Fairhurst: If we're doing SHOULD cancel we should explain why.

slide 23:

Stuart: Optimistic DNS relies on happy eyeballs. 


Max Inden (Mozilla):
We already do this.

Alessandro Ghedini: 
I don't think it's a bad idea, but tricky in certain edge cases.

There is a case where you use a stale record with an expired ECH config.

You might get into a case where it completes with the old record but takes longer because you had to do two handshakes.

If the client has to wait for the full TLS connection then it would work.

I'm not sure if the HEv3 draft becomes too complicated with this

Johnathon Lennox:
Do you do anything with the connection before the DNS response comes back? Like sending user data. Maybe in the clear text case (? scribe doesn't follow)

Stuart:
If you migrate your server to one piece of hardware to another and you update the DNS. There is still the cache that needs to be flushed out.

We had lots of meeting at Apple that covered these same edge cases. You can address this by setting the TTL to half.

I believe we had a limit of 7 days.

There are some non-obvious implications of this. Normally you wouldn't set your TTL too short because you want the caching aspect.. But with optimistic DNS you can now use lower TTLs

Tommy:
I've not seen instances of breakage in the many years we've been doing this.

Philipp Tiesel:
Cloud native systems, it might be a bad idea. As your IP address may be recycled when something restarts. How much penalty do you pay for that?

Erik N:
7 days seems too long. 30 min fine. Operationally is annoying as clients keep connecting to a machine.

You might be sending your sni to a different company.

Maybe DNS prefreshing could give you some of this benefit (but requires knowing what you are going to do)

Nidhi J:
We've done experiments with this in Chrome and it seems positive. But not sure how long it's safe to do this.

Jen Linkova:
clarifying q: Data is not kept between network interfaces.

Tommy:
cached on the level of resolver config.

Lorenzo C:
A lot of issues can be resolved by picking the right time interval.

Eric K (no hats): What to do if I'm a server and want to drain clients. Maybe some text that would help this case.

Tommy Jensen:

security considerations discussed in RFC 8767. We should write our own, but lean on that RFC.


## Metrics for Happy Eyeballs, Gautam Akiwate, 10 minutes

(HEv3 Configurable Values and how to Tune Them)
Slides: https://datatracker.ietf.org/meeting/124/materials/slides-124-happy-hev3-configurable-values-and-how-to-tune-them-01


Vaibhav Bajpai:

These are answers we can help answers as data collecting people.

With the resolution delay, consider the fact that we know have encrypted DNS.

What's the network? Most of these values will not work for starlink for example.

Gautam: We have the data, but not sure what we should look at. How does the tradeoff look like? How much lower latency for lower v6 connections?

Would be nice if we could get a list of things we care about and we could look a the data from data.

Erik N:
Would be helpful to be explicit on why we are doing this for each of these values.
v4 vs v6 for example is an architectural decision.

Brian Trammell:
These numbers we choose will be stored in different weird ways. These numbers are baked into the internet. That's a good reason to not change it.

Would be interested to see which are tied to physics vs politics vs existing practices.

Tommy Pauly:

Slide 8:

We may need to get new measurements as the observation method changes.
A lot of it boils down to "If you knew the future what would you have done?".

Highlight Brian's point. Maybe there are cliffs to be aware of.

Vaibhav Bajpai:
I would prefer no fixed timers in the doc.

Johannes Zirngibl:
50 ms only in Apple. Firefox and chrome do not.


## Happy Eyeballs Webtester, Johannes Zirngibl, 10 minutes

Slides: https://datatracker.ietf.org/meeting/124/materials/slides-124-happy-happy-eyeballs-webtester-00

Tim talks about Error reporting.

What does the WG want to do for error reporting? What it is we want to report, Who we report to, and how? Along with all the privacy issues around this.

Jordi Palet Martinez
We can make a version 0 draft even to just summarize the mailing list discussion.

Tommy:
Different option documents for different mechanism approaches.

I'm interested in an ICMP option approach. Not volunteering to write that. If that's something someone would liek to talk about and pen that would be helpful.

Mirja K:
Maybe would be helpful to define a qlog format for it.


Philip T:
Wouldn't mind helping pen this.


Closing and Next Steps, Chairs, 5 minutes

