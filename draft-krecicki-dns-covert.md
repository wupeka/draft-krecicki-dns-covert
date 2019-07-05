%%%
Title = "Domain Name System (DNS) Resource Record types for transferring covert information from primary to secondaries"
abbrev = "dns-covert"
docname = "@DOCNAME@"
category = "std"
ipr = "trust200902"
area = "Internet"
workgroup = "DNSOP Working Group"
updates = [6195]
date = 2019-07-05T00:00:00Z

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
that are integral part of the zone but cannot be normally accessed via a
QUERY operation.
An example usages could be a zone comment that's transferrable with the zone,
expiry time for dynamically updated records, or Zone Signing Key for inline
signing - this document however does not define any specific Covert RR-TYPEs.

{mainmatter}

# Introduction

Domain Name System (DNS) has no mechanism for sending control information
in-band with the zone from Primary to Secondary servers.
This document specifies a range of Resource Record TYPEs that can be used for
Covert Resource Records - ones that are transferred with zone using Zone
Transfer but are not accessible by a normal QUERY operation.
It also specifies a method for signalling Primary server that the Secondary
understands Covert semantics and that it won't disclose contents of Covert RRs
to querying clients.

## Definitions

The key words "**MUST**", "**MUST NOT**", "**REQUIRED**",
"**SHALL**", "**SHALL NOT**", "**SHOULD**", "**SHOULD NOT**",
"**RECOMMENDED**", "**NOT RECOMMENDED**", "**MAY**", and
"**OPTIONAL**" in this document are to be interpreted as described in
BCP 14 [@!RFC2119] [@!RFC8174] when, and only when, they appear in all
capitals, as shown here.

# Handling of Covert Resource Records

Covert Resource Records require special handling for both queries and zone
transfers. This document does not define any specific Covert Resource
Record, they might require additional handling on the server side but it's
out of scope of this document.

## The COVERT-OK option
A client querying or a secondary transferring a zone from a primary server
must explicitly signal its understanding of the COVERT RR-TYPEs. The mechanism
for this is an EDNS option, with OPTION-CODE [TBD].
OPTION-LENGTH MUST be zero and OPTION-DATA MUST be empty.

## Protection of Zone Transfers

If a Secondary Server requesting a zone transfer does not understand the Covert
semantics it will serve the Covert Resource Records to its clients - therefore
a protection mechanism must be put in place.
If a server requesting zone transfer understands Covert semantics, it
MUST send COVERT-OK  option in the transfer request. If a Primary Server providing
zone transfer receives such request it then knows it can transfer the zone
securely.
If the primary server receives a zone transfer request without the COVERT-OK
option it MUST NOT transfer the zone with Covert RRs. The default
behaviour MUST be to refuse the transfer altogether, but an implementation MAY
have a configuration option to allow transfer of the zone with Covert RRs
stripped when transferring to a non-compliant secondary.

## Authoritative server behaviour

The Covert Resource Records might contain sensitive data and therefore they
MUST NOT be served to regular clients. The server MAY provide a mechanism
allowing clients to query for Resource Records in Covert range, but it MUST be
protected by a mechanism disallowing access from general public (e.g. an
ACL or TSIG authentication) and the general access MUST NOT be enabled by
default. The server MUST verify that the query has the COVERT-OK option
and not return COVERT records otherwise.
With this exception in place, a server queried for a Covert RR MUST return
an answer as if the particular leaf for which client asks does not exists -
either NODATA if there are some other Resource Records or it's
an empty non-terminal, or NXDOMAIN otherwise.

## Recursive server behaviour

Recursive server MUST NOT send COVERT-OK option when iterating. If a COVERT
record is received when iterating it MUST NOT be cached and it MUST NOT be
returned to the client. If a recursive server receives a request for a
COVERT record it may either iterate to verify whether the answer should be
an NXDOMAIN or NODATA, or simply return a NODATA response immediately.

## Interaction with DNSSEC

Since Covert Resource Records are not available for regular querying and are
used only internally their existence in zone should not, in any way, change
the behaviour of zone. Therefore, when signing the zone, Covert Resource
Records MUST be ommited entirely and treated as they do not exist.

## Interaction with ZONEMD

TBD

# Update to RRTYPE Allocation Template

RRTYPE Allocation Template from [@!RFC6895] is updated to contain a checkbox
for Covert-RR:

~~~ ascii-art

B.2 Kind of RR:  [ ] Data RR  [ ] Meta-RR  [ ] Covert-RR

~~~

# Security considerations

Since Zone Transfers are unencrypted the contents of Covert RRs might still
be snooped by an on-path attacker. Protection against this kind of attack is
outside of scope of this document, it can be achieved using e.g. a secure
tunnel, private network or using XFR over TLS transport.

# IANA Considerations

IANA is requested to reserve range 61440-61695 (0xF000-0xF0FF) in Resource
Record TYPEs registry for Covert types. The procedure for registering RR
types from [@!RFC6895] should be used.

IANA is requested to assign an EDNS option code to COVERT-OK option.

# Acknowledgments

Thanks to Evan Hunt and Dan Mahoney for suggestions and clarifications based
on their NOTE RR draft.

{backmatter}

