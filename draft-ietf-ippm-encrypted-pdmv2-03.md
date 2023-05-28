---
title: "IPv6 Performance and Diagnostic Metrics Version 2 (PDMv2) Destination Option"
abbrev: "draft-ietf-ippm-encrypted-pdmv2-03"
category: std

docname: draft-ietf-ippm-encrypted-pdmv2-03
submissiontype: IETF  # also: "independent", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Transport"
workgroup: "IP Performance Measurement"
keyword:
 - Extension Headers
 - IPv6
 - PDMv2
 - Performance
 - Diagnostic
venue:
  group: "IP Performance Measurement"
  type: "Working Group"
  mail: "ippm@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/ippm/"
  github: "ameyand/PDMv2"
  latest: "https://ameyand.github.io/PDMv2/draft-elkins-ippm-encrypted-pdmv2.html"

author:
 -
    fullname: Nalini Elkins
    organization: Inside Products, Inc.
    email: "nalini.elkins@insidethestack.com"

 -
    fullname: Michael Ackermann
    organization: BCBS Michigan
    email: "mackermann@bcbsm.com"

 -
    fullname: Ameya Deshpande
    organization: NITK Surathkal/Google
    email: "ameyanrd@gmail.com"

 -
    fullname: Tommaso Pecorella
    organization: University of Florence
    email: "tommaso.pecorella@unifi.it"

 -
    fullname: Adnan Rashid
    organization: Politecnico di Bari
    email: "adnan.rashid@poliba.it"

normative:
 RFC2780:
 RFC8250:
 RFC8200:

informative:
 RFC9180:
 RFC1421:

--- abstract

RFC8250 describes an optional Destination Option (DO) header embedded
in each packet to provide sequence numbers and timing information as
a basis for measurements.  As this data is sent in clear-text, this
may create an opportunity for malicious actors to get information for
subsequent attacks.  This document defines PDMv2 which has encryption to
secure this data.  Additional performance metrics which may be of use
are also defined.

--- middle

# Introduction

## Current Performance and Diagnostic Metrics (PDM)

The current PDM is an IPv6 Destination Options header which provides
information based on the metrics like Round-trip delay and Server
delay.  This information helps to measure the Quality of Service
(QoS) and to assist in diagnostics.  However, there are potential
risks involved transmitting PDM data during a diagnostics session.

PDM metrics can help an attacker understand about the type of machine
and its processing capabilities.  Inferring from the PDM data, the
attack can launch a timing attack.  For example, if a cryptographic
protocol is used, a timing attack may be launched against the keying
material to obtain the secret.

Along with this, PDM does not provide integrity.  It is possible for
a Man-In-The-Middle (MITM) node to modify PDM headers leading to
incorrect conclusions.  For example, during the debugging process
using PDM header, it can mislead the person showing there are no
unusual server delays.


## PDMv2 Introduction

PDMv2 adds confidentiality, integrity and authentication to PDM.

# Conventions used in this document

{::boilerplate bcp14-tagged}

# Terminology

-  Symmetric Key (K): A uniformly random bitstring as an input to the
   encryption algorithm, known only to Secondary Clients and
   Secondary Servers, to establish a secure communication.

-  Public and Private Keys: A pair of keys that is used in asymmetric
   cryptography.  If one is used for encryption, the other is used
   for decryption.  Private Keys are kept hidden by the source of the
   key pair generator, but Public Key is known to everyone.  pkX
   (Public Key) and skX (Private Key).  Where X can be, any client or
   any server.

-  Pre-shared Key (PSK): A symmetric key.  Uniformly random
   bitstring, shared between any client or any server or a key shared
   between an entity that forms client-server relationship.  This
   could happen through an out-of band mechanism: e.g., a physical
   meeting or use of another protocol.

-  Session Key: A temporary key which acts as a symmetric key for the
   whole session.


# Protocol Flow

{:req1: counter="bar" style="format Step %d:"}

{: req1}

The main flow is PDMv2 data flow between Client and Server.

After-the-fact (or real-time) data analysis of PDM flow may occur by
network diagnosticians or network devices.  The definition of how
this is done is out of scope for this document.

The Client and Server exchange a set of data to ensure the respective
identities.

They use HPKE KEM to negotiate a "SharedSecret".

Each Client and Server derive a "SessionTemporaryKey" by using HPKE
KDF, using the following inputs:

-  The "SharedSecret".

-  The 5-tuple (SrcIP, SrcPort, DstIP, DstPort, Protocol) of the
   communication.

-  A Key Rotation Index (Kri).

The Kri SHOULD be initialized to zero.

The Server and Client initialize (separately) a pseudo-random non-
repeating sequence between 1 and 2^15-1.  How to generate this
sequence is beyond the scope of this document, and does not affect
the rest of the specification.  When the sequence is used fully, or
earlier if appropriate, the sender signals the other party that a key
change is necessary.  This is achieved by flipping the "F bit" and
resetting the PRSEQ.  The receiver increments the Kri of the sender,
and derives another SessionTemporaryKey to be used for decryption.

It shall be stressed that the two SessionTemporaryKeys used in the
communication are never the same, as the 5-tuple is reversed for the
Server and Client.  Moreover, the time evolution of the respective
Kri can be different.  As a consequence, each entity must maintain a
table with (at least) the following informations:


-  Flow 5-tuple, Own Kri, Other Kri

An implementation might optimize this further by caching the
OwnSessionTemporaryKey (used in Encryption) and
OtherSessionTemporaryKey (used in Decryption).

# Security Goals

As discussed in the introduction, PDM data can represent a serious
data leakage in presence of a malicious actor.

In particular, the sequence numbers included in the PDM header allows
correlating the traffic flows, and the timing data can highlight the
operational limits of a server to a malicious actor.  Moreover,
forging PDM headers can lead to unnecessary, unwanted, or dangerous
operational choices, e.g., to restore an apparently degraded Quality
of Service (QoS).

Due to this, it is important that the confidentiality and integrity
of the PDM headers is maintained.  PDM headers can be encrypted and
authenticated using the methods discussed in Section 5.4, thus
ensuring confidentiality and integrity.  However, if PDM is used in a
scenario where the integrity and confidentiality is already ensured
by other means, they can be transmitted without encryption or
authentication.  This includes, but is not limited to, the following
cases:

{:req2: style="format %c)"}

{: req2}
- PDM is used over an already encrypted medium (For example VPN
  tunnels).
- PDM is used in a link-local scenario.
- PDM is used in a corporate network where there are security
  measures strong enough to consider the presence of a malicious
  actor a negligible risk.

## Security Goals for Confidentiality

PDM data must be kept confidential between the intended parties,
which includes (but is not limited to) the two entities exchanging
PDM data, and any legitimate party with the proper rights to access
such data.


## Security Goals for Integrity

PDM data must not be forged or modified by a malicious entity.  In
other terms, a malicious entity must not be able to generate a valid
PDM header impersonating an endpoint, and must not be able to modify
a valid PDM header.

## Security Goals for Authentication

An unauthorized party must not be able to send PDM data and must not
be able to authorize another entity to do so.  The protocol to define
this will be defined in a subsequent document.  Alternatively, if
authentication is done via any of the following, this requirement may
be seen to be met.

{:req3: style="format %c)"}

{: req3}
- PDM is used over an already authenticated medium (For example,
  TLS session).
- PDM is used in a link-local scenario.
- PDM is used in a corporate network where security measures are
  strong enough to consider the presence of a malicious actor a
  negligible risk.

## Cryptographic Algorithm

Symmetric key cryptography has performance benefits over asymmetric
cryptography; asymmetric cryptography is better for key management.
Encryption schemes that unite both have been specified in [RFC1421],
and have been participating practically since the early days of
public-key cryptography.  The basic mechanism is to encrypt the
symmetric key with the public key by joining both yields.  Hybrid
public-key encryption schemes (HPKE) [RFC9180] used a different
approach that generates the symmetric key and its encapsulation with
the public key of the receiver.

Our choice is to use the HPKE framework that incorporates key
encapsulation mechanism (KEM), key derivation function (KDF) and
authenticated encryption with associated data (AEAD).  These multiple
schemes are more robust and significantly efficient than the
traditional schemes and thus lead to our choice of this framework.

# PDMv2 Destination Options


## Destinations Option Header

The IPv6 Destination Options extension header [RFC8200] is used to
carry optional information that needs to be examined only by a
packet's destination node(s).  The Destination Options header is
identified by a Next Header value of 60 in the immediately preceding
header and is defined in RFC 8200 [RFC8200].  The IPv6 PDMv2
destination option is implemented as an IPv6 Option carried in the
Destination Options header.

## Metrics information in PDMv2

The IPv6 PDMv2 destination option contains the following base fields:

{:req4: style="empty"}

{: req4}
- SCALEDTLR: Scale for Delta Time Last Received
- SCALEDTLS: Scale for Delta Time Last Sent
- GLOBALPTR: Global Pointer
- PSNTP: Packet Sequence Number This Packet
- PSNLR: Packet Sequence Number Last Received
- DELTATLR: Delta Time Last Received
- DELTATLS: Delta Time Last Sent

PDMv2 adds a new metric to the existing PDM [RFC8250] called the
Global Pointer.  The existing PDM fields are identified with respect
to the identifying information called a "5-tuple".

The 5-tuple consists of:

{:req5: style="empty"}

{: req5}
- SADDR: IP address of the sender
- SPORT: Port for the sender
- DADDR: IP address of the destination
- DPORT: Port for the destination
- PROTC: Upper-layer protocol (TCP, UDP, ICMP, etc.)

Unlike PDM fields, Global Pointer (GLOBALPTR) field in PDMv2 is
defined for the SADDR type.  Following are the SADDR address types
considered:

{:req6: style="format %c)"}

{: req6}
- Link-Local
- Global Unicast

The Global Pointer is treated as a common entity over all the
5-tuples with the same SADDR type.  It is initialised to the value 1
and increments for every packet sent.  Global Pointer provides a
measure of the amount of IPv6 traffic sent by the PDMv2 node.


When the SADDR type is Link-Local, the PDMv2 node sends Global
Pointer defined for Link-Local addresses, and when the SADDR type is
Global Unicast, it sends the one defined for Global Unicast
addresses.

### PDMv2 Layout

PDMv2 has two different header formats corresponding to whether the
metric contents are encrypted or unencrypted.  The difference between
the two types of headers is determined from the Options Length value.

Following is the representation of the unencrypted PDMv2 header:

~~~
   0                   1                   2                   3
   0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |  Option Type  | Option Length | Vrsn  | Epoch |  Reserved     |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |       PSN This Packet         |       Reserved                |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |                         Global Pointer                        |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |    ScaleDTLR   |   ScaleDTLS  |    PSN Last Received          |
  |-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |   Delta Time Last Received    |     Delta Time Last Sent      |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~

Following is the representation of the encrypted PDMv2 header:

~~~
   0                   1                   2                   3
   0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |  Option Type  | Option Length | Vrsn  | Epoch |   Reserved    |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |      PSN This Packet          | Reserved                      |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |                      Encrypted PDM Data                       :
  :                          (12 bytes)                           |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~

{:req7: style="empty"}

{: req7}
- Option Type

    0x0F

    8-bit unsigned integer.  The Option Type is adopted from RFC 8250 [RFC8250].


- Option Length

    0x12: Unencrypted PDM

    0x22: Encrypted PDM

    8-bit unsigned integer.  Length of the option, in octets, excluding the Option
    Type and Option Length fields.  The options length is used for differentiating
    PDM [RFC8250], unencrypted PDMv2 and encrypted PDMv2.


- Version Number

    0x2

    4-bit unsigned number.


- Epoch

    0x0

    4-bit unsigned number.

    This field is initialized at 0 and is incremented monotonically when
    PSNTP > 2^16.  When the field is incremented, this indicates that the
    sender has used a new _SessionTemporaryKey_ and the offline analysis
    should increment the Kri and derive the same new _SessionTemporaryKey_.


- Reserved Bits

    8-bits.

    Reserved bits for future use.  They are initialised to 0 for PDMv2.


- Packet Sequence Number This Packet (PSNTP)

    16-bit unsigned number.

    This field is initialized at 1 and is incremented monotonically 
    for each packet of the 5-tuple.  This field is sent in the clear 
    and is used in the encryption process.  Alternatively, if entropy
    is desired by the the field may be initialized at a random number.
    The level of entropy should be clearly specified to the user.


- Reserved Bits

    16-bits.

    Reserved bits for future use.  They are initialised to 0 for PDMv2.


 - Global Pointer

    32-bit unsigned number.

    Global Pointer is initialized to 1 for the different source
 	  address types and incremented monotonically for each packet
 	  with the corresponding source address type.

    This field stores the Global Pointer type corresponding to the
 	  SADDR type of the packet.


- Scale Delta Time Last Received (SCALEDTLR)

    8-bit unsigned number.

    This is the scaling value for the Delta Time Last Sent
 	  (DELTATLS) field.


- Scale Delta Time Last Sent (SCALEDTLS)

    8-bit unsigned number.

    This is the scaling value for the Delta Time Last Sent
          (DELTATLS) field.


- Packet Sequence Number Last Received (PSNLR)

    16-bit unsigned number.

    This field is the PSNTP of the last received packet on the
 	  5-tuple.


- Delta Time Last Received (DELTATLR)

    16-bit unsigned integer.

    The value is set according to the scale in SCALEDTLR.

    Delta Time Last Received =
    (send time packet n - receive time packet (n - 1))


- Delta Time Last Sent (DELTATLS)

    16-bit unsigned integer.

    The value is set according to the scale in SCALEDTLS.

    Delta Time Last Sent =
    (receive time packet n - send time packet (n - 1))


# Security Considerations

PDMv2 DOH can be used by an attacker to gather information about a
victim (passive attack) or to force the victim to modify its
operational parameters to comply with forged data (active attacks).

In order to mitigate these, it is important that the PDMv2 DOH is
subject to:
{:req8: style="format %d)"}

{: req8}
- Confidentiality and
- Integrity

with respect to an attacker.

In the following we will refer to two different "groups", that can or
cannot belong to the same operational and management domain:

{:req9: style="format %d)"}

{: req9}
- Servers - implementing services.

- Clients-devices willing to interact with the services offered by Servers.

We will assume, for the sake of generalization, that the Servers are
managed by an Organization (OrgA) implementing management procedures
over them, and the Clients by a different Organization (OrgB).

An attacker could be in the following positions:

{:req10: style="format %d)"}

{: req10}
- External to OrgA or OrgB.
- Inside OrgA (i.e., a Server), either because it is a legitimate-but-curious
  device, or as a consequence of an attack to a device.
- Inside OrgB (i.e., a Client), either because it is a legitimate-
  but-curious device, or as a consequence of an attack to a device

Furthermore, since PDMv2 DOH encryption could consume resources
(albeit limited), it is possible to foresee a call of DoS by resource
exhaustion.  Hence, it is relevant to consider a form of access
control to verify that the Server and Client belong to OrgA and OrgB
respectively.  This could be a _delegated trust_.

In other terms, a Client could just want to verify that the Server
belongs to OrgA, without actually verifying the identity of the
Server.

The Authentication and Authorization of Clients and Servers is thus
delegated to the respective Organizations.  In other terms, we do not
expect, or want, that a Client and a Server should be forced to
verify the respective identities (Authentication) or the permissions
to use PDMv2 (Authorization).

The simple knowledge of the secrets required by the flow is
considered sufficient to enable PDMv2.  On the opposite, an
unsuccessful decryption MUST result in dropping the PDMv2 DOH without
further processing or, if configured to do so, might lead to
throttling, filtering, and/or logging the activity of the other
entity (Client or Server).

The present document specifies a methodology to enable this delegated
trust, along with the Confidentiality and Integrity requirements, in
the PDMv2 DOH.

Server-Client

- Perform a HPKE KEM and obtain a PairMasterSecret (PMS).
- The PMS is stored securely in both Server and Client, and is NOT to be
  leaked.
- The PMS is valid only for the Server-Client pair.
- Server sends to the Client the Security Mode of Operation (SecMoP) to be
  used, see below.
- Each Server receives a PairServerSecret (PSS), derived using HPKE 
  KDF.
- Each Client receives a PairClientSecret (PCS), derived using HPKE KDF,
  and valid for the specific Client.

Since there are multiple use-cases, we define 4 modes of operations:

- **No Protection**: The Secrets are discarded (or not even created),
  and the flows do not use PDMv2.   

- **TrustedServers**: The Servers are trusted, and they know a secret
  derived by the PMS.

- **AsymmetricPoll**: A Client must acquire a secret from the Server.

- **Identity Based Cryptography (IBC)**: IBC (RFC5091) is used to
  generate a shared secret between the Server and the Client.

 
# Privacy Considerations

PDMv2 greatly improves the privacy aspects of PDM by providing
encryption.

# IANA Considerations

Option Type to be assigned by IANA [RFC2780].

# Contributors

The authors wish to thank NITK Surathkal for their support and
assistance in coding and review.  In particular Dr. Mohit Tahiliani
and Abhishek Kumar (now with Google).  Thanks also to Priyanka Sinha
for her comments.  Thanks to the India Internet Engineering Society
(iiesoc.in), in particular Dhruv Dhody, for providing the funding for
servers needed for protocol development.

--- back

# Change Log

Note to RFC Editor: if this document does not obsolete an existing
RFC, please remove this appendix before publication as an RFC.

# Open Issues

Note to RFC Editor: please remove this appendix before publication as
an RFC.