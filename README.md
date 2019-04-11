# IETF104 LOOPS Side meeting

Time: 2019/03/27 13:45-15:00 Wednesday

The meeting started a bit late due to technical difficulties with the
projector; not all slides in the repo were actually visible on the
screen.

## Carsten's intro

(Slides 1 to 10)

While still struggling with the projector, Carsten gave a short
introduction to the LOOPS problem statement and opportunity, and
what's out of scope for now.

## Yizhou’s presentation

(Slides 11 to 22)

Yizhou Li presented the use case as seen by her, focusing on packet
loss as seen in various wide-area (long-haul) paths and how it can be
fixed by local recovery provided by cooperating overlay nodes on the
path.

David Black: There is a proposal floating around for detnet for an
SRv6 data plane.  If every SRv6 segment was a LOOPS segment, the loss
detection is not that interesting because if you ever drop a detnet
packet for congestion purposes, the network has clearly made an error.
However the timing measurement that LOOPS uses to disambiguate loss
versus error could be useful additional information to the detnet
control plane as an early warning that some chunk of the network is
not behaving and which chunk is not behaving.

Yizhou: With detnet, you know more about flow characteristics and
their requirements.

David: Suspect over time, some of those assumptions will get relaxed
when you figure out what you can get away with in an implementation.

Carsten: So the take-home message is: those measurements taken by
LOOPS segments may actually be useful for other purposes than the
original function of the LOOPS measurements.

David: That’s right. (More comments on how it is useful for a
particular segment of a SRv6 path to report such information to the
control plane.)

Carsten: In LOOPS, there is also the assumption that there is
something that sets up those paths (LOOPs), and that something could
get back some measurement information.
David: For detnet, that is a bit of a handwave; we don't know how
their control plane will look like, but that is not a LOOPS problem.

Tom reminds us about IOAM work; there may be some overlap or some
considerations.  They want the same thing, measure path
characteristics from point a to point b but the difference here is
what you do with that information.  We’are saying we want to put that
into a controller for retransmission.  They may share a lot of the
same information, which means there may be some commonality.

Carsten: What we just talked about is just an additional benefit — we
could get some measurements that can be relayed to a control plane.
But that’s not the purpose of this work.

Yizhou: IOAM might be overkill here, carrying a lot of metadata and
collecting information from other nodes in the path segment.
For LOOPS, the timestamp might be the most fundamental thing to carry;
I don't know yet.

Tom: We have the convergence of three concepts here: We have source
routing (segment routing), we have IOAM getting measurements from
intermediate nodes, and we have a control loop between intermediate
nodes.  There is a lot of commonality somewhere between those.  What
has to happen is [find out] how are these going to work together.  I
think it’s just keep an open mind here.  "This is not as simple as
building an e2e transport protocol."  [Some hilarity on this usage of
"simple" ensued...]

## Encapsulation layers

Carsten then pointed out that there are multiple proposals on how an
encapsulation layer for loops could look like.

### Sami’s presentation

(Slide 23)

Carsten coaxed Sami into saying a few words to his slide he quickly
put together.

Sami gave a recap on what his slide shows: A path segment with some
Geneve encapsulation that enables detection of packet loss.
A new Geneve option TLV can carry the necessary information (Carsten
then clarified that the specific design of this information is what
the Generic Information Model is about; the way this is represented in
the packet will differ between encapsulation formats).

Reverse information (such as ACKs) can be piggybacked onto data
packets that go in the reverse direction, if there are any.  Geneve
also provides a control-only Geneve tunneled packet that can be used
to transfer reverse information in case there is no actual payload
data to piggyback that on.

Geneve supports v4 and v6; there is a draft on SRv6 compatibility
(service function list).

Carsten summarized that this was a nice demonstration of why the LOOPS
work doesn't want to be in the business of defining the encapsulation
schemes.

### Tom’s presentation

(No slide)

Tom gave a quick review of how he would encapsulate this in GUE.
He stressed that the contents are important; encapsulation is just
wrapper.
This presents an opportunity to take best of TCP and QUIC and do
better.
Questions like 32 or 64 bits for sequence number.
Some extensibility.
Try to avoid TLVs.
An alternative might be to put this into some sort of extension
headers, more useful if we expect nodes on the path to be interested
in this information.

Carsten: So the fun part will be doing [the information model].
We don't have to meet all the checkmarks of an actual transport
protocol; we don't have to have all packets arrive, we just want to
optimize.

## Charter Discussion

(Slides 24 to 33)

Carsten briefly mentioned https://github.com/loops-wg as the place
where the charter will be further refined and presents the charter
proposal in a number of slides.

Carsten then clarified that LOOPS isn't just for encrypted traffic;
the point is that the protocol does not rely on peeking into the
packets (but there may still be some market differentiation based on
doing that for packets where it is still possible).

The specific text about how to present congestion events to the e2e
transport protocol is still a bit wishy-washy, as there are
optimization opportunities in that.

We want to talk to the owners of the e2e protocols to make sure we are
not doing something stupid, but LOOPS is designed to not peek into
those, so we don't have a TCP module and a QUIC module etc., treat all
traffic opaquely, no dependency.  On the other side we have various
tunneling protocols that we could use; based on how quickly we get
bindings to them, we will have to talk to the owners of those.

Carsten then presented the current, somewhat aggressive set of
milestones; there was no comment on that.

Carsten then started to present the "FAQ" slides (31..32).

Magnus asked what exactly was going to be the set of information from
the e2e flows that we would rely on; does "L3 only" include, e.g.,
UDP/TCP port numbers?
Carsten: Probably not beyond the port numbers.
Occasionally it may be useful to send packets with a wrong checksum,
so that may be a third field.

Carsten: Then, how do we know which packets are important and which aren't?
Generally, there will be no marking (except in certain service
provider scenarios) that we could rely on.

Carsten: How do we transport measurement-related information?  In the
forward direction, this is probably going to be identical to what we
already use to facilitate retransmission: sequence number, timestamp.
In the reverse direction, this will use the same mechanism we
transport ACKs, e.g. Geneve control-only packets, maybe some form of
no-op packets for other encapsulations.  With luck, we have a
symmetric flow, and can piggyback, but we need to cover the asymmetric
case as well.

Carsten: If you are reducing the MTU, adding latency by additional
processing, adding data — how do you know that you don't make the
situation worse?  The answer could be that the measurement layer can
provide information on this — if the path turns out to a perfect piece
of fiber, maybe LOOPS can be switched off.

Carsten: indicating congestion events via ECN or via dropping a packet
(e.g., not even requesting retransmission if a congestion event would
be relayed anyway.)

David: for congestion indication on transports that are not ECN
capable, we have a piece of the puzzle for you, a currently expired
draft: tunnel congestion notification
\[<https://tools.ietf.org/html/draft-ietf-tsvwg-tunnel-congestion-feedback>],
currently on hold in TSVWG; basic trick: Inner header says
non-ECT, outer header set to ECT(0), do what you want to do, when
decapsulating check whether a packet needs to be dropped.
This draft may be coming back now because we finally have a use case
for it, and it looks like LOOPS may be another one.

## Next Steps

(Slide 34)

With 5 seconds left on the agenda for discussion, Carsten went on to
next steps.  We already have a github organization ("loops-wg",
because we don't want to change the name once it actually is a WG),
but need a mailing list.  Magnus indicated that he would be willing to
create that \[mailing list now exists at <loops@ietf.org>, see
<https://www.ietf.org/mailman/listinfo/loops> to subscribe].

Carsten then asked if people think that going for a BOF in Montreal is
too aggressive; about a month after the IETF to get a sketch of a
solution ready, including sketches of bindings; does not need to be
complete but should be useful enough to indicate feasibility.

Yizhou opined that this might be a little bit hard doing this because
we need to have a generic information model first then sketch for
encapsulation bindings. But in case the information model was written,
the authors can cooperate tightly, then we may can carry on.
Carsten: Sami already had an idea of how the encapsulation would look
like before we even had started the work...

John pointed out that not all that needs to be done by the end of
April.  Even the sketch — do you need that to get the BOF approved?
Carsten opined that that depends on how trusting the IESG will be that
this can be done.  So it would be good to have those sketches.

Zahed: With respect to encapsulations, it might be helpful to do a gap
analysis what exactly is already there.  Might be helpful for a BOF,
to give an overview of what the gaps are.  Carsten: We don't want to
invent a new encapsulation protocol, but we do have to map the LOOPS
information to it.  Decisions like 16 bits vs. 24 bits; answer may be
different for different encapsulation protocols.  Carsten: An
encapsulation overview document would be very useful, but difficult to
produce and get consensus on with all these competing approaches; I
wouldn't want LOOPS to be the WG that tries to get consensus on such a
document — may be too contentious; would like to avoid.

## Polls

At the end, Carsten asked the usual chair questions:

* So, who would be interested in contribute to this sketch？\\
  Carsten, Sami, Yizhou, Joe, (Tom already left)

* Who would be interested in reviewing that? \\
  David, John, Marcus, Zahed (?)

The questions that would be asked at the end of the BOF:

* Do we think we have a problem that be understand, that is well
  defined that actually can be worked on?\\
  Floor: Yes (and no disagreement)

* Is that the work should be done in the IETF?\\
  Floor: Yes (and no disagreement)

Carsten thanked the attendees for coming.

## Attendees

| Name                      | Affiliation              |
|---------------------------|--------------------------|
| Carsten Bormann           | Universität Bremen       |
| Chi-Jiun Su               | Hughes Network Systems   |
| David Black               | Dell EMC                 |
| Hariharan Ananthakrishnan | Netflix                  |
| Jeffrey He                | Huawei                   |
| John Border               | Hughes Network Systems   |
| Jörg Deutschmann          | FAU Erlangen             |
| Liang Geng                | China Mobile             |
| Magnus Westerlund         | Ericsson                 |
| Marcus Ihlar              | Ericsson                 |
| Mauro Cociglio            | Telecom Italia           |
| Sami Boutros              | VMWare                   |
| Szilveszter Nádas         | Ericsson                 |
| Tobias Guggemos           | LMU University of Munich |
| Tom Herbert               | Quantonium               |
| Xingwang Zhou             | Huawei                   |
| Yizhou Li                 | Huawei                   |
| Zaheduzzaman Sarker       | Ericsson                 |

