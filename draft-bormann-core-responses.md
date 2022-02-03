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
  -
    ins: C. Amsuess
    name: Christian Amsüss
    email: christian@amsuess.com

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

Non-traditional response:
: A response that is not the single response generated for a request received
  on the same transport.

Non-matching response:
  : A response that has properties
  (typically options) that make it incompatible with the original request,
  and thus in particular unsuitable as a cached response to that request (but
  possibly suitable to populate the cache for a similar request).
  Options that make a response non-matching need to be proxy unsafe.

  For example,
  a Block2 response with a different block number × block size value than indicated in the request is non-matching.

Configured request:
: A request that reaches the server in another way than by
  transmitting a usual CoAP request on the same communication channel
  a response is expected on.

Embedded request:
: A request that is provided by the server to the recipient of its
  response by embedding it into the response.

# Sending non-traditional responses

Non-traditional responses are sets of responses produced for a single request,
or responses sent without a transmitted request.

Where tokens are involved,
all non-traditional responses use the request's token;
in any case, they are bound to the original request
(e.g. by using the same reqest_kid/request_piv pair in OSCORE).
Where message IDs are involved,
one of the non-traditional response (the first sent, not necessarily the first received as generally the network might reorder messages)
can be sent as a piggy-back response in an ACK (thus sharing the request's message ID),
the others are CON or NON responses.

Some established responses
(observations defined in {{-observe}},
and multicast responses in {{?I-D.ietf-core-groupcomm-bis}})
match this definition and already follow the guidance set out here for non-traditional responses;
{{extensions-explained}} gives details for them.

A second response differing from the first that can be sent by a non-deduplicating server
responding to a retransmission of a request
is not non-traditional because there is a second request --
that is probably the last corner case at the line separating traditional from non-traditional responses.


## Preconditions to sending non-traditional responses

A server may send multiple responses to a request if there is any
property in the request that indicates the client's intention to receive
them. This is typically indicated by a request option,
and rarely in external properties of the message
(in the multicast case, the destination address).

A mechanism for eliciting multiple responses must specify the conditions
under which a token gets freed, as the traditional arrival of the
response is insufficient. It may also specify for which requests the
token can be reused immediately in follow-up requests. On unordered
transports, or when it's a client's follow-up request and not a response
that terminates the token, the client needs to wait until no reordered
non-traditional responses can be expected any more.

If a non-traditional response answers the original request, no further
action is required (this is the case of observation: ordering is added
on top of that to ensure that only the latest response is used). If
the response does not answer the original request,
it must be non-matching,
either by an option introduced with the eliciting option
or by a generic option like Response-For.


## Responses without request

Endpoints may agree out of band on a token (or other request-matching
details). One way to do that is to exchange a "phantom request", which
is a request that client and server will agree to have sent and
received, respectively, without it actually being sent between those
endpoints.

As tokens are managed by the client, that request needs to be
generated by the client, or in close collaboration with the client (for
example by the client allowing a third party to use a subset of its
token values in order to set up non-traditional responses).

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


## Leisure-For-Responses

This new option allows the server
to send up as many responses as indicated in the option's uint value in
addition to the requested response. They are to be sent immediately
after the original response.

| No. | C | U | N | R | Name                  | Format | Length | Default |
|-----|---|---|---|---|-----------------------|--------|--------|---------|
| TBD |   | U | - |   | Leisure-For-Responses | uint   |    1-4 |       0 |

The option is elective, but unsafe for proxies
(as the option would otherwise cause multiple responses to a proxy that expects only one and that needs to be a matching response).
A proxy that chooses not to implement it may forward the request
with the Leisure-For-Responses option removed.

On its own, the option does not indicate which more responses the client
would expect (though further elective proxy-safe no-cache-key options
can be added on top of that to give better guidance), and the server may
choose not to send any at all.

Intermediaries may add or remove the option, and use incoming responses to
populate their cache. They may serve additional responses from their
cache, but in most cases the sensible course of action is to forward the
additional responses the origin server sends.

Use cases for Leisure-For-Responses include sending further blocks in a
Block2 transfer (which are obviously non-matching and thus don't need a
Response-For), or serving follow-up documents (a response containing a
single link can be followed by a representation of the linked resource,
which needs a Request-For header that indicates the URI).
<!-- or just provide
the ETag of a freshly created resource (which would have a Reqeust-For
option for a GET with the given path and an ETag, and be a 2.03 Valid
response). / but that probably already works as there is the concept of a "tagged representation" -->


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


CoAP extensions explained by non-traditional responses {#extensions-explained}
==============================================================================

Observation
-----------

This section describes the Observe option {{?RFC7641}} in the terms of this
document, [ so nothing in here should contradict that document ].

When Observe:0 is present in a request, this sets up non-traditional
responses until either of the following conditions is met:

* A follow-up request on the same token carries an Observe:1 option.

  (This is primarily in here because; Observe:1 and No-Response:any
  could be combined; otherwise, the other conditions suffice).

* Any response does not carry an Observe option.
* Any response has a non-successful status.

Follow-up requests are limited to extending the request ETag set.
Responses are obviousy non-matching by their Observe option; each hop
discards the Observe option for the purpose of caching and refreshes its
cache with the most recent one as per the Observe value.

Responses to multicast requests
-------------------------------

As with observe, this just phrases the existing mechanism in the context
of this generalization.

When the destination address of a CoAP request is a multicast address,
that token is valid for any member of that group (which, for the purpose
of the client, is any server at all) on any port.

(Except for that the implications of having received a multicast request
still need to be followed, it might be seen as a template for creating a
phantom request to any endpoint, if that suits the reader's mental
model.)

Responses can only be sent for up to the deployment's Leisure time (see {{RFC7252}} Section 8.2) plus
the application's timeout (in proxy situations, this needs to be
communicated explicitly in the Multicast-Signaling option of
{{?I-D.tiloca-core-groupcomm-proxy}}).

Triangular responses (Response-To)
----------------------------------

The Response-To option can be viewed as a short-hand notation for
"Consider this a No-Response:any request, but take a copy of it, make it
into a CoAP-over-UDP request with that particular address as a source
and any address of yours as a response, and treat that as a phantom
request".

[ It may make sense to add an explicit return token, and include a
No-Response option; that might allow it to be used even across proxies. ]

Other current documents
-----------------------

{{?I-D.tiloca-core-observe-multicast-notifications}} is a
straightforward application of the phantom requests (the concept was
developed there); Leisure-For-Responses could help it around the topic
of joining a multicast group securely through a proxy.

{{?I-D.tiloca-core-groupcomm-proxy}} seems to fit well with the concepts
here as well, and might be simplified by it both in terminology and by
replacing Response-Forwarding with Response-For(Proxy-Scheme, Uri-Host).


Acknowledgements
================
{: numbered="no"}

TBD

<!--  LocalWords:  CBOR extensibility IANA uint sint IEEE endian
 -->
<!--  LocalWords:  signedness endianness
 -->
