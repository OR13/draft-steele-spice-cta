---
title: "Credential Type Assertions"
category: info

docname: draft-steele-spice-cta-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Security"
workgroup: "Secure Patterns for Internet CrEdentials"
keyword:
 - next generation
 - unicorn
 - sparkling distributed ledger
venue:
  group: "Secure Patterns for Internet CrEdentials"
  type: "Working Group"
  mail: "spice@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/spice/"
  github: "OR13/draft-steele-spice-cta"
  latest: "https://OR13.github.io/draft-steele-spice-cta/draft-steele-spice-cta.html"

author:
 -
    fullname: "Orie Steele"
    organization: Transmute
    email: "orie@transmute.industries"

normative:
  I-D.draft-prorock-spice-cose-sd-cwt: SD-CWT
  I-D.draft-ietf-oauth-sd-jwt-vc: SD-JWT-VC
  RFC8392: CWT
  RFC8725: JWT-BCP

informative:
  I-D.draft-ietf-rats-eat: EAT
  I-D.draft-demarco-oauth-status-attestations: Status-Assertions
  I-D.draft-ietf-oauth-status-list: OAuth-Status-List


--- abstract

Requirements for physical supply chain credentials are subject to frequent changes due to regulatory, logistic and market pressures.
Preventing fraud and counterfeiting requires strong cryptographic assurance and binding to identities for people, organizations, vehicles and devices.
New information regarding a credential subject needs to be conveyed by holders to verifier, after credential issuance.
This specification describes the role of credential types or schemas in supporting changing requirements and realtime information regarding digital credentials.

--- middle

# Introduction

Requirements for physical supply chain credentials are subject to frequent changes due to regulatory, logistic and market pressures.
Preventing fraud and counterfeiting requires strong cryptographic assurance and binding to identities for people, organizations, vehicles and devices.
New information regarding a credential subject needs to be conveyed by holders to verifier, after credential issuance.
This specification describes the role of credential types or schemas in supporting changing requirements and realtime information regarding digital credentials.

When products in a physical supply chain are produced, it is common for certain required information to become available over time, sometimes as a result or tranformation processes which can be performed in different locations.

When creating a digital twin or product passport for a marketable product, the issuer needs to balance the need to create identifiers as soon as possible, so that claims can be associated to them, with the need to wait for sufficient claims to be available to justify a credential issuance.

A credential type or credential schema, is a set of mandatory and optional data elements, and is often based on a paper document such as a government form, or an evaluation or test performed by a trusted third party.

When specifying digital representation of product information, it is common to encounter data elements which are "required when present", but which do not become present for some time after credential issuance.

For example, when producing steel products, the chemical composition of steel is determined at the location and time when the steel is still in liquid form.
However, mechanical properties, including length, weight and other dimensions as well as material properties such as tensile strength become available later as the raw product is further refined on its journey to the end customer.
In particular, attributes such as protective coatings or finishing details which can impact weight and price may only be available shortly after the product is purchased.
These attributes are critical to intermediate supply chain parties, such as logistics providers, who need to load and unload product from vessels or vehicles.
Accurate information regarding dimensions and weight is critical to estimating fuel costs, and enabling advanced models of supply chain activity based on realtime digital twins.

In some cases, flaws in a batch of products are discovered only after transport has begun, and downstream supply chain partners need access to the latest information regarding a product in order to comply with regulatory or business policies.

Traditionally these challenges have been solved for with combinations of the following recommendations:

- Issue small credentials linked together with a common identifier
- Enable the latest revocation, suspension or other status information to be retrieved in realtime
- Use credential types or schemas to distinguish credentials with different required fields
- Use a confirmation method to selectively disclosed attributes to a single credential

Depending on the nature of the new information associated to a credential, these approaches can be difficult to implement, consume needless resources such as CPU, memory or network bandwidth, or reveal confidential supply chain metadata such as location information associated with a credential verification.

This document proposes a simple profile for JOSE and COSE based credentials, enabling simple, secure and efficient realtime digital twins.

# Terminology

{::boilerplate bcp14-tagged}

This document uses many terms common to the digital credential ecosystem and the OAUTH, SCITT, SPICE, JOSE and COSE working groups.

Media Type:
: JWT and CWT credential formats support the use of media types to identify the claimset (cty) and the entire secured object (typ). This specification relates these media type extension points to other application specific extension points such as verifiable credential type (vct).

Credential ID:
: CWT ID (CTI) and JWT ID (JTI) as defined in Section 3.1.7 of {{-CWT}}, are used to identify a digital credential represented as a token.

Credential Type:
: Section 3.2.2.1.1 of {{-SD-JWT-VC}} and Section TBD of {{-SD-CWT}} describe a stringOrURI based identifier for distinguishing required or optional JWT or CWT claims.

Credential Schema:
: [Verifiable Credentials Data Model v2.0](https://w3c.github.io/vc-data-model/#data-schemas) describes solutions for validating instances of a JSON data model. This specification generalizes this approach to support validating instances of JWT or CWT claimsets similar to the approach taken in {{-EAT}}, by relating the use of schema languages to the vct claim in JWT or CWT.

Credential Assertions:
: {{-Status-Assertions}} describes a mechanism whereby a holder can retrieve updated credential state from an issuer, and present it to a verifier.

Credential Status Lists:
: {{-OAuth-Status-List}} describes a mechanism whereby a verifier can retrieve updated credential states for a list of credentials from an issuer.

# Overview

This section describes how supply chain event driven architectures can be mapped to high level digital credential workflows to support digital twin ecosystems.

~~~ aasvg

+---------+ ... +-------------+ ... +-------------+ ... +--------------+
| Product |     | Product     |     |  Product    |     |  Regulations |
| Created |     | Transformed |     |  Transfered |     |  Updated     |
+-----+---+     +------+------+     +-------+-----+     +-------+------+
      |                |                    |                   |
      v                v                    v                   v
+-----+------+   +-----+------+     +-------+-------+   +-------+------+
| Credential |   | Credential |     |  Credential   |   |  Credential  |
| Identifier |   | Claims     |     |  Confirmation |   |  Schema      |
| Created    |   | Added      |     |  Updated      |   |  Updated     |
+------------+   +------------+     +---------------+   +--------------+

~~~

Although the figure above suggests a linear ordering of events, complex supply chains also known as value networks often combine component products into higher order products over time with events arriving out of order or for products which are not yet known to the stakeholder.

Regulations and therefor credential types requirements can change after or during a product custody transfers, making responsibility for compliance difficult.
When requirements change, or in certain policy situations, additional information might be required from one or more supply chain actors, for example a border agent might request specific information from the original manufacturer and the logistics providers that are partnered to transport products.

Businesses account for these changes today through a combination of messages exchanged over email or through proprietary digital document management systems.

As workflows for producing physical or digital products embrace transparency, traceability and security enabled by digital credentials, using the correct extension points at the the correct time becomes essential to reducing data duplication, and improving speed and security.

The following sections cover guidance for when to use specific extension points which are available to profiles built on JOSE and COSE credential formats.

# Entity Identifiers

It is important to distinguish subject entity identifiers (`sub` or `iss`) for a product, organization or device, from credential identifiers such as `cti` or `jti`.
Credential identifiers are used to name a set of secured claims about a subject.
Consider that in the future, a supply chain party may wish to aggregate all credentials associated with a product identifier such as serial number, including multiple credentials from the same or different issuers.
In these cases, the subject identifer can be considered as a primary key for the product, and the credential id can be considered as a foreign key for a table with columns representing the claims available for the token type (typ) and then finally the credential type (vct).

For example:

~~~ aasvg

Token Type                    Verifiable Credential Type
+-----------------------+     +--------------------------------------+
| typ: foo-passport+jwt |     | vct: https://.../foo-passport/v1.2.3 |
| (jti, iss, sub)       |     | (product_name, serial_number)        |
+---------+-------------+     +--------------------+-----------------+
          |                                        |
          |                                        |
          |         Credential Instance            |
          |         +----------------------+       |
          |         | jti: urn:uuid:123... |       |
          +-------->| ... typ claims ...   |       |
                    | ... vct claims ...   +<------+
                    | sub: urn:uuid:456... |
                    +-----------+----------+
                                |
Internal System                 |
+----------------------+        |
| id: 0xfacade123...   |        |
| sub: urn:uuid:456... +<-------+
+----------------------+

~~~

# Credential & Token Types

Per {{-JWT-BCP}} it is considered a best practice for both COSE and JOSE based digital credentials to distinguish types of tokens using the `typ` header parameter, which is used to indicate the media type of the associated token (including protected headers and signatures or ciphertexts).
As described in Section 3.2.2.1.1 of {{-SD-JWT-VC}} and Section TBD of {{-SD-CWT}} the type of a JWT or CWT claimset can also be constrained using the `vct` CWT or JWT claim.
It is recommended that both `typ` and `vct` be used when possible, and that application specific constraints which might be extended, updated or changed frequently be expressed through `vct`.
The specific semantics of `vct` have been left intentionally open to support many use cases.
A value of `vct` indicating `https://issuer.gov.example/foo-passport/v1.2.3` might be present in a credential of media type `application/foo-passport+jwt` and `application/foo-passport+cwt`.
In other words, it is recommended to not include media type parameters which using `typ` parameters.
When versioning is necessary, it should be performed in the context of mandatory and optional claims and communicated throught the `vct` property.
Although the example above includes a version identifier in the example URI, version information could be requested in other ways, including via HTTP headers.
When HTTP is used to transport credential type information it is recommended to support negotiation of the credential type requirements through the use of the Accept Header.
This enables human and machine readable requirements to be supported for each version of a digital credential.

For example:

```
GET https://issuer.gov.example/foo-passport
Accept: application/cddl or application/schema+json or text/html; charset=utf-8
```

## Schemas

Schemas are another name for verifiable credential type.
Schemas constrain the expected mandatory and optional fields, and can recognize if an instance of a credential is valid according a data definition lanugage such as JSON Schema or CDDL.
Schemas can be used to convey updated requirements associated with a credential, such as new fields which are mandatory in a new version.
A new schema and associated status assertions can be conveyed using the mechanism described in {{-Status-Assertions}}.
This enables a holder of a credential to disclose the original credential with confirmation, and any new information or requirements that have changed since the original credential was issued.

For example:

``` cbor-diag
/ cose-sign1 / 18([
  / protected / << {
    / alg / 1  : -35 / ES384 /
    / typ / 16 : "application/sd+cwt"
    / kid /    : "https://issuer.gov.example/key-42"
  } >>,
  / unprotected / {
    / disclosed claims /
    / sd_claims / TBD1 : <<[
        [
            / salt /   h'399c641e...2aa18c1e',
            / claim /  "region",
            / value /  "ca" / California /
        ],
        [
            / salt /   h'82501bb4...6c655f32',
            / value /  "4.1.7"
        ]
    ]
    / sd_kbt    / TBD2 : << [
      / protected / << {
          / alg / 1 : -35 / ES384 /
          / typ / 16 : "application/kb+cwt"
      } >>,
      / unprotected / {},
      / payload / <<
        / cnonce / 39    : h'e0a156bb3f',
        / aud     / 3    : "https://verifier.example",
        / iat     / 6    : 1783000000,
        / sd_alg  / TBD4 : -16  /SHA-256/
        / sd_hash / TBD3 : h'c341bb...4a5f3f',  / hash of sd_claims  /
                                                / using sd_alg       /
      >>,
      / signature / h'1237af2e...6789456'
    ] >>
  },
  / payload / <<
    / iss / 1      : "https://issuer.example",
    / sub / 2      : "https://device.example",
    / vct / TBD10  : "https://issuer.gov.example/foo-passport/v1.2.3",
    / exp / 2   : 1883000000,
    / iat / 2   : 1683000000,
    / cnf / 8   : {
      / cose key / 1 : {
        / alg: ES256 /  3: 35,
        / kty: EC2   /  1: 2,
        / crv: P-256 / -1: 1,
        / x / -2: h'768ed8...8626e',
        / y / -3: h'6a48cc...fd5d5'
      }
    },
    / status / TBD8 : {
      / status assertion / TBD9 : {
        / credential hash alg / : "sha-256"
      }
    },
    / sd_alg / TBD4        : -16, / SHA-256 /
    / redacted measurements / -65537 : [
      { TBD6 : h'45dd...87af'  / redacted_element / }
    ],
    "address": {
        "country" : "us",            / United States /
        / redacted_keys / TBD5 : [
            h'adb70604...03da225b',  / redacted region /
            h'e04bdfc4...4d3d40bc'   / redacted post_code /
        ]
    }
  >>,
  / signature / h'3337af2e...66959614'
])
```

The associated status assertion could convey updated measurments that the issuer was not aware of at the time they issued the associated SD-CWT.

For example:

``` cbor-diag
/ cose-sign1 / 18([
  / protected / << {
    / alg / 1  : -35 / ES384 /
    / typ / 16 : "application/status-assertion+cwt"
    / kid /    : "https://issuer.gov.example/key-42"
  } >>,
  / unprotected / {},
  / payload / <<
    / iss / 1   : "https://issuer.example",
    / sub / 2   : "https://device.example",
    / vct / TBD10  : "https://issuer.gov.example/foo-passport/v2.0.0",
    / exp / 2   : 1883000000,
    / iat / 2   : 1683000000,
    / credential hash / TBD : h'227a33...45975',
    / credential hash alg / TBD : "sha-256",
    / cnf / 8   : {
      / Note this must match the original confirmation method /
      / cose key / 1 : {
        / alg: ES256 /  3: 35,
        / kty: EC2   /  1: 2,
        / crv: P-256 / -1: 1,
        / x / -2: h'768ed8...8626e',
        / y / -3: h'6a48cc...fd5d5'
      }
    },
    / realtime updates for / -65548 : {
      / last location geohash / 282 : h'542aaa...66559ff'
    }
  >>,
  / signature / h'627aff...649596dd'
])
```

Note that in the example above the status assertion added realtime location information which was not present in the original credential type `https://issuer.gov.example/foo-passport/v1.2.3`, but which is mandatory in the new credential type `https://issuer.gov.example/foo-passport/v2.0.0`.

Status assertions allow for new schemas and new credential type mandatory properties to be added over time, without issuing fresh credentials or updating existing schemas.

However, there are cases where a new credential MUST be issued, and they are covered in the next section.

# Confirmation

Confirmation methods and their associated claims are used to ensure that only the authorized holder of a credential retaining the ability to use the associated confirmation method, can present the credential and receive facilitation benefits.

Confirmation is critical to prevent credential theft and forgery, but it introduces complications to supply chain workflows that might have previously relied on a bearer token or bearer documents.

When custody of a product is transfered for example in relation to a bill of lading and mate receipt for an ocean liner vessel, the controller of the product credential and the associated digital twin needs to be updated.

This can be accomplished through an issuance of a new credential, binding to the receiver of the product via their confirmation method, and a revocation of the previous credential to ensure the original holder is no longer able to present the proof of custody credential associated with the product shipment.

There exist other registry updates which might be required to support product traceability such as notifications to shared databases or new entries in a certificate transparency system.

After a new credential has been issued, the ability to provide status assertions transfers from the previous issuer, to the new issuer.

It is essential that the validity period of associated credentials be aligned to the revocation and issuance process to ensure that the latest product information come from the current authoritative issuer for the product, and that the fresh status assertions be presentable by the current holder, and with the latest confirmation methods.

# Status

Status information regarding a credential can be expressed in several ways including those described in {{-OAuth-Status-List}} and {{-Status-Assertions}}.
It is important to note that while status assertions are bound to the holder of a credential thought the associated confirmation method, status list information can be aggregated across several issuers over time, and issuer's might a previous issuer's authority to communicate status information for a newly issued credential.
In these cases, credential may have 1 or more associated status lists, and a single associated status assertion.

# Verifier Policies

Supply chain actors for who receive credential presentations decide how to interpret the associated claims presented.
In some cases, a verifier might reject presentations of a credential that did not include fresh status assertions for the latest version of a schema describing the required elements in a digital twin.
For example, if a vessel leaves a port with bill of lading version 1, and regulations change in transit, a port authority might reject presentations of bill of lading version 1 unless they are companied with an assoicated status assertion covering the required data elements associated with bill of lading version 2.
A verifier might also communicate with the holder out of band, and decide to accept the presentation of revoked or outdated credentials based on internal information of confidential evidence provided by the holder.
It can be useful, especially when crafting machine readable and enforcable verifier policies, to refer to token types `typ` and crendential types `vct` explicitly.
It can also be useful to refer to specific supported protocol versions in a machine readable way, because certain credential features are only available to specific credential protocols.

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
