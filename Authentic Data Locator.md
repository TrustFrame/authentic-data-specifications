# Community Specification Template 1.0

Community Specifications are recommended to be drafted in accordance with international best practices.  Doing so provides clarity, helps adoption, and also eases the transition of this specification to other standards body if so desired.  Accordingly, the recommended template below is based on ISO standard drafting conventions.

To help you, the ISO/TMP has published a [guide on writing standards][0].

A model manuscript is available of a draft International Standard known as [“The Rice Model”][1]

In addition, we recommend using the key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED",  "MAY", and "OPTIONAL" as described in RFC 2119 - https://tools.ietf.org/html/rfc2119

## Authentic Data Locator
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
3. [Authentic Data Locators](#authentic-data-locators)
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

Provenance logs are named/identified by a URN with the namespace ID of "adi" called an [authentic data identifier][4] (ADI). ADIs are specifically designed to not have a way of resolving them into a network resource to maximize the sovereignty of the user/controller. Authentic data locators (ADLs) are intended to provide a verifiable and resolvable URL that allows a client to download a copy of the associated [authentic data provenance log][5]. 

ADLs are a form of uniform resource identifier that specifically acts as a resource locator and conforms to the [Uniform Resource Identifier][6] specification. ADLs are strictly for locating a provenance log in a network since the identifier standard for logs is a URN as defined in [RFC 8141][7]. ADLs are constructed by combining the access method part with the authentic data identifier URN. To provide cryptographic integrity, a query portion consisting of a [Base58][8] encoded, [BLS][9] digital signature over the ADL is included. The BLS signature over the ADL is created by the controller using the BLS key pair active in the provenance log at the time of the ADL creation.

The construction of the ADL from the ADI allows for a single provenance log to be stored in multiple locations simultaneously creating multiple valid ADLs, one for each location. This approach increases resistence to denial-of-service attacks and also maximizes the freedom of provenance log controller to use this technology in multiple network contexts both public and private without any loss of security and trust.

### Scope

This specification...
* Describes the authentic data locator standard
* Describes the algorithm for calculating an authentic data locator from an access method and an authentic data identifier.

### Normative References

There are no normative references in this document.

### Terms and Definitions

For the purposes of this document, the following terms and definitions apply.

- **digital signature**: A mathematical scheme for verifying the authenticity of digital data.
- **provenance log**: A data structure made up of digitally signed events that tracks the provenance of authentic data such as signing keys and related data.

ISO and IEC maintain terminological databases for use in standardization at the following addresses:

* ISO Online browsing platform: available at https://www.iso.org/obp
* IEC Electropedia: available at http://www.electropedia.org/

## Authentic Data Locators

As described in the introduction above, the authentic data locator is constructed from the access method and the authentic data identifier and a BLS digital signature. The overall structure looks like:

```
  foo://example.com:1981/{provenance log identifier}?{BLS signature}
  \_/   \______________/\__________________________/ \_____________/
   |           |                     |                      |
scheme     authority      authentic data identifier  digital signature
```

All ADLs are access method agnostic as long as the resource can be identified by the ADI. This means the storage mechanism must *not* be content addressable because as events are added to the provenance log, its content--and therefor address--changes.

## Examples

An authentic data identifier for a provenance log with only one event that hash the [Blake2b][10] has value of `bda54eb8e94c7472328481738b5003acbdc433e0d7fefe7410d4f9bafb659379`:

```
urn:adi:DmJFiNCt6qKBqL5TX9cofWkVA27wDQp4eXfaJh13XUwa
```

If the provenance log file is stored on an HTTPS web server at `example.com/` then the resulting ADL would look like:

```
https://example.com/urn:adi:DmJFiNCt6qKBqL5TX9cofWkVA27wDQp4eXfaJh13XUwa?3nkZbRYAjswsTgb6PSeyEgUvfoKREn9cxe41UavgocFraxbDm2MHZQ6EWAs6qcVfqh
```

In this case the Base58 encoded query value `3nkZbRYAjswsTgb6PSeyEgUvfoKREn9cxe41UavgocFraxbDm2MHZQ6EWAs6qcVfqh` is the BLS signature over the string `https://example.com/urn:adi:DmJFiNCt6qKBqL5TX9cofWkVA27wDQp4eXfaJh13XUwa` created using the active key pair in the provenance log.

Given an authentic data provenance log with three events, the inception event has a hash of `bda54eb8e94c7472328481738b5003acbdc433e0d7fefe7410d4f9bafb659379` and the tail event has a hash of `4b25fc6c77cf6935ac343162e41f559eb5af23c144e43e2ed15cc20fbc3d32cd` then the identifier looks like:

```
urn:adi:DmJFiNCt6qKBqL5TX9cofWkVA27wDQp4eXfaJh13XUwa:2:64MAzrPhKLBHFRy25uC82Eo5MU1FC1XLdM2Jvsx2tTMv
```

If this log is stored on the FTP server at `ftp.u.washington.edu/pub/id` then the resulting locator string looks like:

```
ftp://ftp.u.washington.edu/pub/id/urn:adi:DmJFiNCt6qKBqL5TX9cofWkVA27wDQp4eXfaJh13XUwa:2:64MAzrPhKLBHFRy25uC82Eo5MU1FC1XLdM2Jvsx2tTMv?DFFZgbtiKTXJJ9d1qrXM8XjDt6HkL3d4d7e1sxpnuY1BVYyg8LgubjrkXHdz2V1iv
```

Again, in this example the Base58 encoded query value `DFFZgbtiKTXJJ9d1qrXM8XjDt6HkL3d4d7e1sxpnuY1BVYyg8LgubjrkXHdz2V1iv` is the BLS signature over the string `ftp://ftp.u.washington.edu/pub/id/urn:adi:DmJFiNCt6qKBqL5TX9cofWkVA27wDQp4eXfaJh13XUwa:2:64MAzrPhKLBHFRy25uC82Eo5MU1FC1XLdM2Jvsx2tTMv`

## Conclusion

Authentic data locators are a resolvable network resource locator with strong cryptographic linking to the authentic data identifier which is in turn cryptographically linked to the underlying provenance log. By separating locating from identification this maximizes user sovereignty without sacrificing any of the assurance created by the cryptography used to ensure integrity.

[0]: https://www.iso.org/files/live/sites/isoorg/files/developing_standards/docs/en/how-to-write-standards.pdf
[1]: https://www.iso.org/files/live/sites/isoorg/files/developing_standards/docs/en/model_document-rice_model.pdf
[2]: https://github.com/CommunitySpecification/1.0
[3]: https://github.com/TrustFrame/authentic-data-specifications
[4]: https://github.com/TrustFrame/authentic-data-specifications/blob/main/Authentic%20Data%20Identifier.md
[5]: https://github.com/TrustFrame/authentic-data-specifications/blob/main/Authentic%20Data%20Provenance%20Log.md
[6]: https://tools.ietf.org/html/rfc3986
[7]: https://tools.ietf.org/html/rfc8141
[8]: https://tools.ietf.org/id/draft-msporny-base58-01.txt
[9]: https://link.springer.com/article/10.1007%2Fs00145-004-0314-9
[10]: https://tools.ietf.org/html/rfc7693
