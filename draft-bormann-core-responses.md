---
title: >
  CoAP: Non-traditional response forms
docname: draft-bormann-core-responses-latest
# date: 2017-11-13

stand_alone: true

ipr: trust200902
keyword: Internet-Draft
cat: info

pi: [toc, sortrefs, symrefs, compact, comments]

author:
  -
    name: Carsten Bormann
    org: Universität Bremen TZI
    street: Postfach 330440
    city: Bremen
    code: D-28359
    country: Germany
    phone: +49-421-218-63921
    email: cabo@tzi.org

normative:
  RFC7252: coap
  I-D.ietf-core-object-security: oscore

informative:
  RFC7641: observe

entity:
  SELF: RFCthis

--- abstract

In CoAP as defined by RFC 7252, responses are always unicast back to a
client that posed a request.  The present memo describes two forms of
responses that go beyond that model.  These descriptions are not
intended as advocacy for adopting these approaches immediately, they
are provided to point out potential avenues for development that would
have to be carefully evaluated.

--- middle

Introduction        {#intro}
============

In CoAP as defined by RFC 7252, responses are always unicast back to a
client that posed a request.  A server may want to send a response to
a request that it did not receive, may want to multicast a response,
or both.

The descriptions in this specification are not intended as advocacy
for adopting these approaches immediately, they are provided to point
out potential avenues for development that would have to be carefully
evaluated.


## Terminology         {#terms}


The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in BCP
14 {{!RFC2119}} {{!RFC8174}} when, and only when, they appear in all
capitals, as shown here.

The term "byte" is used in its now customary sense as a synonym for
"octet".

Terms used in this draft:

Configured request:
: A request that reaches the server in another way than by
  transmitting a usual CoAP request on the same communication channel
  a response is expected on.

Embedded request:
: A request that is provided by the server to the recipient of its
  response by embedding it into the response.

# Response with embedded request

A server can send a response to a request that it did not actually
receive by embedding the request which the response answers in the
response.

The option "Response-For" contains a request packaged as in Section
5.2 of {{-oscore}}.  The response is then intended to serve as a
response to this request.


| No. | C | U | N | R | Name         | Format | Length | Default |
|-----|---|---|---|---|--------------|--------|--------|---------|
| TBD | C | - | - | - | Response-For | opaque | 0-1023 | (none)  |
{: #response-for-option title="The Response-For Option" cols="r l l l l l l r l"}

The CoAP Token becomes meaningless for this form of response;
responses with embedded requests are therefore sent with an
zero-length Token.  (In essence, the "Response-For" option takes the
place of the request the Token usually stands for.)

The congestion control considerations for confirmable and
non-confirmable messages apply unchanged.

# Response for configured request

A request may reach the server using a different means than that used
for the response.  For instance, the request may be configured in the server.
Without limiting generality, we speak about *configured requests*.

The client MUST be cognizant of that configuration as the request uses
a token from the token name space it controls.

## Examples for configured requests

## Example: Periodic request

A server may be configured to act on a configured request every day at 12:00.

## Example: Event driven request

A server may be configured to act on a configured request each time it reboots.

## Example: Configured observe

A server may be configured with a GET request from a client that
includes an Observe option with value 0.  This means that the server
will send updates to the state of the resource addressed by the GET
request to the configured address of the client.

The considerations of Section 4.5 of {{-observe}} apply.  How losing
interest reflects back into to configuration and whether there is some
form of error notification to the source of the configuration is out
of scope of the present specification.

## Multicast responses

A server MAY send a response to a multicast address.
(This needs to be a response to a configured request as a normal
request cannot be sent *from* a multicast address.)

Note that, as the originator of a multicast response is a unicast
address, the relaxation of matching rules described in Section 8.2 of
{{RFC7252}} does not apply.

The token space in CoAP is owned by the client, which is identified by
a transport endpoint (address/port).  Here, the address is a multicast
address, so the token name space is shared by all nodes joined to that multicast
address.  The assumption for multicast responses is that, for each
multicast group, there is some form of management for the token space
(and the port number) that everyone can participate that needs to
join that multicast group; the specific form of management is out of
the scope of this specification.  Note that this means that multicast
responses MUST NOT be sent to unmanaged multicast addresses such as
All Coap Nodes (Section 12.8 of {{-coap}}).

Multicast responses are always non-confirmable.  The congestion
control considerations for non-confirmable multicast messages apply
unchanged.

## Respond-To option

What has been called "configured request" here may also be triggered
by a usual CoAP request that carries the Respond-To option.
(The term "configured request" is still appropriate as the server
ought to be configured to accept this option; see {{seccons}}.)

If a single client wants to request a server to send the response to a
specific multicast address, it can include the "Respond-To" option.
This contains an opaque string with the port number as a 16-bit number
(in network byte order), followed by the IP address (4-byte IPv4 or
16-byte IPv6).

| No. | C | U | N | R | Name       | Format | Length | Default |
|-----|---|---|---|---|------------|--------|--------|---------|
| TBD | C | U | - | - | Respond-To | opaque |   6-18 | (none)  |

IANA Considerations
============

This draft adds the following option numbers to the CoAP Option
Numbers registry of
{{-coap}}:

| Number | Name         | Reference |
|--------|--------------|-----------|
| TBD    | Response-For | {{&SELF}} |
| TBD    | Respond-To   | {{&SELF}} |
{: #tab-option-registry title="CoAP Option Numbers"}


Security Considerations {#seccons}
============

TBD

(Clearly, multicast responses pose a potential for amplification, in
particular if unverified sources can cause them via Respond-To.
Discuss how to mitigate.)

A Respond-To option can be used to incite a server to send data to a
third party.  This ought not be done blindly, i.e., only with
considered application assent.

The CoAP request/response mechanism allows the client to ascertain a
level of authentication (not resistant though to on-path attackers
unless the communication is protected) and freshness of the response:
The Token echoed in the response shows that the responder had
knowledge of the (fresh) request (Section 5.3.1 of {{-coap}}).
Responses with embedded requests can not be authenticated or checked
for freshness this way.  Their content therefore is less trustworthy
than normal responses unless authenticated in another way (e.g., via
{{-oscore}}).

--- back



Acknowledgements
================
{: numbered="no"}

TBD

<!--  LocalWords:  CBOR extensibility IANA uint sint IEEE endian
 -->
<!--  LocalWords:  signedness endianness
 -->
