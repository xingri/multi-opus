---
title: "RTP/SDP for Opus Multistream"
abbrev: MULTI-OPUS
docname: draft-shin-avtcore-rtp-mult-opus-latest
date: {DATE}
category: info

ipr: trust200902
area: General
submissionType: IETF
workgroup: Independent Submission
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

--- abstract

This document specifies RTP/SDP signaling for Opus multistream (multi‑channel)
operation, enabling negotiation of layouts such as 5.1 and 7.1 in real‑time communications. It defines an SDP encoding name and format parameters to
describe multistream configurations, and specifies Offer/Answer procedures
for interoperable negotiation. This document extends the Opus RTP
payload format defined in RFC 7587 and reuses the channel‑mapping
concepts defined for the Ogg container in RFC 7845.

--- middle


# Introduction

Opus (RFC 6716) supports up to 255 channels via multistream with explicit
channel mapping. The RTP payload format for Opus (RFC 7587), however,
standardizes only mono/stereo signaling for RTP/SDP and leaves multistream
out of scope. RFC 7845 defines channel‑mapping families for Opus carried
in the Ogg container, but it does not define RTP/SDP signaling or
Offer/Answer behavior. This document fills that gap by specifying
interoperable SDP signaling and Offer/Answer procedures for multistream
Opus in RTP sessions, while aligning channel‑mapping semantics with RFC 7845.

# Relationship to Existing RFCs

This section summarizes the scope and relationship between RFC 6716 (Opus codec),
RFC 7587 (Opus over RTP), RFC 7845 (Ogg encapsulation of Opus), and this draft.
While RFC 7845 defines channel mapping families for multistream Opus in the Ogg
container, it does not define SDP signaling or RTP usage. RFC 7587 defines the
RTP payload format for Opus but only covers mono/stereo signaling. This draft
extends RFC 7587 to define SDP signaling for multistream Opus in RTP sessions
and reuses the mapping semantics from RFC 7845.

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

Deployed systems (e.g., libwebrtc‑based) interoperate using a non‑standard
SDP encoding name “multiopus” with fmtp parameters such as num_streams,
coupled_streams, and channel_mapping.
Standardizing these semantics improves interoperability and removes the need
for application‑level SDP text modifications.

# SDP Signaling for Opus Multistream

## Encoding Name and RTPMAP

The SDP rtpmap attribute for Opus multistream uses the encoding name multiopus:
a=rtpmap: multiopus/48000/[channels]

[channels]: the total number of output channels (e.g., 6 for 5.1, 8 for 7.1).


## Format Parameters (FMTP)

### The following fmtp parameters are defined:

num_streams (integer): number of Opus elementary streams.
coupled_streams (integer): number of coupled (stereo) streams.
channel_mapping (comma‑separated integers): per‑channel mapping from decoded streams to output channels, ordered consistently with Opus multistream semantics.
mapping_family (integer, OPTIONAL): identifies a well‑known mapping family for common layouts as defined by RFC 7845; if present, it qualifies the interpretation of channel_mapping. 
When omitted, channel_mapping is authoritative.
Other Opus parameters from RFC 7587 (e.g., maxaveragebitrate, minptime, useinbandfec) remain applicable.


# Offer/Answer Procedures

An offerer willing to negotiate multichannel Opus MAY include one or more
payload types using multiopus with appropriate fmtp, and SHOULD include a stereo alternative using opus/48000/2
(RFC 7587) for backward compatibility.

An answerer that supports the offered multiopus configuration MUST select the corresponding payload type and include the selected
multistream parameters in the answer. If unsupported, the answerer MAY select a stereo opus payload or reject the m‑section per RFC 3264.
Down‑conversion to stereo SHOULD NOT occur silently when the answerer supports the offered configuration.


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


# Contributors
{:numbered="false"}

- Sun Shin
