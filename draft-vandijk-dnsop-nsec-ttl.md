%%%
title = "NSEC(3) TTLs and NSEC Aggressive Use"
abbrev = "nsec-ttl"
docName = "draft-vandijk-dnsop-nsec-ttl-00+"
category = "std"
updates = [4034, 4035, 5155]

ipr = "trust200902"
area = "General"
workgroup = "dnsop"
keyword = ["Internet-Draft"]

[seriesInfo]
name = "Internet-Draft"
value = "draft-vandijk-dnsop-nsec-ttl-00+"
stream = "IETF"
status = "standard"

[pi]
toc = "yes"

[[author]]
initials = "P."
surname = "van Dijk"
fullname = "Peter van Dijk"
organization = "PowerDNS"
[author.address]
 email = "peter.van.dijk@powerdns.com"
[author.address.postal]
 city = "Den Haag"
 country = "Netherlands"


%%%

.# Abstract

Due to a combination of unfortunate wording in earlier documents, aggressive use of NSEC(3) records may deny names far beyond the intended lifetime of a denial.
This document changes the definition of the NSEC(3) TTL to correct that situation.
This document updates RFC 4034, RFC 4035, and RFC 5155.

{mainmatter}

# Introduction

[RFC editor: please remove this block before publication.

Earlier notes on this: 

* https://indico.dns-oarc.net/event/29/sessions/98/#20181013 
* https://lists.dns-oarc.net/pipermail/dns-operations/2018-April/thread.html#17420
* https://lists.dns-oarc.net/pipermail/dns-operations/2018-March/017416.html

This document lives [on GitHub](https://github.com/PowerDNS/draft-dnsop-nsec-ttl); proposed text and editorial changes are very much welcomed there, but any functional changes should always first be discussed on the IETF DNSOP WG mailing list.

]

[@!RFC2308] defines that the SOA TTL to be used in negative answers (NXDOMAIN, NoData NOERROR) is 

> the minimum of the MINIMUM field of the SOA record and the TTL of the SOA itself

Thus, if the TTL of the SOA in the zone is lower than the SOA MINIMUM value (the last number in a SOA record), the negative TTL for that zone is lower than the SOA MINIMUM value.

However, [@!RFC4034] section 4 has this unfortunate text:

> The NSEC RR SHOULD have the same TTL value as the SOA minimum TTL field. This is in the spirit of negative caching ([RFC2308]).

This text, while referring to RFC2308, can cause NSEC records to have much higher TTLs than the appropriate negative TTL for a zone.
[@!RFC5155] contains equivalent text.

[@!RFC8198] section 5.4 tries to correct this:

> Section 5 of [RFC2308] also states that a negative cache entry TTL is taken from the minimum of the SOA.MINIMUM field and SOA's TTL.  This can be less than the TTL of an NSEC or NSEC3 record, since their TTL is equal to the SOA.MINIMUM field (see [@!RFC4035], Section 2.3 and [RFC5155], Section 3).
>
> A resolver that supports aggressive use of NSEC and NSEC3 SHOULD reduce the TTL of NSEC and NSEC3 records to match the SOA.MINIMUM field in the authority section of a negative response, if SOA.MINIMUM is smaller.

But the NSEC(3) RRs should, per RFC4034, already be at the MINIMUM TTL, which means this advice would never actually change the TTL used for the NSEC(3) RRs.

As a concrete example, the `.com` SOA currently looks like this:

`com.			900	IN	SOA	a.gtld-servers.net. nstld.verisign-grs.com. 1606158464 1800 900 604800 86400`

The SOA record has a 900 second TTL, and a 86400 MINIMUM TTL.
Negative responses from this zone have a 900 second TTL, but the NSEC3 records in those negative responses have a 86400 TTL.
If a resolver were to use those NSEC3s aggressively, they would be considered valid for a day, instead of the intended 15 minutes.
(Note that, because .com uses opt-out NSEC3, such aggressive use would not in fact apply to this zone - it is merely used as a very visible example here.)

# Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 [@!RFC2119] [@RFC8174] when, and only when, they appear in all capitals, as shown here.

# NSEC(3) TTL changes

## Updates to RFC4034

Where [@!RFC4034] says:

> The NSEC RR SHOULD have the same TTL value as the SOA minimum TTL field.  This is in the spirit of negative caching ([RFC2308]).

This is updated to say:

> The NSEC RR MUST have the same TTL value as the minimum of the MINIMUM field of the SOA record and the TTL of the SOA itself.  This matches the definition of the TTL for negative responses in [@!RFC2308].

## Updates to RFC4035

Where [@!RFC4035] says:

> The TTL value for any NSEC RR SHOULD be the same as the minimum TTL value field in the zone SOA RR.

This is updated to say:

> The TTL value for any NSEC RR MUST be the same TTL value as the minimum of the MINIMUM field of the SOA record and the TTL of the SOA itself.  This matches the definition of the TTL for negative responses in [@!RFC2308].

## Updates to RFC5155

Where [@!RFC5155] says:

> The NSEC3 RR SHOULD have the same TTL value as the SOA minimum TTL field.  This is in the spirit of negative caching [RFC2308].

This is updated to say:

> The NSEC3 RR MUST have the same TTL value as the minimum of the MINIMUM field of the SOA record and the TTL of the SOA itself.  This matches the definition of the TTL for negative responses in [@!RFC2308].

## Zone operator guidance

If signers & DNS servers for a zone cannot immediately be updated to conform to this document, zone operators are encouraged to consider setting their SOA record TTL and the SOA MINIMUM field to the same value.
That way, the TTL used for aggressive NSEC use matches the SOA TTL for negative responses.

# Security Considerations

An attacker can prevent future records from appearing in a cache by seeding the cache with queries that cause NSEC(3) responses to be cached, for aggressive use purposes.
This document reduces the impact of that attack in cases where the NSEC(3) TTL is higher than the zone operator intended.

# IANA Considerations

IANA is requested to add a reference to this document in the "Resource Record
(RR) TYPEs" subregistry of the "Domain Name System (DNS) Parameters" registry, for the NSEC and NSEC3 types.


# Acknowledgements

Ralph Dolmans helpfully pointed out that fixing this in RFC8198 is only possible for negative (NXDOMAIN/NoData NOERROR) responses, and not for wildcard responses.

Warren Kumari gracefully acknowledged that the current behaviour of RFC8198, in context of the NSEC TTL defined in RFC4034, is not the intended behaviour.

{backmatter}

# Implementation Status

[RFC Editor: please remove this section before publication]

Implemented in PowerDNS Authoritative Server 4.3.0 https://doc.powerdns.com/authoritative/dnssec/operational.html?highlight=ttl#some-notes-on-ttl-usage .

Implemented in BIND 9.16 and up, to be released early 2021 https://mailarchive.ietf.org/arch/msg/dnsop/ga41J2PPUbmc21--dqf3i7_IY6M .

# Document history

