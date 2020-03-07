# rtcweb-interop
Infrastructure and notes for interop testing of rtcweb stacks at the IETF 107 hackathon.
## Goals
Test interop between the various RTCWeb wire protocol implementations.
All of them test against libwebRTC in one form or another, but rarely if ever test against each other
This hackathon aims to help remedy that
## Non-goals
This aspires to _not_ be an SDP interop session. We aim to spend time on ICE, STUN, TURN, DTLS, SRTP, SCTP etc.
## Infrastructure
Minimal. The idea is that we focus on the rtcweb wire protocol. With that in mind I'm setting up a simple websocket service
that sends JSON between endpoints. The JSON will represent offer, answer and candidates. Endpoints self-identify with unique UUIDs to the service. Other endpoints can then send offers etc to their selected oposite numbers.
(see https://github.com/pipe/PodCall/rendezvous.md for a description).
So each implementation would need to write a shim layer to talk to this websocket service, which would then forward the JSON to the peer device.

## Assumptions 
Lets take the following simplifying assumptions (at least for 107 - we can get clever once we get this to work):
### FULL ICE 
discuss - ICE lite?
### BUNDLE
### RTCPMUX
Together these mean that we are only dealing with a single 'connection' for each session
### Opus
### H264
At this stage I don't think we want to be interop testing codec selection - let's assume the most common codecs
### One audio and one Video track only
Baby steps - no simulcast, Plan B or other complexities, yet.
### DTLS/SRTP
### Datachannel TBD
For implementations that support datachannels, we need some way of agreeing what to expect. |pipe| puts a URI into the lable to indicate the purpose and semantics of a new datachannel - discuss....


## Discussion of O/A JSON format for minimal signalling
What should the exchanged JSON contain? - PodCall ignores the problem by just sending SDP with 'to', 'from' and 'session' added. |pipe| on the other hand sends JSON that looks a bit like the XMPP jingle spec, but JSON-ified.
It seems to me that we could take 2 Lowest common denominator tactics:
  1) Raw SDP : Even though many of the implementations don't do SDP natively - they all have tools to allow SDP creation and consumption in order to support libwebRTC/chrome/safari etc - however SDP is a trap....
  2) Need to Know: If you take a set of reasonable assumptions above it turns out you only need to exchange about 4 things in an O/A
the NtK JSON would exchange just those, then implemenations's shims would template this minimal info up into whatever they natively consume. (and extract in the answer phase)

## NtK minimal info
  to: self-declared id of sender
  from: self-declared id of recipient
  session: unique session id (eg timestamp of 1st offer)
  type: offer/answer/candidate
for offer and answer we need
  fingerprint: dtls fingerprint
  ufrag: 
  upass:
for candidates we just need
  candidate: (full candidate line)  
  
Thoughts ?

