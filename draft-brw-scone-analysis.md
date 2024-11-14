---
title: "SCONE Solution Analysis"
abbrev: "Solution Analysis"
category: std

docname: draft-brw-scone-analysis-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: wit
workgroup: scone
keyword:
 - collaborative networking
 - adaptive application

author:
 -
    fullname: Mohamed Boucadair
    organization: Orange
    email: mohamed.boucadair@orange.com
 -
    fullname: Dan Wing
    organization: Cloud Software Group Holdings, Inc.
    abbrev: Cloud Software Group
    country: United States of America
    email: danwing@gmail.com
 -
    fullname: Tirumaleswar Reddy
    organization: Nokia
    country: India
    email: kondtir@gmail.com
 -
    fullname: Sridharan Rajagopalan
    organization: Cloud Software Group Holdings, Inc.
    abbrev: Cloud Software Group
    country: United States of America
    email: "sridharan.girish@gmail.com"
 -
    ins: L. Contreras
    name: Luis M. Contreras
    org: Telefonica
    country: Spain
    email: luismiguel.contrerasmurillo@telefonica.com

normative:

informative:

--- abstract

TBC

--- middle

# Introduction

tbc

# Summary

| Criteria                                      |MASQUE| NRLP |SCONE |TRAIN |
|----------------------------------------------:|:----:|:----:|:----:|:----:|
| Guard against random advice injection         |Y/N   |  Y   |Y/N   |  Y/N |
| Mobility (guard against changing 5-tuple)     |Y/N   |  Y   |Y/N   |  Y/N |
| Require guards against app abuse              |Y/N   |  Y   |Y/N   |  Y/N |
| Fate sharing                                  |Y/N   |  Y   |Y/N   |  Y/N |
| Atomic configuration                          |Y/N   |  Y   |Y/N   |  Y/N |
| Updatability/Proactive signalling             |Y/N   |  Y   |Y/N   |  Y/N |
| Integration with network management tools     |Y/N   |  Y   |Y/N   |  Y/N |
| Applicable to any transport protocol          |Y/N   |  Y   |Y/N   |  Y/N |
| Applicable to QUIC                            |Y/N   |  Y   |Y/N   |  Y/N |
| Applicable to any application                 |Y/N   |  Y   |Y/N   |  Y/N |
| Require an OS API                             |Y/N   |  Y   |Y/N   |  Y/N |
| Requires PvD                                  |Y/N   | Y(*) |Y/N   |  Y/N |
| Support cascaded environments                 |Y/N   |  Y   |Y/N   |  Y/N |
| Path coupled signaling                        |Y/N   |  Y   |Y/N   |  Y/N |
| Path decoupled signaling                      |Y/N   |  Y   |Y/N   |  Y/N |
| Traffic direction (h2n, n2h, both)            |Y/N   |  Y   |Y/N   |  Y/N |
| Per-host policies                             |Y/N   |  Y   |Y/N   |  Y/N |
| Per-subscriber policies                       |Y/N   |  Y   |Y/N   |  Y/N |
| Require data plane upgrade/change             |Y/N   |  N   |Y/N   |  Y/N |
| Require transport payload inspection (network)|Y/N   |  N   |Y/N   |  Y/N |
| Require transport payload inspection (host)   |Y/N   |  N   |Y/N   |  Y/N |
| Require steering policies on the host         |Y/N   |  N   |Y/N   |  Y/N |
| Depend on the server to consume the signal    |Y/N   |  N   |Y/N   |  Y/N |
| Impact the connection setup delay             |Y/N   |  N   |Y/N   |  Y/N |
| Require the identity of the target server     |Y/N   |  N   |Y/N   |  Y/N |
| Require MTU tweaking                          |Y/N   |  N   |Y/N   |  Y/N |
| Incur multi-layer encryption                  |Y/N   |  N   |Y/N   |  Y/N |
| Incur nested congestion control               |Y/N   |  N   |Y/N   |  Y/N |
| Incur multiple round-trips                    |Y/N   |  N   |Y/N   |  Y/N |
| Overhead of unauthenticated re-encryption     |Y/N   |  N   |Y/N   |  Y/N |
| Forwarding peformance impact                  |Y/N   |  N   |Y/N   |  Y/N |
| IP address sharing issues                     |Y/N   |  N   |Y/N   |  Y/N |
| Penalizing the proxy                          |Y/N   |  N   |Y/N   |  Y/N |

# Security Considerations

TBC.

# IANA Considerations

This document does not make any IANA request.

--- back
