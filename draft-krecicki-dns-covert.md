%%%
Title = "Domain Name System (DNS) Resource Record types for transferring covert information from primary to secondaries"
abbrev = "dns-covert"
docname = "@DOCNAME@"
category = "std"
ipr = "trust200902"
area = "Internet"
workgroup = "DNSOP Working Group"
updates = [6195]
date = 2019-03-25T00:00:00Z

[seriesInfo]
name = "Internet-Draft"
value = "@DOCNAME@"
stream = "IETF"
status = "standard"

[[author]]
initials = "W."
surname = "Krecicki"
fullname = "Witold Krecicki"
organization = "Internet Systems Consortium"
[author.address]
 email = "wpk@isc.org"
[author.address.postal]
 country = "PL"
%%%


.# Abstract

Domain Name System (DNS) Resource Record TYPEs IANA registry reserves range
128-255 for Q-TYPEs and Meta-TYPEs [@!RFC6895] - Resource Records that can
only be queried for or contain transient data associated with a particular
DNS message.
This document reserves a range of RR TYPE numbers for Covert-TYPEs - types
that are integral part of the zone but cannot be regularly queried for.
It also defines a NOTE Covert Resource Record - an equivalent of zone
comment as defined in [@!RFC1035] that is transferred with the zone.

{mainmatter}

# Introduction

Domain Name System (DNS) has no mechanism for sending control information
in-band with the zone from Primary to Secondary servers.
This document specifies a range of Resource Record TYPEs that is to be used for
Covert Resource Records - ones that are transferred with zone using Zone
Transfer but are not accessible by regular querying clients.
It also specifies a method for signalling Primary server that
the Secondary understands Covert semantics and that it won't disclose
contents of Covert RRs to querying clients.

A new NOTE Resource Records is defined in this document, other usages
could be e.g. transferring DNSSEC Zone Signing Key for online signing or
timeouts for dynamic records.


## Definitions

The key words "**MUST**", "**MUST NOT**", "**REQUIRED**", 
"**SHALL**", "**SHALL NOT**", "**SHOULD**", "**SHOULD NOT**",
"**RECOMMENDED**", "**NOT RECOMMENDED**", "**MAY**", and
"**OPTIONAL**" in this document are to be interpreted as described in
BCP 14 [@!RFC2119] [@!RFC8174] when, and only when, they appear in all
capitals, as shown here.

# Covert Resource Records

The usage of Covert Resource Records is not clearly defined - they can be
used for any kind of unidirectional communication between Primary and
Secondary server.

## Handling of Covert Resource Records

The Covert Resource Records might contain sensitive data and therefore they
MUST NOT be served to regular clients. The server MAY provide a mechanism
allowing clients to query for Resource Records in Covert range, but it MUST be
protected by a mechanism disallowing access from general public (e.g. an
ACL or TSIG authentication) and the general access MUST NOT be enabled by
default. 
With this exception in place, a server queried for a Covert RR MUST return
an answer as if the particular node for which client asks does not exists
 - either NODATA if there are some other Resource Records or it's
an empty non-terminal, or NXDOMAIN otherwise. (TODO: should it be signed?)

## Protection of Zone Transfers

If a Secondary Server requesting a zone transfer does not understand the Covert
semantics it might serve the RR to its clients - therefore a protection
mechanism must be put in place.
If a server requesting zone transfer understands Covert semantics, it
MUST set COVERT-OK bit in EDNS header flags. If a server providing zone
transfer receives such request it then knows it can transfer the zone
securely.
However, if the server receives a zone transfer request without the
COVERT-OK flag set it MUST NOT transfer the zone with Covert RRs - it might
either provide the zone without them or refuse the transfer completely, the
behaviour depends on the specific Type of Resource Record, and it MUST be
specified by the document defining the Type.

# Definition of a NOTE Resource Record

The NOTE RR is defined for all classess, with mnemonic NOTE and type code
61440. RDATA and presentation formats are identical to those of
TXT RR defined in [@!RFC1034], e.g.

~~~ ascii-art

       $ORIGIN example.com.
       joesbox   7200  IN  A       198.51.100.42
                 7200  IN  AAAA    2001:DB8:3F:B019::17
                 0     IN  NOTE    "Desktop system for Joe Smith, x7889"

~~~

## Behaviour of NOTE RR for Zone Transfer clients not supporting COVERT-OK

If a Zone Transfer is requested by a server not supporting Covert records
the queried server SHOULD send the zone ommiting those records.

# Update to RRTYPE Allocation Template

RRTYPE Allocation Template from [@!RFC6895] is updated to contain a checkbox
for Covert-RR:

~~~ ascii-art

B.2 Kind of RR:  [ ] Data RR  [ ] Meta-RR  [ ] Covert-RR

~~~

# IANA Considerations

IANA is requested to reserve range 61440-61695 (0xF000-0xF0FF) in Resource
Record TYPEs registry for Covert types. The procedure for registering RR
types from [@!RFC6895] should be used.

IANA is requested to assign Bit 1 of EDNS Header to COVERT-OK bit.

IANA is requested to assign a Resource Record type 61440 for NOTE RR.

# Security considerations

Since Zone Transfers are unencrypted the contents of Covert RRs might still
be snooped by an on-path attacker. Protection against this kind of attack is
outside of scope of this document, it can be achieved using e.g. a secure
tunnel, private network or a form of DNS over TLS transport. 
 
{backmatter}

