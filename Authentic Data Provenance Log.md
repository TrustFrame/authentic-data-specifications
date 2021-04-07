# Community Specification Template 1.0

Community Specifications are recommended to be drafted in accordance with international best practices.  Doing so provides clarity, helps adoption, and also eases the transition of this specification to other standards body if so desired.  Accordingly, the recommended template below is based on ISO standard drafting conventions.

To help you, the ISO/TMP has published a [guide on writing standards][0].

A model manuscript is available of a draft International Standard known as [“The Rice Model”][1]

In addition, we recommend using the key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED",  "MAY", and "OPTIONAL" as described in RFC 2119 - https://tools.ietf.org/html/rfc2119

## Authentic Data Provenance Log Format
### Version 0.0.1
### Status Pre-draft

© 2021 TrustFrame, Inc.

This specification is subject to the [Community Specification License 1.0][2].

## Contents

1. [Foreword](#foreword)
2. [Introduction](#introduction)
   1. [Scope](#scope)
   2. [Normative References](#normative-references)
   3. [Terms and Definitions](#terms-and-definition)
3. [Events](#events)
   1. [Puzzle and Solution Scripts](#puzzle-and-solution-scritps)
4. [Associated Data](#associated-data)
   1. [Key Rotation](#key-rotation)
   2. [Registrars](#registrars)
   3. [Authentic Data](#authentic-data)
5. [Anchor Receipts](#anchor-receipts)
6. [Conclusion](#conclusion)

## Foreword

Attention is drawn to the possibility that some of the elements of this document may be the subject of patent rights. No party shall not be held responsible for identifying any or all such patent rights.

Any trade name used in this document is information given for the convenience of users and does not constitute an endorsement.

This document was prepared by TrustFrame, Inc.

Known patent licensing exclusions are available in the specification’s repository’s Notices.md file.

Any feedback or questions on this document should be directed to the [specification repository][3].

THESE MATERIALS ARE PROVIDED “AS IS.” The Contributors and Licensees expressly disclaim any warranties (express, implied, or otherwise), including implied warranties of merchantability, non-infringement, fitness for a particular purpose, or title, related to the materials.  The entire risk as to implementing or otherwise using the materials is assumed by the implementer and user. IN NO EVENT WILL THE CONTRIBUTORS OR LICENSEES BE LIABLE TO ANY OTHER PARTY FOR LOST PROFITS OR ANY FORM OF INDIRECT, SPECIAL, INCIDENTAL, OR CONSEQUENTIAL DAMAGES OF ANY CHARACTER FROM ANY CAUSES OF ACTION OF ANY KIND WITH RESPECT TO THIS DELIVERABLE OR ITS GOVERNING AGREEMENT, WHETHER BASED ON BREACH OF CONTRACT, TORT (INCLUDING NEGLIGENCE), OR OTHERWISE, AND WHETHER OR NOT THE OTHER MEMBER HAS BEEN ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

## Introduction

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

The design that follows is an opinionated one. The digest function used is the [Blake2b algorithm][4]. The public key cryptography algorithm used is [BLS][5]. The text encoding of binary data (e.g. digests and keys) when necessary is [Base58][6]. The script and stack machine used for the puzzle and solution in each event is a subset of [CCLang][7]. The default encoding for provenance logs is [BARE message encoding][8]. Wherever possible, this standard uses already established standards. In the few instances where no standard exists this standard strives to *minimize* the amount of invention.

### Scope

This specification...
* Describes the authentic data provenance log file format.

### Normative References

There are no normative references in this document.

### Terms and Definitions

For the purposes of this document, the following terms and definitions apply.

- **digital signature**: A mathematical scheme for verifying the authenticity of digital data.
- **provenance log**: A data structure made up of digitally signed events that tracks the provenance of authentic data such as signing keys and related data.
- **event**: A single record in a provenance log.
- **registrar**: An external oracle that abstracts away the process of storing authentic data identifiers in immutable storage.
- **anchor receipt**: A proof-of-existence from a registrar that a given authentic data identifier has been recorded in the immutable storage.

ISO and IEC maintain terminological databases for use in standardization at the following addresses:

* ISO Online browsing platform: available at https://www.iso.org/obp
* IEC Electropedia: available at http://www.electropedia.org/

## Events

All provenance logs are made up of digitally signed events that are linked together with hashes to form a hash-linked list. Each event must contain a minimum set of data fields to meet the requirements of the provenance log.

What follows is description of the fields in an event:

- **type**: The event object type identifier.
- **version**: The version of this format, current always 1.
- **timestamp**: Either an [RFC3339 timestamp][9] or it can also be a Bitcoin block height and block hash timestamp.
- **hash prev**: The Blake2b hash of the previous event and its associated anchor receipt(s).
- **associated data**: A list of one or more objects of associated data.
- **puzzle**: A script used to validate the next event in the log.
- **solution**: A script that provides the solution to the puzzle in the previous event used to validate this event.

Every event contains a timestamp of when it is created by the controller of the log. This may be different than the timestamp associated with the anchoring receipt from the registrar, however the timestamp in the event must always come before the timestamp in the anchoring receipt.

The "hash prev" field creates the hash-linked list of the provenance log and allows end-to-end integrity verification. As noted above, the associated data is a list of any kind of data.

### Puzzle and Solution Scripts

The puzzle and solution fields in an event are a pair of scripts for a stack machine that function exactly like the scriptPubKey locking script and scriptSig unlocking scripts on Bitcoin outputs and inputs. Each event has a puzzle script that the next event must solve with a solution script for the next event to be valid. When validating the next event in the log, the scripts are evaluated in the same manner as Bitcoin script: the solution script is combined with the puzzle script and evaluated on a stack machine.

```
| solution script | puzzle script |
```

By using puzzle and solution scripts we gain flexibility in the controllership of the provenance log and allow for more complex arrangements for the control including multi-party and cross-chain atomic swaps of controllers. Like Bitcoin there are standard types of scripts, the most common one being a simple digital signature check.

The digital signature of an event must include all fields except for the solution script.

## Associated Data

Every [event](#events) can have zero or more associated data objects in a list that record the data associated with the event. The types of objects in this list is intended to be extensible to accomodate future applications that are unforseen at the time of this writing. Each associated data object starts with an associated data type identifier so that implementations may skip over objects that it does not support. The initial set of associated data objects are listed below.

### Key Rotation

This establishes the current key by revealing the public key of the current key pair as well as committing to one or more future keys by revealing the digest of the future public keys. This is a post-quantum secure way of allowing for recovery of control of a log after a key theft or destruction event. A key rotation associated data object has the following fields:

- **type**: The key rotation type identifier.
- **version**: The version of the key rotation object schema, set to 1 for now.
- **key**: The new public key that is now the active key for this log.
- **next**: The list of Blake2B digests of one or more future key pairs.

### Registrars

This associated data specifies one or more registrars used to anchor the authentic data identifier for this provenance log. The fields are as follows:

- **type**: The registrars type identifier.
- **version**: The version of the registrars object schema, set to 1 for now.
- **registrars**: The list of one or more authentic data locators for registrar API endpoints used for anchoring this provenance log.

### Authentic Data

This establishes the cryptographic link to an external piece of data that this log is now tracking the provenance of. This is used to establish controllership over an external piece of data. The controller of this log is the controller of the external data as well. Typically this is the creator, and rights holder of the external data.

- **type**: The authentic data type identifier.
- **version**: The version of the authentic data object schema, set to 1 for now.
- **digest**: The Blake2b digest of the external data.
- **author**: A list of one or more authentic data identifiers of the digital identities of the creators of the external data.
- **sigs**: A list of one or more digital signatures over the external data created by the authors' digital identity keys.
- **locators**: A list of zero or more URLs that point to the external data.
- **data size**: The size of the external data.
- **data**: The external data itself.

In the authentic data object, the locators, data size, and data fields are optional but either the locators or the data size and data fields must be present. So either this object points to an external file or it contains the data directly.

## Anchor Receipts

When a provenance log is anchored at one or more registrars, the registrar issues an anchor receipt as a piece of authentic data. The anchor receipt contains information required by an external verifier to verify that the authentic data identifier for this log containing this event was anchored at the specified registrar. Each new event in a provenance log creates a new authentic data identifier for the log that incorporates the new event and must be anchored. The anchor receipt is then appended to the end of the event object so that a provenance log on disk is an alternating sequence of event object, one or more anchor receipts, event object, one or more anchor receipts, etc.

Anchor receipt objects have the following fields:

- **type**: The anchor receipt object type identifier.
- **version**: The version of the anchor receipt object schema, set to 1 for now.
- **timestamp**: Either an RFC3339 timestamp or it can also be a Bitcoin block height and block hash timestamp.
- **registrar**: The authentic data identifier for the registrar's digital identity. This indexes into the list of operative registrars stored in the registrars associated data object described above.
- **proof**: The associated proof data that contains the set proof for the authentic data identifier's inclusion in the current set and the identifier for the immutable record storage.
- **sig**: The script that verifies the digital signature over the receipt.

## Conclusion

Authentic data provenance logs are designed to be portable and an easily understood means for recording the provenance log of any digital identity and/or authentic data. This is just a quick and dirty draft specification based off of the open source implementation and will be updated with more details as the code firms up.

[0]: https://www.iso.org/files/live/sites/isoorg/files/developing_standards/docs/en/how-to-write-standards.pdf
[1]: https://www.iso.org/files/live/sites/isoorg/files/developing_standards/docs/en/model_document-rice_model.pdf
[2]: https://github.com/CommunitySpecification/1.0
[3]: https://github.com/TrustFrame/authentic-data-specifications
[4]: https://tools.ietf.org/html/rfc7693
[5]: https://link.springer.com/article/10.1007%2Fs00145-004-0314-9
[6]: https://tools.ietf.org/id/draft-msporny-base58-01.txt
[7]: https://github.com/dhuseby/cclang
[8]: https://baremessages.org/
[9]: https://tools.ietf.org/html/rfc3339
