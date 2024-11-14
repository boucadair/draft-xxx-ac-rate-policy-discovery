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

This document provides an analysis of various SCONE solutions.

--- middle

# Introduction

The document provides an analysis of proposed SCONE solutions. The currently analyzed solutions (listed in alphabetic order)
are as follows:

MASQUE:
: "MASQUE extension for signaling throughput advice" {{!I-D.ihlar-scone-masque-mediabitrate}}
: See {{sec-masque}}.

NRLP:
: "Discovery of Network Rate-Limit Policies (NRLPs)" {{!I-D.brw-scone-rate-policy-discovery}}
: See {{sec-nrlp}}.

SCONE:
: "A new QUIC version for network property communication" {{!I-D.joras-scone-quic-protocol}}
: See {{sec-scone}}.

TRAIN:
: "Transparent Rate Adaptation Indications for Networks (TRAIN) Protocol" {{!I-D.thomson-scone-train-protocol}}
: See {{sec-train}}.

# Criteria Classification {#sec-class}

The following criteria are used to classify the various criteria:

* Security: Indicates whether this impact security/privacy. Some of the criteria that are classified a security-related may also have implications on the efficiency of sharing an advice (as that is likely to be ignored).
* Deployability: Captures a criteria that is important for unlocking the deployment of a solution at both network and host sides.
* Performance: May impact the performance of the network device that enables the solution and/or the performance of the flow.
* Service Interference: captures implications on other services (e.g., side effects). For example, tweaking MTU may have an implication on all the flows that share the same network attachment, not only those that consumes an advice. Likewise, requiring address sharing has a plenty of issues that are discussed in {{?RFC6269}}.
* Functional: Characterizes the functional capabilities offered by activating a solution.

A criterion may belong to one or more categories.

| Criteria                                      |Security| Deployability |Performance|Service Interference|Functional|
|----------------------------------------------:|:------:|:-------------:|:---------:|:------------------:|:--------:|
| Guard against random advice injection         |X       |               |           |                    |          |
| Mobility (guard against changing 5-tuple)     |X       |               |           |                    |   X      |
| Require guards against app abuse              |X       |               |           |                    |   X      |
| Fate sharing                                  |        |  X            |           |                    |          |
| Atomic configuration                          |        |  X            |           |                    |          |
| Updatability/Proactive signalling             |        |               |           |                    |    X     |
| Integration with network management tools     |        |  X            |           |                    |          |
| Applicable to any transport protocol          |        |               |           |                    |    X     |
| Applicable to QUIC                            |        |               |           |                    |    X     |
| Applicable to any application                 |        |               |           |                    |    X     |
| Require an OS API                             |        |  X            |           |                    |          |
| Requires PvD                                  |        |  X            |           |                    |          |
| Support cascaded environments                 |        |               |           |                    |    X     |
| Path coupled signaling                        |        |               |           |                    |    X     |
| Path decoupled signaling                      |        |               |           |                    |    X     |
| Traffic direction (h2n, n2h, both)            |        |               |           |                    |    X     |
| Per-host policies                             |        |               |           |                    |    X     |
| Per-subscriber policies                       |        |               |           |                    |    X     |
| Extendable                                    |        |               |           |                    |    X     |
| Require data plane upgrade/change             |        |  X            |           |                    |          |
| Require transport payload inspection (network)|        |  X            |           |                    |          |
| Require transport payload inspection (host)   |        |  X            |           |                    |          |
| Require steering policies on the host         |        |  X            |           |                    |          |
| Depend on the server to consume the signal    |        |  X            |           |                    |          |
| Impact the connection setup delay             |        |               |           |                    |    X     |
| Require the identity of the target server     |   X    |               |           |                    |    X     |
| Require MTU tweaking                          |        |  X            |           |        X           |          |
| Incur multi-layer encryption                  |        |  X            |     X     |                    |          |
| Incur nested congestion control               |        |  X            |     X     |                    |          |
| Incur multiple round-trips                    |        |  X            |     X     |                    |          |
| Overhead of unauthenticated re-encryption     |        |  X            |     X     |                    |          |
| Forwarding peformance impact                  |        |  X            |     X     |        X           |          |
| IP address sharing issues                     |        |  X            |           |        X           |          |
| Penalizing the proxy                          |        |  X            |           |        X           |          |

# Detailed Analysis

## Summary  {#sec-analysis}

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
| Extendable                            |Y/N   |  Y   |Y/N   |  Y/N |
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

## MASQUE (to be completed by the authors of MASQUE) {#sec-masque}

### Key Idea
### Discussion

## NRLP {#sec-nrlp}

### Key Idea

NRLP leverages existing discovery mechanisms (DHCP, RA, PvD) for networks to advertise throughout advices.
The same generic blob is used independent of the signaling mechanism.

### Discussion

Only network elements that are entitled to send DHCP/RA/DHCP configuration are allowed to share an advice. As such, NRLP has built-in:

* Guard against random advice injection

Taking into account that NRLP advices are bound to a traffic category, NLRP relies upon the OS to enforce the received policies
for applications falling under a traffic category (or all traffic). In doing so, NRLP adheres to the following:

* Mobility (guard against changing 5-tuple)
* Require guards against app abuse

NRLP meets the following criteria:

* Fate sharing: RA/DHCP are needed anyway so that connectivity is provided over a network attachment. NRLP ensures that throughput advices shares the fare of the other network configuration on the host.
* Atomic configuration: Only one packet (e.g., RA) is required to share the advice. Also, only a specific portion of the configuration can be provided.
* Updatability/Proactive signaling: It is possible to change the policy at any time and notify hosts (e.g., by sending a new RA).

Given that NRLP advices are shared during the establishment of a network attachment and then as part of the maintenance of the attachment, NRLP is therefore:

* Applicable to any transport protocol: This allows specifically to ensure a feature parity for applications that fallback to another transport protocol (e.g., QUIC to TCP).
* Applicable to QUIC
* Applicable to any application

To that aim aim, NRLP:

* Requires an OS API to expose the signal to applications, and ensure application fairness.

NRLP leverages existing mechanisms for the provisioning of network attachments, including supply of the various policies ({{?I-D.ietf-opsawg-ntw-attachment-circuit}}). Also, NRLP leverages AAA mechanisms (e.g., {{?RFC9445}}). Therefore, NRLP eases:

* Integration with network management tools

One of NRLP flavors:

* Requires PvD discovery. This is not required for DHCP/RA.

NRLP does not restrict the deployment options as providers can deploy distributed or centralized DHCP servers, use relays, enable NRLP RA in access routers, etc. Similar to other network configuration purposes, NRLP has the following capabilities:

* Support cascaded environments
* Path coupled signaling
* Path decoupled signaling

The same generic blob is used in NRLP independent of the signaling mechanism. The blob is designed with the following key characteristics:

* Traffic direction (h2n, n2h, both): policies for one or both directions can be supplied.
* Per-host policies: An explicit indication in inserted in the advice to tag per-host policies.
* Per-subscriber policies: An explicit indication in inserted in the advice to tag per-subscriber policies. This covers deployment scenarios such as tethering or CPE-based service offerings.
* Provide provisions for extensions: NLRP includes provisions for future attributes that are tracked in IANA registries.

Given that NRLP leverages existing control plane mechanisms, NRLP does not:

* Require data plane upgrade/change
* Require transport payload inspection (network)
* Require transport payload inspection (host)

Also, given that NRLP signals are exchanged before connection establishment, NRLP does not:

* Depend on the server to consume the signal: NRLP advices are immediately consumable by applications and do not require involving a remote server.
* Require the identity of the target server to receive or consume the advices.

Moreover, NRLP does require any encapsulation or proxy function at the network. As such, NRLP does not:

* Require steering policies on the host to decide which flows are eligible to the proxy service.
* Impact the connection setup delay: NRLP signals are available on bootstrap of a host (and prior to any connection establishment).
* Require MTU tweaking
* Incur multi-layer encryption
* Incur nested congestion control
* Incur multiple round-trips: The signal is immediately available in one packet (RA NRLP, typically).
* Overhead of unauthenticated re-encryption
* Forwarding performance impact
* IP address sharing issues: NRLP does not require changing the source IP address used by a host.
* Penalize any network node (a proxy, typically)


## SCONE  (to be completed by the authors of SCONE) {#sec-scone}

### Key Idea
### Discussion

## TRAIN  (to be completed by the authors of TRAIN) {#sec-train}

### Key Idea
### Discussion

# Security Considerations

TBC.

# IANA Considerations

This document does not make any IANA request.

--- back
