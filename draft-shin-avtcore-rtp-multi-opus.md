---
title: "RTP/SDP for Opus Multistream"
abbrev: MULTI-OPUS
docname: draft-shin-avtcore-rtp-multi-opus-latest
date: 2026-04-16
category: info

ipr: trust200902
area: "Web and Internet Transport"
submissionType: IETF
workgroup: "Audio/Video Transport Core Maintenance"
keyword: Internet-Draft

stand_alone: yes
smart_quotes: no
pi: [toc, sortrefs, symrefs, docmapping]

author:
  -
    ins: S. Shin
    name: Sun Shin
    organization: NVIDIA
    email: sushin@nvidia.com

informative:
  libwebrtc:
    target: https://webrtc-review.googlesource.com/c/src/+/129768
    title: Opus multistream mapping
    author:
      org: WebRTC

--- abstract

This document specifies RTP/SDP signaling for Opus multistream (multi‑channel)
operation, enabling negotiation of layouts such as 5.1 and 7.1 in real‑time communications.
It defines an SDP encoding name and format parameters to
describe multistream configurations, and specifies Offer/Answer procedures
for interoperable negotiation. This document does not define new Opus codec behavior.
It extends the the SDP signaling defined in {{?RFC7587}} and reuses the channel‑mapping
semantics defined in {{?RFC7845}}.

--- middle


# Introduction

Opus ({{?RFC6716}}) supports up to 255 channels via multistream encoding with
explicit channel mapping. The RTP payload format for Opus ({{?RFC7587}}),
however, standardizes only mono and stereo signaling and does not dfeind for
RTP/SDP for multistream operation.

{{?RFC7845}} defines channel‑mapping families for Opus carried
in the Ogg container, assigning semantic meaning to decoded output channels
(e.g., speaker positions or spherical harmonics). Theses mapping families
do not alter the Opus bitstream or RTP payload format, but define how decoded
channels are to be interpreted.

This document fills that gap by specifying interoperable SDP signaling and
Offer/Answer procedures for multistream operations in RTP sessions,
while reusing the channel mapping semantics defined in {{?RFC7845}}.

This document is limited to RTP/SDP signaling for Opus multistream configurations
already defined in {{?RFC6716}}, {{?RFC7587}}, and {{?RFC7845}}. It does not define
new Opus codec features, bitstream formats, or decoding behavior.

The MLCODEC Working Group is defining extensions to the Opus codec that
may introduce new codec capabilities. Such extensions are out of scope
for this document. Future Opus codec extensions may reuse the SDP
signaling framework defined here, subject to the constraints of the
extension specification.

# Relationship to Existing RFCs

This section summarizes the scope and relationship between {{?RFC6716}} (Opus codec),
{{?RFC7587}} (Opus over RTP), {{?RFC7845}} (Ogg encapsulation of Opus), and this draft.
While {{?RFC7845}} defines channel mapping families for multistream Opus in the Ogg
container, it does not define SDP signaling or RTP usage. {{?RFC7587}} defines the
RTP payload format for Opus but only covers mono/stereo signaling. This draft
extends {{?RFC7587}} to define SDP signaling for multistream Opus in RTP sessions
and reuses the mapping semantics from {{?RFC7845}}.

# Relationship of RFCs and This Draft:

      +----------------+     +-------------------+     +----------------+
      |   RFC 6716     |     |     RFC 7845      |     |   RFC 7587     |
      |  Opus Codec    |     | Ogg Encapsulation |     | Opus over RTP  |
      +----------------+     +-------------------+     +----------------+
               |                       |                        |
               |                       |                        |
               +-----------------------+------------------------+
                                       |
                                       v
                       +------------------------------+
                       |  This Draft (Multi-Opus RTP) |
                       |  SDP Signaling + O/A Rules   |
                       +------------------------------+

# Summary of RFCs and This Draft:

     +------------+----------------------+-----------------------+-----------------------+
     | RFC/Draft  | Scope                | Defines Channel Map?  | Defines SDP Signaling |
     +------------+----------------------+-----------------------+-----------------------+
     | RFC 6716   | Opus codec           | Yes (API level)       | No                    |
     | RFC 7845   | Ogg encapsulation    | Yes (families)        | No                    |
     | RFC 7587   | RTP payload format   | No (mono/stereo only) | Yes (mono/stereo)     |
     | This Draft | RTP multistream SDP  | Reuses RFC 7845       | Yes (multi-channel)   |
     +------------+----------------------+-----------------------+-----------------------+

# Overview and Rationale

Deployed systems (e.g., {{libwebrtc}} based) interoperate using a non‑standard
SDP encoding name “multiopus” with fmtp parameters such as num_streams,
coupled_streams, and channel_mapping.
Standardizing these semantics improves interoperability and removes the need
for application‑level SDP text modifications.

# SDP Signaling for Opus Multistream

## SDP Syntax for Multichannel Opus

### SDP Example for 5.1 Audio

```sdp
a=rtpmap:111 multiopus/48000/6
a=fmtp:111 num_streams=4;coupled_streams=2;channel_mapping=0,4,1,2,3,5
```

### SDP Example for 7.1 Audio

```sdp
a=rtpmap:111 multiopus/48000/8
a=fmtp:111 num_streams=5;coupled_streams=3;channel_mapping=0,6,1,2,3,4,5,7
```

###  Mapping Families and Extensibility

Channel mapping families define the semantic interpretation of decoded
output channels and do not affect the RTP payload format beyond
establishing the number of streams and output channels.

This document normatively specifies SDP signaling for mapping family 1
(channel‑based speaker layouts) as defined in {{?RFC7845}}. The SDP signaling
mechanism is designed to be extensible to additional mapping families
(e.g., Ambisonics or externally defined mappings) without changes to the
RTP payload format.

An endpoint that receives an offer specifying a mapping family it does
not support MUST reject that payload type during Offer/Answer
negotiation.

### Field Descriptions

- `a=rtpmap:<pt> multiopus/48000/<channel-count>`
  - `<pt>`: Dynamic payload type (e.g., 96).
  - `48000`: Fixed clock rate for Opus.
  - `<channel-count>`: Total number of output channels resulting from decoding.

- `a=fmtp:<pt> num_streams=<N>;coupled_streams=<M>;channel_mapping=<C>`
  - `num_streams`: Total number of Opus streams.
  - `coupled_streams`: Number of stereo (coupled) streams.
  - `channel_mapping`: Comma-separated list defining the mapping from decoded
channels to semantic output positions, following the channel mapping semantics
defined in {{?RFC7845}}.

### Field Presence and Constraints

The "channel_mapping" parameter is REQUIRED for configurations with more
than two output channels.

For configurations with a single stream and one or two output channels,
"channel_mapping" MAY be omitted. If omitted, the receiver MUST interpret
the configuration as equivalent to {{?RFC7845}} mapping family 0 semantics
for mono or stereo output.

The values of "num_streams", "coupled_streams", and each entry in
"channel_mapping" MUST be integers in the range 0..255.

For mapping family 1, the number of output channels MUST NOT exceed 8.

# Offer/Answer Procedures

An offerer willing to negotiate multichannel Opus MAY include one or more
payload types using multiopus with appropriate fmtp parameters, and
SHOULD include a stereo alternative using opus/48000/2 ({{?RFC7587}}) for
backward compatibility.

An answerer that supports the offered configuration if it can parse the offered
paramters and decode the resulting multichannel audio without loss of the
negotiated channel semantics.

If the answer supports the offered multiopus configration, it MUST select
the corresponding payload type and include the selected multistream parameters
in the answer.

If unsupported, the answerer MAY select a stereo opus payload or reject
the m‑section per {{?RFC3264}}. An answer that supports the offered configurations
SHOULD NOT silently down-convert the decoded output to streo without indicating
that behavor to the application.

## Examples

### Offer: 5.1 (6 channels)

m=audio 9 UDP/TLS/RTP/SAVPF 111 112
a=mid:audio
a=rtpmap:111 multiopus/48000/6
a=fmtp:111 num_streams=4;coupled_streams=2;channel_mapping=0,4,1,2,3,5
a=rtpmap:112 opus/48000/2
a=sendrecv

### Answer: accept 5.1

m=audio 9 UDP/TLS/RTP/SAVPF 111
a=mid:audio
a=rtpmap:111 multiopus/48000/6
a=fmtp:111 num_streams=4;coupled_streams=2;channel_mapping=0,4,1,2,3,5
a=sendrecv

## Security Considerations

The use of the `multiopus` encoding in SDP does not introduce new security concerns beyond those already
described in {{?RFC7587}}. Implementers should ensure that SDP parsing and RTP payload handling are robust
against malformed or malicious input. Applications using multichannel Opus streams must also consider
the privacy implications of transmitting spatial audio data, which may reveal environmental context.

Transport-level security mechanisms such as DTLS-SRTP are recommended to protect RTP streams.

## IANA Considerations

This document does not require any new IANA registrations. The `multiopus` encoding name and associated
SDP attributes are used in accordance with existing conventions and do not introduce new protocol elements.
