---
title: "Partitioning as an Architecture for Privacy"
abbrev: "Partitioning for Privacy"
category: info

docname: draft-kuepau-iab-privacy-partitioning-latest
v: 3
keyword: Internet-Draft

author:
 -
    fullname: Mirja Kühlewind
    organization: Ericsson Research
    email: mirja.kühlewind@ericsson.com
 -
    fullname: Tommy Pauly
    organization: Apple
    email: tpauly@apple.com

normative:

informative:


--- abstract

This document describes the principle of privacy partitioning, which selectively spreads data and communication across
multiple parties as a means to improve the privacy by separating user identity from user actions.
This document describes emerging patterns in protocols to partition what data and metadata is
revealed through protocol interactions, provides common terminology, and discusses how
to analyze such models.

--- middle

# Introduction

The focus of many common security protocols, such as TLS or IPsec, is to prevent data
from being modified or seen by parties other than the protocol participants. Encrypting
and authenticating communication (in HTTP, in DNS, and more) is therefore a prerequisite for
user privacy by ensuring that information about user identity and activity cannot be
read by passively observing traffic.

However, this is not sufficient to provide a complete user privacy solution.
Another aspect of privacy has come into focus: preventing
protocol participants from being exposed to unnecessary data or metadata. Some examples
of this include:

- A user accessing a service on a website might not want to reveal their location,
but if that service is able to observe the client's IP address, it can infer many
location details.

- A user might want to be able to access content for which they are authorized,
such as a news article, without needing to have which specific articles they
read on their account being recorded.

- A client device that needs to upload metrics to an aggregated
service might want to be able to contribute into the aggregated metrics system without
having specific activities being tracked and identified.

These are all problems that involve needing to separate out which information
is seen at different steps of protocol interaction, or between different
participants in a protocol. In order to protect user privacy, it is therefore particularily
important to separate user identity (who) from user actions (what) whenever possible.

Several working groups in the IETF are working on solutions in this space, including
OHAI, MASQUE, Privacy Pass, and PPM. One commonality between these is that they
usually involve at least three parties: a client and at least two parties acting
as servers or intermediaries. While at first glance, the involvement of more parties
can seem counterintuitive for providing privacy, the ability to separate what data
is shared between these parties limits the amount of information about a single
client that can be concentrated by a server.

# Privacy Partitioning

This document defines "privacy partitioning" as the general technique used to separate the data
and metadata visible to various parties in network communication, with the aim of improving
user privacy.

At a high level, privacy partitioning can be described as separating *who* someone is
from *what* they do.

Partitioning is not a binary state, but a spectrum. It is difficult, and potentially impossible,
to completely guarantee that no metadata is linkable across various actions. Instead, as protocols
develop new techniques to partition data and metadata, it becomes easier to prevent entities
from being able to correlate information about a user and reduce their privacy.

## Communication Contexts

In order to define an analyze how various partitioning techniques work, the boundaries of what is
being partitioned need to be established.

This document defines the term "communication context" to refer to a set of data and metadata,
and the set of networked entities that share a common view of that data and metadata. Partitioning
creates more contexts where there would otherwise be a single context.

For example, in an unencrypted HTTP session over TCP, the communication context includes both the
content of the transaction as well as any metadata from the transport and IP headers; and the
participants include the client, routers, other network middleboxes, intermediaries, and server.

TODO: Diagram of a basic unencrypted client-to-server connection with middleboxes

Adding TLS encryption to the HTTP session is a simple partitioning technique that splits the
previous context into two separate contexts: the content of the transaction is now only visible
to the client, TLS-terminating intermediaries, and server; while the metadata in transport and
IP headers remain in the original context. In this scenario, without any further partitioning,
the entities that participate in both contexts can allow the data in both contexts to be correlated.

TODO: Diagram of how adding encryption splits the context into two.

Another way to create a partition is to simply use separate connections. For example, to
split two separate HTTP requests from one another, a client could issue the requests on
separate TCP connections, each on a different network, and at different times; and avoid
including obvious identifiers like HTTP cookies across the requests.

TODO: Diagram of making separate connections to generate separate contexts.

The privacy-oriented protocols described in this document generally involve more complex
partitioning, but the techniques to partition communication contexts still employ the
same techniques:

1. Encryption allows partitioning of contexts within a given network path.

1. Using separate connections across time and/or space allow partitioning of contexts for
different transactions.

## Identities and Correlation

Creating separate communication contexts is one important aspect of partioning, but
ensuring that the contexts are split to prevent correlation of identities is necessary
to make the partitions effective for improving privacy.

{{?RFC6973}} discusses the importance of identifiers in reducing correlation:

"Correlation is the combination of various pieces of information related to an individual
or that obtain that characteristic when combined... Correlation is closely related to
identification.  Internet protocols can facilitate correlation by allowing individuals'
activities to be tracked and combined over time."

{{?RFC6973}} goes on to discuss different ways that identities can be obscured: anonymity
and pseudonymity.

"To enable anonymity of an individual, there must exist a set of individuals that appear to
have the same attribute(s) as the individual."

"Pseudonymity is strengthened when less personal data can be linked to the pseudonym; when
the same pseudonym is used less often and across fewer contexts; and when independently
chosen pseudonyms are more frequently used for new actions (making them, from an observer's or
attacker's perspective, unlinkable)."

The "anonymity level" of a given identity exists on a scale, not a clear line between real
identity, pseudonymity, and anonymity. Some techniques for partitioning contexts assign
new pseudonymous or anonymous identities to clients within the context, and the selection of
these and the set of users that share the identity can greatly impact how effective
partitioning is.

For the purposes of user privacy, the identity that this document focuses on most is the
identity within a context that represents the user or the client; and which may be
transmitted within the context as explicit data or as implicit metadata. Every context
involves at least one identity for the client, and usually involves multiple - for example,
a user interacting with a website may have a logged-in user account identity as well as
an IP address identity.

In order to prevent correlation of two specific identities across communication
contexts, partitions need to ensure that any single entity (other than the client itself)
does not participate in contexts where both identities are visible. For example,
encrypting an HTTP exchange might prevent a network middlebox that sees a client IP address
from seeing the user account identity, but it doesn't prevent the TLS-terminating server
from observing both identities and correlating them. As such, preventing correlation
requires making contexts more disjoint, such as by using proxying.

## Mitigating Collusion

Partitioning has the goal of not allowing a single entity to gather information beyond the context
that a user or client intends. It can make trivial correlation of data and identities difficult for
a single entity. However, designing protocols to use partitioning cannot (alone) prevent
tracking if entities across the different contexts collude and share data. Thus, partitioning is not a
panacea, but rather a tool, and a necessary condition to achieve privacy.

Other techniques that can be used to mitigate collusion include:

- Policy and contractual agreements between entities involved in partitioning, to disallow
logging or sharing of data, or to require auditing.

- Protocol requirements to make collusion or data sharing more difficult.

- Adding more partitions and contexts, to make it increasingly difficult to collude with
enough parties to recover identities.

# Archictural Models using Partitioning

In order to achieve privacy partitioning, more entities than the user (data owner) and service provider need
to be involved in the communication architecture. Usually at least one entity is introduced that knows the user's identity and can
attest certain properties or rights that are connected with that identity (attester). However, this entity should not require
any knowledge about the service used or related user actions that could reveal privacy-sensitive data about the user or the user behavior.
Therefore another entities might be needed that has knowledge about the service that is requested by the user and its requirements but relies
on an anonymous identify provided by the attester (???).

The following section discusses currently on-going work in the IETF and how the privacy partitioning princplie is applied.

## OHAI

Oblivious HTTP introduces a Proxy Resource and a Request Resource. The Proxy Resource can identify the user based on its IP address, however, it can not see the user request that can only be only decapsulated by the Request Resource which in turn only sees the IP address of the proxy.

## ODoH

Oblivious DoH applies the same principle as Oblivious HTTP to DNS...

## MASQUE

MASQUE provide an HTTP3-based generic proxying framework similar as many VPN services today. Such a proxy can be used conceal user identity/IP address from the server. However, the proxy itself still has full visibility. Only if the MASQUE framework is used with at least two encapsulated proxies, identify and user actions can be decouples for all entities involved.

MASQUE / OHTTP / ODoH decouple data on a network path. Decoupling in space, effectively. Other decoupling strategies necessarily have to build on top of that.

## PrivacyPass

The privacypass protocol connects thee client to an attester that forwards a request to a token issuer. This token can then be used by the client to accesss services anonymously.

Privacy Pass decouples trust/attestation using blinding, and relies on decoupled paths.

## PPM/PRIO

PPM doesn't necessarily separate identity from other user data, however, enables anonymous data collection (without the need of knowledge of identity) and ensures anonymity by only providing partial data to each so-called leader while still enabling analysis of aggregated data by the collector.

PPM decouples based on computation, but also relies on separate servers / network paths.

# Impacts of Partitioning

Applying privacy partitioning to communication protocols lead to a substantial change in communication patterns.
Instead of sending traffic directly to a service, essentially all user traffic is routed through a set of intermediaries.
Information has has been observed passively in the network or metadata that has been unintentionally revealed to the service provider
cannot be used anymore for e.g. existing security procedures such as access rate limiting or DDoS mitigation.

However, network management techniques deployed at present often rely on information that is exposed by most traffic but without any guarantees that the information is accurate. Privacy partitioning provides an opportunity for improvements in these management techniques by providing opportunities to actively exchange information with each entity in a privacy-preserving way and requesting exactly the information needed for a specific task or function rather then relying on assumption that are derived on a limited set of unintentionally revealed information which cannot be guaranteed to be present and may disappear any time in future.

# Security Considerations

In these models, only the client knows the fully joined set of information. No server individually knows how to join all the data as long as servers do not collude the data. Therefore, for privacy partitioning to be applicable there must always be a non-collusion assumption between all involved entities.

Clients needs to be able to be explicit about what data they share (location, etc.) with which entity. All information shared need to be analysed based on its privacy-sensitivity and in relation to the entities/service it is shared with. In order to do this, clients need to be able to explicitly select different entities and hops over which to partition data. As a result, the client still needs to trust these entities to not collude but these entities also need to actively/more explicitly collude in order to rejoin data.

TLS and ECH are good examples of using encryption to hide information from most parties, but in that case clients and servers (two entities) still can join all informations. VPNs hide identity from the server but the proxy has still the full information. Therefore privacy partitioning requires at least two additional entities to avoid revealing both (identity (who) and user actions (what)) from each involved party.

Timing side-channels, etc, are still possible. They can of course be mitigated in very expensive ways by e.g. fuzzing the data, but it's important to point out that this general architecture doesn't prevent these attacks. It does, however, move the threat model to this kind of analysis rather than just looking at direct metadata.


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.