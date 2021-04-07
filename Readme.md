# Authentic Data Specifications

This project manages the set of specifications for authentic data provenance logs and their associated URN, URL, and protocols. It is specified using the [Community Specification][0] developed via the [Joint Development Foundation][1], with inspiration from the [Open Web Foundation Agreements][2] and the [Alliance for Open Media Patent License 1.0][3].

These standards draw a great deal of inspiration from prior work at [Hyperledger][4], [Sovrin Foundation][5], [Decentralized Identity Foundation][6], and the [Internet Identity Workshop][7]. This is the culmination of years of learning and research and a great deal of thanks is due to the many people and organizations that created and grew the self-sovereign identity technology. However, it is now time to scale it all up into the authentic data economy starting with provenance logs.

## Introduction

The self-sovereign identity community has spent many years working at the W3C trying to develop standards for decentralized identity URI's (DID strings) and decentralized identity documents (DID documents). The end result, however, has three _major_ flaws that effectively make the standards completely unsuable.

### Problems with DID Strings

The first flaw is in the decentralized identity URI standard. The W3C proposed standard conflates identification with location causing complete chaos in the implementation and adoption of the standard. The standard says, "...the DID [string] identifies the DID subject and resolves to the DID document." By creating a standard URI format that both identifies _and_ resolves, it has created a ridiculous explosion of pet storage and provenance solutions, each one having their own decentralized identity method (DID method) standard. The [registry of DID methods][4] contains almost 100(!) different methods that a fully standards compliant implementation must support. Most of the DID methods require access to a running node for whatever blockchain is used for tracking provenance. This is simply not scalable and what is worse is that it is easily solvable. The existing W3C standard is just plain wrong.

### Problems with DID Documents

The second flaw is in the decentralized identity document standard. Ignoring the fact that text based formats (JSON and JSON-LD) were chosen for byte-sensitive, digitally signed documents, the existing standard simply doesn't meet the requirements for an Internet-scale provenance-driven digital identity solution. DID documents completely ignore the time dimension. To put it more precisely, they provide no solution for tracking the evolution of digital identities over time. There is no mechanism for recording key rotations and other provenance aggregations that occur over the lifetime of the controller.

It is considered good hygiene to rotate identity keys on a regular basis, even if they have not become compromised. The consequence of having old keys is that there is the potential for old digitally signed data to be stored for years and still require the old keys for verification. DID docs do not support this. Some DID method solutions such as Microsoft's ION do keep a history of the changes made to a DID document but those changes are not available in the "rendered" DID document retrieved from their storage facility. This is makes it extremely difficult for old data to be verified using old keys without writing code that is DID method specific. The end result is that any tool used for validating signatures will have to be programmed to work with each DID method server's API peculiarities for getting old keys. That's hardly portable.

### Problems with Portability

The last flaw in the existing standards is the lack of interchangeability of DID methods/services. Interchangeability is critical for portability of data and the sovereignty of users. Because not all DID methods are the same, moving from one method to another is neither automatic nor universal. Nor is there a standard for a DID document to keep a history of the DID methods that have been used to manage it throughout its past.

Truly decentralized systems use standard file formats and standard protocols to ensure that users can take there data from one system to another with minimal effort. The best example we have today is still email. Anybody can use an [IMAP][5] client to download all of their email from their email provider (e.g. GMail, Yahoo Mail, etc), store them locally in [.mbox][6] files, and then later use the IMAP client to upload them to a different email provider. This is only possible because of the standard protocol (IMAP) and standard file format (MBox) used with email.

## Authentic Data Provenance Logs

Authentic data provenance logs are intended to capture all relevant events in the history of a digital identity. They begin with an inception event that captures, at a minimum, a cryptographic key pair but can also include any kind of data that is also tied to, and managed in conjuction with the key pair.

The data structure is a hash-linked list of events that are digitally signed and validated using lock/unlock scripts similar to how Bitcoin scripts are used. They may contain any data and have references to external data. Identifying a particular log is done using a self-certifying "authentic data identifier" that is calculated from the log itself and is cryptographically linked. The trust and integrity of the logs comes from the anchoring of the log identifier in an external oracle. The only requirement is that the oracle is available to any verifiers at the time of verification.

When a log is published somewhere, the log identifier is incorporated into an "authentic data locator" that is digitally signed using the current key specified in the log. These logs are designed to be managed by the controller and stored wherever the controller wishes, even privately. By constructing locators from the identifier, the controller is able to store logs in multiple locations if they wish, each with a different locator. The digital signature over the locator is the cryptographic binding of the locator to the log. The locator resolves to the log that contains the public key to verify the signature over the locator. The log then also has the registrar(s) endpoints that can be used to get proofs of existence for the identifier calculated from the log itself.

In the end, this design follows closely the six principles of user sovereignty:

- Absolute privacy by *default*
- Absolute pseudonymity by *default*
- Strong, open encryption *always*
- Standard, open protocols and formats for *all data*
- *What*, not *who*, for authorization
- *Revocable* consent-based power structure

The first three are self-evident. The fourth is the motivation for this and the other related standards. The fifth is true simply because all authorization is done with the use of bearer tokens (i.e. cryptographic keys). The last is handled by the use of logs for everything. Interaction between any two controllers requires each to have a key pair for the interaction, managed by their respective logs. If the logs record every message that passes between the controllers, any consent requestes and granted is recorded in both logs. Revocation of consent is through unilateral key rotation by the consent granting controller.

When combined with the fact that the creation, maintenance, and storage of a log is handled by entirely replaceable tools and services and controllers are free to change tools and services at will, we have a complete system for provenance that is at least as decentralized as the current email system.

Authentic data provenance logs always manage keys and rotations and mixins of supporting data to link to the identity. But they can also manage any piece of digital data and provide the same level of assurance over arbitrary data as they do for key pairs. This is how things like non-fungible tokens (NFTs) can be made universal and no longer bound to a utility token in a blockchain silo.

The design that follows is an opinionated one. The digest function used is the [Blake2b algorithm][7]. The public key cryptography algorithm used is [BLS][8]. The text encoding of binary data (e.g. digests and keys) when necessary is [Base58][9]. The script and stack machine used for the puzzle and solution in each event is a subset of [CCLang][10]. The default encoding for provenance logs is [BARE message encoding][11]. Wherever possible, this standard uses already established standards. In the few instances where no standard exists this standard strives to *minimize* the amount of invention.

## Authentic Data URNs

## Authentic Data URLs

## Status

- [ ] Complete the pre-draft specification.
- [ ] Publish the prototype implementation.
- [ ] Publish article describing the thinking behind this approach.
- [ ] Incorporate feedback.

## Repository Contents

* The provenance log format is described in [Authentic Data Provenance Log.md][8].
* The URN format and the protocol for generating them is described in [Authentic Data Indentifier.md][9].
* The URL format and the protocol for generating them is described in [Authentic Data Locator.md][10].
* Instructions on how to contribute are in [Contributing.md][11].
* Our code of conduct is in [Code of Conduct][12].

[0]: https://github.com/CommunitySpecification/1.0
[1]: http://www.jointdevelopment.org
[2]: http://openwebfoundation.org
[3]: http://aomedia.org/license/patent-license/
[4]: https://hyperledger.org
[5]: https://sovrin.org
[6]: https://identity.foundation
[7]: https://iternetidentityworkshop.com
[8]: https://github.com/TrustFrame/provenance-log-specifications/blob/main/Authentic%20Data%20Provenance%20Log.md
[9]: https://github.com/TrustFrame/provenance-log-specifications/blob/main/Authentic%20Data%20Identifier.md
[10]: https://github.com/TrustFrame/provenance-log-specifications/blob/main/Authentic%20Data%20Locator.md
[11]: https://github.com/TrustFrame/provenance-log-specifications/blob/main/Contributing.md
[12]: https://github.com/TrustFrame/provenance-log-specifications/blob/main/Code%20of%20Conduct.md
