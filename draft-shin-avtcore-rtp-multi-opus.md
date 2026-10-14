---
title: "RTP/SDP Signaling for Opus Multistream Operation"
abbrev: OPUS-MULTISTREAM
docname: draft-shin-avtcore-rtp-multi-opus-latest
date: 2026-10-14
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

This document specifies RTP/SDP signaling for Opus multistream (multichannel) operation,
enabling negotiation of layouts such as 5.1 and 7.1 in real-time communications.
It defines SDP format parameters for describing multistream configurations and specifies
Offer/Answer procedures for interoperable negotiation of multichannel Opus sessions.
This document does not define new Opus codec behavior. Rather, it standardizes
SDP signaling for Opus multistream operation while continuing to use the Opus RTP payload
format defined in {{?RFC7587}} and the channel mapping semantics defined in {{?RFC7845}}.
The signaling procedures specified in this document are intended to provide
a standardized alternative to deployment-specific approaches that have historically
been referred to as "multiopus".

--- middle


# Introduction

Opus ({{?RFC6716}}) supports multistream encoding and channel mapping, enabling representation
of multichannel audio configurations beyond mono and stereo. The RTP payload format for Opus
defined in {{?RFC7587}} specifies signaling and transport procedures for Opus in RTP sessions,
but primarily focuses on mono and stereo operation and does not define SDP signaling
procedures for multistream multichannel configurations.

{{?RFC7845}} defines channel mapping families for Opus carried in the Ogg container format.
These mapping families assign semantic meaning to decoded output channels, such as speaker
positions for surround sound layouts. The channel mappings do not alter the Opus bitstream
or RTP payload format but define how decoded channels are to be interpreted by receivers.

Several deployed implementations have supported multichannel Opus using implementation-specific
SDP signaling approaches, commonly referred to as "multiopus". While these approaches have
enabled deployment of surround-sound audio in practical systems, they have not been standardized
and may present interoperability challenges across implementations.

This document specifies interoperable SDP signaling and Offer/Answer procedures for Opus
multistream operation in RTP sessions while continuing to use the Opus RTP payload format
defined in {{?RFC7587}} and the channel mapping semantics defined in {{?RFC7845}}.

This document is limited to RTP/SDP signaling for Opus multistream configurations already
supported by {{?RFC6716}}, {{?RFC7587}}, and {{?RFC7845}}. It does not define new Opus codec
features, bitstream formats, RTP payload formats, or decoding behavior.

The MLCODEC Working Group is defining extensions to the Opus codec that may introduce
additional codec capabilities. Such extensions are outside the scope of this document.
Future Opus extensions may reuse the SDP signaling framework defined here, subject to
the requirements and constraints of their respective specifications.

# Relationship to Existing RFCs

This section summarizes the scope and relationship between {{?RFC6716}} (Opus codec),
{{?RFC7587}} (Opus over RTP), {{?RFC7845}} (Ogg encapsulation of Opus), and this document.

{{?RFC6716}} defines the Opus codec, including support for multistream encoding and
channel mapping.

{{?RFC7587}} defines RTP transport and SDP signaling for Opus in RTP sessions but
primarily addresses mono and stereo operation.

{{?RFC7845}} defines channel mapping families for Opus carried in the Ogg container format.
These mappings provide semantic interpretation of decoded output channels but do not
define RTP usage or SDP signaling procedures.

This document specifies SDP signaling and Offer/Answer procedures for Opus multistream operation
in RTP sessions while continuing to use the RTP payload format defined in {{?RFC7587}} and
the channel mapping semantics defined in {{?RFC7845}}.

# Relationship of RFCs and This Draft

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
                       |         This Document        |
                       |      Opus Multistream SDP    |
                       |      Signaling Framework     |
                       +------------------------------+

# Summary of RFCs and This Document

     +------------+------------------------+-----------------------+-----------------------+
     | RFC/Doc    | Primary Scope          | Channel Mapping       | SDP Signaling         |
     +------------+------------------------+-----------------------+-----------------------+
     | RFC 6716   | Opus codec             | Yes                   | No                    |
     | RFC 7845   | Ogg encapsulation      | Yes (families)        | No                    |
     | RFC 7587   | Opus RTP payload       | Mono/Stereo only      | Yes (mono/stereo)     |
     | This Doc   | Multistream SDP rules  | Reuses RFC 7845       | Yes (multichannel)    |
     +------------+------------------------+-----------------------+-----------------------+

# Overview and Rationale

Opus supports multistream operation and channel mappings that enable multichannel audio
configurations such as 5.1 and 7.1. While such configurations have been deployed in practice,
interoperable SDP signaling for these configurations has not been standardized for RTP sessions.

Several deployed implementations (e.g., {{libwebrtc}} based) have historically used
implementation-specific SDP signaling approaches, commonly referred to as "multiopus",
together with parameters such as num_streams, coupled_streams, and channel_mapping.
Although these approaches have enabled deployment of multichannel Opus,
they are not standardized and may lead to interoperability challenges.

This document standardizes SDP signaling semantics for Opus multistream operation while
continuing to use the Opus RTP payload format defined in {{?RFC7587}}. Standardizing these semantics
improves interoperability and removes the need for application-specific SDP modifications.

# SDP Parameters for Opus Multistream

## SDP Example for 5.1 Audio

```sdp
a=rtpmap:111 opus/48000/6
a=fmtp:111 num_streams=4;coupled_streams=2;channel_mapping=0,4,1,2,3,5
```

## SDP Example for 7.1 Audio

```sdp
a=rtpmap:111 opus/48000/8
a=fmtp:111 num_streams=5;coupled_streams=3;channel_mapping=0,6,1,2,3,4,5,7
```

## Mapping Families and Extensibility

Channel mapping families define the semantic interpretation of decoded output channels
and do not alter the Opus bitstream or RTP payload format.

This document normatively specifies SDP signaling for Mapping Family 1
(channel-based speaker layouts) as defined in {{?RFC7845}}.

The signaling framework is designed to be extensible to additional mapping families,
including Ambisonics or externally defined mappings, without requiring changes to
the underlying Opus RTP payload format.

An endpoint that receives an offer specifying a mapping family it does not support
MUST reject the corresponding payload type during Offer/Answer negotiation.

## Field Descriptions

- `a=rtpmap:<pt> opus/48000/<channel-count>`
  - `<pt>`: Dynamic payload type (e.g., 96).
  - `48000`: Fixed clock rate for Opus.
  - `<channel-count>`: Total number of output channels resulting from decoding.

- `a=fmtp:<pt> num_streams=<N>;coupled_streams=<M>;channel_mapping=<C>`
  - `num_streams`: Total number of Opus streams.
  - `coupled_streams`: Number of stereo (coupled) streams.
  - `channel_mapping`: Comma-separated list defining the mapping from decoded
channels to semantic output positions according to the channel mapping semantics
defined in {{?RFC7845}}.

## Field Presence and Constraints

The "channel_mapping" parameter MUST be present when the number of
output channels exceeds two.

For configurations with one or two output channels, the channel_mapping parameter
MAY be omitted. If omitted, the receiver MUST interpret the configuration according
to Mapping Family 0 semantics defined in {{?RFC7845}}.

The values of "num_streams", "coupled_streams", and each entry in "channel_mapping"
MUST be integers in the range 0 to 255.

For Mapping Family 1, the number of output channels MUST NOT exceed eight.

# Offer/Answer Procedures

An offerer willing to negotiate multichannel Opus MAY include one or more
Opus multistream payload configurations with the appropriate format parameters.
For backward compatibility, an offerer SHOULD also include a stereo alternative
using opus/48000/2 as defined in {{?RFC7587}}.

An answerer MAY accept an offered multistream configuration if it can parse
the offered parameters and decode the resulting multichannel audio without loss
of the negotiated channel semantics.

If the answerer supports the offered multistream Opus configuration, it MUST select
the corresponding payload type and include the selected parameters in the answer.

If the offered configuration is not supported, the answerer MAY select a stereo
Opus payload type or reject the media section according to {{?RFC3264}}.

An answerer SHOULD NOT silently down-convert decoded output to stereo without
providing an indication of that behavior to the application.

## Examples

### Offer: 5.1 Audio (6 channels)

```sdp
m=audio 9 UDP/TLS/RTP/SAVPF 111 112
a=mid:audio
a=rtpmap:111 opus/48000/6
a=fmtp:111 num_streams=4;coupled_streams=2;channel_mapping=0,4,1,2,3,5
a=rtpmap:112 opus/48000/2
a=sendrecv
```

### Answer: Accept 5.1 Audio

```sdp
m=audio 9 UDP/TLS/RTP/SAVPF 111
a=mid:audio
a=rtpmap:111 opus/48000/6
a=fmtp:111 num_streams=4;coupled_streams=2;channel_mapping=0,4,1,2,3,5
a=sendrecv
```

The offer above advertises both a multistream Opus configuration and a stereo Opus fallback
for backward compatibility. The answer selects the multistream configuration and echoes
the negotiated parameters, indicating support for the offered 5.1-channel layout.

# Security Considerations

The SDP signaling procedures defined in this document do not introduce security considerations
beyond those already described in {{?RFC7587}}.

Implementations SHOULD ensure that SDP parsing and RTP payload handling are robust against
malformed or malicious input. Implementations SHOULD validate multistream configuration parameters,
including channel mappings, stream counts, and coupled-stream counts, before use.

Applications using multichannel Opus streams should also consider the privacy implications of
transmitting spatial audio information, which may reveal environmental or application context
beyond that conveyed by mono or stereo audio.

Transport-level security mechanisms such as DTLS-SRTP are RECOMMENDED to protect RTP streams
from unauthorized access or modification.

# IANA Considerations

This document does not request registration of a new RTP payload format or media subtype.

The SDP parameters defined by this specification are intended for use with the existing
Opus RTP payload format defined in {{?RFC7587}}.

No IANA action is required by this document.

# Acknowledgements

The author would like to thank Philipp Hancke for his detailed review and constructive
feedback during the AVTCORE adoption discussion. His comments helped shape the direction
of this document from standardizing a deployment-specific "multiopus" signaling approach
toward specifying interoperable SDP signaling for Opus multistream operation while
continuing to use the Opus RTP payload format defined in {{?RFC7587}}.

The author would also like to thank Hyunsik Yang for his careful review of the document and
for identifying editorial issues and textual improvements during the adoption process.

The author further thanks Erik Språng, Timothy Terriberry, Mo Zanaty, the AVTCORE chairs,
and other working group participants for their review, comments, and support.
