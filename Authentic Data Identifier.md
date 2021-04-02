# Community Specification Template 1.0

Community Specifications are recommended to be drafted in accordance with international best practices.  Doing so provides clarity, helps adoption, and also eases the transition of this specification to other standards body if so desired.  Accordingly, the recommended template below is based on ISO standard drafting conventions.

To help you, the ISO/TMP has published a [guide on writing standards][0].

A model manuscript is available of a draft International Standard known as [“The Rice Model”][1]

In addition, we recommend using the key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED",  "MAY", and "OPTIONAL" as described in RFC 2119 - https://tools.ietf.org/html/rfc2119

## Authentic Data Idenitifier
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
3. [Namespace ID Proposal](#namespace-id-proposal)
4. [Examples](#examples)
5. [Conclusion](#conclusion)

## Foreword

Attention is drawn to the possibility that some of the elements of this document may be the subject of patent rights. No party shall not be held responsible for identifying any or all such patent rights.

Any trade name used in this document is information given for the convenience of users and does not constitute an endorsement.

This document was prepared by TrustFrame, Inc.

Known patent licensing exclusions are available in the specification’s repository’s Notices.md file.

Any feedback or questions on this document should be directed to the [specification repository][3].

THESE MATERIALS ARE PROVIDED “AS IS.” The Contributors and Licensees expressly disclaim any warranties (express, implied, or otherwise), including implied warranties of merchantability, non-infringement, fitness for a particular purpose, or title, related to the materials.  The entire risk as to implementing or otherwise using the materials is assumed by the implementer and user. IN NO EVENT WILL THE CONTRIBUTORS OR LICENSEES BE LIABLE TO ANY OTHER PARTY FOR LOST PROFITS OR ANY FORM OF INDIRECT, SPECIAL, INCIDENTAL, OR CONSEQUENTIAL DAMAGES OF ANY CHARACTER FROM ANY CAUSES OF ACTION OF ANY KIND WITH RESPECT TO THIS DELIVERABLE OR ITS GOVERNING AGREEMENT, WHETHER BASED ON BREACH OF CONTRACT, TORT (INCLUDING NEGLIGENCE), OR OTHERWISE, AND WHETHER OR NOT THE OTHER MEMBER HAS BEEN ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

## Introduction

Authentic data identifiers (ADIs) are intended to provide a strong cryptographic binding between the identifier and the [authentic data provenance log][4] that they identify so that given any log, the identifier is calculated easily and given an identifier, the validity is easily checked. ADIs are a form of self-certifying identifier similar to [Tor hidden service ".onion" domain names][5] in that the identifier contains a cryptographic proof that it is referring to a specific piece of data. In the case of ADIs, the link is a cryptographic digest of data found within the log.

ADIs are a form of uniform resource name (URN) and adhere to the existing [URN standard][6] with an NID of "adi". Effort is being made to complete the [formal namespace process][7] for the "adi" NID because ADIs are intended to be functional on the global Internet and not limited to users in communities or networks not connected to the Internet.

Why a URN and not its own URI? Well, the functional requirements of a URN match exactly to the functional requirements of an authentic data identifier and the overall architecture of provenance logs and locators. The URN requirements are summed up in [The Curious History of Uniform Resource Names by Leslie Daigle][8] as:

> ...URNs are to be persistent, as well as global in scope and uniqueness, is pretty clear from the stated purpose. Additionally, the document stipulates that URNs are to be scalable (assignable to anything conceivably available on the network for hundreds of years) while supporting legacy naming systems, allowing independent assignment of identifiers by autonomous “naming authorities,” and still allowing “resolution” of URNs-that is, translation from the URN into one or more URLs.

Authentic data identifiers are persistent, global in scope, and unique. They last as long as the associated provenance log lasts. If we are talking about digital identities and authentic data, the associated logs could last forever and since a log can be combined with any data to create authentic data, these identifiers may be assigned to anything available on the network. Authentic data identifiers are calculated from the associated provenance log and are therefor self-assigned autonomously and with the associated [authentic data locator format and protocol][9], any authentic data identifier may be transformed into one or more URLs.

The authentic data identifier design that follows is an opinionated one. The digest function used is the [Blake2b algorithm][10]. The public key cryptography algorithm used is [BLS][11]. The text encoding of binary data (e.g. digests and keys) when necessary is [Base58][12]. Wherever possible, this standard uses already established standards. In the few instances where no standard exists this standard strives to *minimize* the amount of invention.

### Scope

This specification...
* Describes the authentic data identifier standard
* Describes the algorithm for calculating an authentic data identifier from an authentic data provenance log.

### Normative References

There are no normative references in this document.

### Terms and Definitions

For the purposes of this document, the following terms and definitions apply.

- **cryptographic digest**: A mathematical algorithm that maps data of an arbitrary size to a bit array of a fixed size.
- **provenance log**: A data structure made up of digitally signed events that tracks the provenance of authentic data such as signing keys and related data.

ISO and IEC maintain terminological databases for use in standardization at the following addresses:

* ISO Online browsing platform: available at https://www.iso.org/obp
* IEC Electropedia: available at http://www.electropedia.org/

## Namespace ID Proposal

This section contains all of the data for a formal namespace proposal as outlined in [RFC 3406 § Apendix A][13].

**Namespace ID:**

adi

**Registration Information:**

Registration version number: 1
Registration date:           N/A

**Declared registrant of the namespace:**

Registering organization
  Name:    TrustFrame, Inc.
  Address: N/A

Designated contact
  Role:  Technical Program Manager
  Email: info@trustframe.com

**Declaration of syntactic structure:**

The Namespace Specific String (NSS) of all URNs that use the "adi" NID will have the following structure:

urn:adi:{Inception Event Fingerprint}:[{Tail Event Sequence No.}:{Tail Event Fingerprint}]

Where the "{Inception Event Fingerprint}" is the [Base58][12] encoded [Blake2b][10] digest of the first event in the associated authentic data provenance log. It is a US-ASCII string that conforms to the URN syntax requirements in [RFC8141][14] and defines a specific namespace--the NSS--for addressing parts of the associated authentic data provenance log.

The "{Tail Event Sequence No.}" is the 0-based index of the most recent event in the associated authentic data provenance log. If a log has three events, then the sequence number will be 2, the index of the most recent event in the log. The sequence number is useful for quickly comparing the order of events as well as indexing into the list of events.

The "{Tail Event Fingerprint}" is the [Base58][12] encoded [Blake2b][10] digest of the most recent event in the associated authentic data provenance log. It is also a US-ASCII string that conforms to the URN syntax requirements in [RFC8141][14].

The tail event sequence number and fingerprint are optional components and are only used where completeness of the log must be guaranteed. Combined, they act like a versioning mechanism for the related provenance log by specifying the range of events that are expected to be present in the log. By combining the fingerprints of the first and last events in a provenance log, the ADI forms an end-to-end integrity proof over the events between the two due to the fact that the provenance log is a hash linked list. As events are added to the log, the tail event fingerprint is updated to the most recently added event.

When a provenance log only contains the inception event--event with sequence number 0--the sequence number and tail event fingerprint fields are redundant and should not be included in the URN. The vast majority of provenance logs will only ever have just the inception event and this compresses the amount of data required to store the identifiers.

**Relevant ancillary documentation:**

TrustFrame publishes the specification describing the formats and protocols that define the authentic data provenance log and related identifiers and locators. All documentation can be found in the [TrustFrame github repository][3].

**Identifier uniqueness considerations:**

Because the inception event fingerprint is calculated using a cryptographic digest with strong collision resistence, the uniqueness of each fingerprint has the same guarantee as the collision resistence of the [Blake2b][10] digest function. It is astronmically difficult to find two pieces of data that result in the same digest result and still adhere to the authentic data provenance log event format. It is so difficult that it is safe to assume that all inception event fingerprints will be unique.

**Identifier persistence considerations:**

Authentic data provenance log events are immutable once they have been created, digitally signed, and anchored in an external registrar. The inception event fingerprint is therefore immutable once the provenance log is established and anchored. Adding subsequent events to the log does not change the inception event fingerprint that forms the namespace for the given log.

**Process of identifier assignment:**

The identifier for each log is self assigned through a process of calculating the [Base58][12] encoding of the [Blake2b][10] digest of the provenance log inception event. To establish the proof-of-existence and long term integrity of the provenance log and its associated identifier, the identifier string must be "anchored" using an external oracle "registrar" that records the identifier in an immutable record that future verifiers can use to check the assertions made by the identifier and the associated log.

**Process for identifier resolution:**

The namespace is not listed with a Resolution Discovery System (RDS); this is not relevant.

**Rules for Lexical Equivalence:**

No special considerations; the rules for lexical equivalence of [RFC8141][14] apply.

**Conformance with URN Syntax:**

No special considerations.

**Validation mechanism:**

Validation requires the associated authentic data provenance log and consists of calculating the [Base58][12] encoding of the [Blake2b][10] digest of the inception event and doing a string compare with the NSS value to check if they match. If the URN contains the optional tail event sequence number and digest, validation requires calculating the fingerprint of the event identified by the sequence number and comparing with the values in the URN.

**Scope:**

Global

## Examples

An authentic data identifier for a provenance log with only one event that hash the [Blake2b][10] has value of "bda54eb8e94c7472328481738b5003acbdc433e0d7fefe7410d4f9bafb659379":

```
urn:adi:DmJFiNCt6qKBqL5TX9cofWkVA27wDQp4eXfaJh13XUwa
```

An authentic data identifier for a provenance log with three events, the inception event has a hash of "bda54eb8e94c7472328481738b5003acbdc433e0d7fefe7410d4f9bafb659379" and the tail event has a hash of "4b25fc6c77cf6935ac343162e41f559eb5af23c144e43e2ed15cc20fbc3d32cd":

```
urn:adi:DmJFiNCt6qKBqL5TX9cofWkVA27wDQp4eXfaJh13XUwa:2:64MAzrPhKLBHFRy25uC82Eo5MU1FC1XLdM2Jvsx2tTMv
```

## Conclusion

Provenance logs are designed to be portable and an easily understood means for recording the provenance log of any digital identity and/or authentic data. This is just a quick and dirty draft specification based off of the open source implementation and will be updated with more details as the code firms up.

[0]: https://www.iso.org/files/live/sites/isoorg/files/developing_standards/docs/en/how-to-write-standards.pdf
[1]: https://www.iso.org/files/live/sites/isoorg/files/developing_standards/docs/en/model_document-rice_model.pdf
[2]: https://github.com/CommunitySpecification/1.0
[3]: https://github.com/TrustFrame/provenance-log-specifications
[4]: https://github.com/TrustFrame/provenance-log-specifications/blob/main/Provenance%20Log%20Format.md
[5]: https://tools.ietf.org/html/rfc7686
[6]: https://tools.ietf.org/html/rfc8141
[7]: https://tools.ietf.org/html/rfc3406
[8]: https://www.ietfjournal.org/the-curious-history-of-uniform-resource-names/
[9]: https://github.com/TrustFrame/provenance-log-specifications/blob/main/Authentic%20Data%20Locator.md
[10]: https://tools.ietf.org/html/rfc7693
[11]: https://link.springer.com/article/10.1007%2Fs00145-004-0314-9
[12]: https://tools.ietf.org/id/draft-msporny-base58-01.txt
[13]: https://tools.ietf.org/html/rfc3406#appendix-A
[14]: https://tools.ietf.org/html/rfc8141
