JWT VC Issuance Profile
==================

**Profile Status:** Pre-Draft

**Latest Draft:**
[https://identity.foundation/jwt-vc-issuance-profile](https://identity.foundation/jwt-vc-issuance-profile)

Editors:
~ [Kristina Yasuda](https://www.linkedin.com/in/kristina-yasuda-6263b5a2) (Microsoft)
~ [Brent Zundel](https://www.linkedin.com/in/bzundel/) (Gen Digital)

Contributors:
~ Tobias Looker (Mattr)
~ Andrew Hughes (Ping Identity)
~ David Waite (Ping Identity)
~ Daniel Godbout (Microsoft)
~ Rohit Gulati (Microsoft)

Participate:
~ [GitHub repo](https://github.com/decentralized-identity/jwt-vc-presentation-profile)
~ [File a bug](https://github.com/decentralized-identity/jwt-vc-presentation-profile/issues)
~ [Commit history](https://github.com/decentralized-identity/jwt-vc-presentation-profile/commits/main)

------------------------------------

## Abstract

The JWT VC Issuance Profile defines a set of specifications to enable the interoperable issuance of Verifiable Credentials between Wallets and Credential Issuers.

### Audience

The audience of the document includes verifiable credential implementers and/or enthusiasts. The first few sections give an overview of the problem area and profile requirements for JWT VC issuance interoperability. Subsequent sections are detailed and technical, describing the protocol flow and request-responses.

## Status of This Document

The status of the JWT VC Issuance Profile v0.1.0 is a PRE-DRAFT specification under development within the Decentralized Identity Foundation (DIF).

### Description

The [[ref: VC Data Model v1.1]] defines the data model of Verifiable Credentials (VCs) but does not prescribe standards for transport protocol, key management, authentication, query language, etc. As a result, implementers need to decide which standards to use for the issuance without a guarantee that others will support the same set of standards.

This document aims to provide a path to interoperability by standardizing the set of specifications that enable the issuance of JWT-VCs between implementers. It has been developed in conjuction with the [[ref: JWT VC Presentation Profile]] which describes the interoperable presentation of VCs between Wallets and Verifiers. Future versions of this document will include details on the Wallet interoperability. Ultimately, this profile will define a standardized approach to Verifiable Credentials so that distributed developers, apps, and systems can share credentials through common means.

### Scope

#### Scope

This document is currently scoped for the issuance of VCs between the Wallet and the Credential Issuer. The Wallet is a native application. The following aspects of the presentation are covered:

- Data model
- Protocol to issue VCs
- Identifiers of the entities
- Revocation of VCs

#### Out of Scope

The following items are out of scope for the current version of this document:
- Presentation of VCs. See [[ref: JWT VC Presentation Profile]].
- Advanced concepts in the [[ref: VC Data Model v1.1]] such as `credentialSchema` (`credentialType` is used instead), `refreshService`, `termsOfUse`, `evidence`, Selective disclosure and unlinkability, Zero-Knowledge Proofs, and Disputes.
- Non-native Wallets like web applications, PWAs, etc.
- The following [[ref: OpenID4VCI]] endpoints:
  - Deferred Credential Endpoint
  - Batch Credential Endpoint

Note: Although selective disclosure and unlinkability are out of scope of this document, future versions may include SD-JWT ([[ref:SD-JWT]]) once they become stable in IETF.

## Structure of this Document

A description to the reader on how the document is structured.

## Terminology

Common terms in this document come from the specifications listed in [[ref: References]].

## Profile

This interoperability profile is based on [[ref: OpenID4VCI]], limited to following features:
* Authorization Code flow
* Issuer Initiated flow and Wallet Initiated flow
* Immediate issuance
* Same-device and Cross-device Credential Offer

### The Protocol Flow

This section briefly describes the end-to-end issuance flow.

Concepts and terms mentioned here will be described in more detail in subsequent sections of this document.

#### Authorization Code Flow

In Authorization Code Flow, End-User Authentication is performed at the Credential Issuer's Authorization Endpoint.

```mermaid
sequenceDiagram
  participant u as End User
  participant w as Wallet
  participant iss as Issuer
  u ->> iss: interacts
  iss ->> w: (1) (optionally) Credential Offer
  u ->> w: interacts
  w ->> w: (2) Wallet obtains Issuer's server metadata
  w ->> iss: (3) Authorization Request<br>(type of credentials to be issued)
  note over w: User Authentication / Consent
  iss -->> w: Authorization Response (authorization code)
  w ->> iss: (4) Token Request (authorization code)
  iss -->> w: Token Response (access_token)
  w ->> iss: (5) Credential Request<br>(access_token, proof)
  iss -->> w: Credential Response (credential)   
```

(1) When the issuance flow is initiated by the Issuer, the flow begins with the Issuer generating a Credential Offer and communicating it to the Wallet. This is presented as a QR Code. When the issuance flow is initiated by the Wallet, this step is omitted and the flow begins directly with step 2. 

(2) Wallet obtains Issuer's metadata including details about the credential that this issuer wants to issue. In the Issuer initiated flow, Wallet usually uses information in the Credential Offer to do so.

(3) Wallet sends a Authorization Request to the Issuer's Authorization Endpoint. Issuer returns Authorization Response with the authorization code after successfully authenticating and obtaining consent from the End-User. This step happens via a frontchannel, by redirecting the End-User via the user agents.

(4) Wallet sends a Token Request to the Issuer's Token Endpoint with the authorization code obtained in step 3. Issuer returns an Access Token in the Token Request after successfully validating the authorization code. This step happens via the backchannel using server to server communication. 

(5) Wallet sends a Credential Request to the Issuer's Credential Endpoint with the Access Token and proof of control over the public key to which the the issued VC shall be bound. Upon successfully validating the Access Token and proof, the Issuer returns a VC in the Credential Response if it is able to issue the credential right away. 

### Overview of the Open Standards Requirements

- As the protocol to issue VCs, OpenID for Verifiable Credentials Issuance MUST be used as defined in [[ref: OpenID4VCI]]. Annex E.1.1 MUST be used as a Credential Format Profile. PKCE as defined in [RFC 7636](https://www.rfc-editor.org/rfc/rfc7636.html) MUST be supported.
- VCs MUST adhere to the [[ref: VC Data Model v1.1]] and be serialized as JSON and signed as a JWS as defined in 6.3.1 of [[ref: VC Data Model v1.1]]. VCs serialized as JSON-LD and signed using Linked Data Proofs are NOT supported.
- Decentralized Identifiers (DIDs), as defined in [[ref: DID Core]], MUST be used as identifiers of the entities. Implementations MUST support did:web, did:ion (long-form), and did:jwk as mandatory DID methods as defined in [[ref: did-web]], [[ref: did-ion]], and [[ref: did-jwk]].
- DID Documents MUST use either `JsonWebKey2020` or `EcdsaSecp256k1VerificationKey2019` as the type for Verification Material intended for use in the profile. ([[ref: DID Core]] Section 5.2.1)
- Verification Material intended for use in the profile MUST use `publicKeyJwk` ([[ref: DID Core]] Section 5.2.1). The keys MUST be secp256k1 or Ed25519, see the _Cryptographic Signature_ section.
- DID Documents may contain Verification Material not intended for use with this profile of any Verification Material Type and any key format or algorithm.
- To bind an owner of an Issuer's DID to a controller of a certain origin, a Well Known DID Configuration MUST be used as defined in [[ref: Well Known DID]].
- For Revocation of VCs, Status List 2021 as defined in [[ref: Status List 2021]] MUST be used in combination with ID Hubs as defined in [[ref: Identity Hub (0.0.1 Predraft)]].

### Credential Offer

Credential Offer MUST contain the parameters as defined in Section 4.1 and Annex E.1.1.3. of [[ref: OpenID4VCI]].

Credential Issuer's metadata SHOULD be obtained using `credential_issuer` parameter as defined in Section 10.2 of [[ref: OpenID4VCI]].

The Credential Offer URL MUST use the custom URL scheme `openid-vc://` and the Issuer and the Wallet provider MUST support this URL.

`grants` object with Grant Type `authorization_code` MUST be present, when `issuer_state` parameter is conveyed.

Issuer MUST use either a QR code or deep link to communicate the request.

#### Sending Credential Offer by Value

The Credential Issuer MUST send Credential Offer by value in a single, self-contained parameter `credential_offer_uri` as defined in Section 6.1 of [[ref: OpenID4VCI]].

Below is a non-normative example of the Credential Offer displayed by the Credential Issuer as a QR code when the Credential Offer is passed by value:

```
openid-credential-offer://?
  credential_offer=%7B%22credential_issuer%22%3A%22https%3A%2F%2Fserver.example.com%2FverifiableCredential%2Fcontracts%2FVerifiedEmployee%22%2C%22credentials%22%3A%5B%7B%22format%22%3A%22jwt_vc_json%22%2C%22types%22%3A%5B%22VerifiableCredential%22%2C%22VerifiedEmployee%22%5D%7D%5D%2C%22grants%22%3A%7B%22authorization_code%22%3A%7B%22issuer_state%22%3A%22djHkmsG8gtS%22%7D%7D%7D
```

Below is a Credential Offer object included in a Credential Offer above:

```
{
  "credential_issuer": "https://server.example.com/verifiableCredential/contracts/VerifiedEmployee",
  "credentials": [
    {
      "format": "jwt_vc_json",
      "type": [
        "VerifiableCredential",
        "VerifiedEmployee"
      ]
    }
  ],
  "grants": {
    "authorization_code": {
      "issuer_state":"djHkmsG8gtS"
    }
  }
}
```

#### Sending Credential Offer by Reference

The Credential Issuer MAY communicate Credential Offer by reference, which is particularly useful for large requests. If passing by reference, the following parameter is required:

* `credential_offer_uri`: OPTIONAL for the Credential Issuer. REQUIRED for the Wallet. A URL using an `https` scheme referencing a resource containing a Request Object, which is a JSON object containing the Credential Offer parameters. MUST NOT be present when Credential Offer is sent by value.

The Wallet will perform a HTTP GET request to the `credential_offer_uri` to obtain the Credential Offer Object. The response from the Credential Issuer MUST use the media type `application/json` and MUST NOT be signed.

Below is a non-normative example of the Credential Offer displayed by the Credential Issuer as a QR code when the Credential Offer is passed by refernce: 

```
openid-vc://?
  credential_offer_uri=https%3A%2F%2Fserver%2Eexample%2Ecom%2Fcredential-offer
```

Below is a non-normative example of a response from the Credential Issuer that contains an Credential Offer Object:

```
HTTP/1.1 201 OK
Content-Type: application/json

{
  "credential_issuer": "https://server.example.com/verifiableCredential/contracts/VerifiedEmployee",
  "credentials": [
    {
      "format": "jwt_vc_json",
      "type": [
        "VerifiableCredential",
        "VerifiedEmployee"
      ]
    }
  ],
  "grants": {
    "authorization_code": {
      "issuer_state":"djHkmsG8gtS"
    }
  }
}
```

Note: [RFC 9101](https://datatracker.ietf.org/doc/html/rfc9101) might suggest the usage of `application/jwt` media type in the response so that the URL-safe claims representation and processing can be the same in both the signed and unsigned cases as described in [RFC 8725](https://datatracker.ietf.org/doc/html/rfc8725). However, since Credential Offer cannot be signed, `application/json` media type is used for simplicity and to avoid using `"alg": "none"`.

#### Credential Offer Response

There is no Credential Offer Response from the Wallet. UX control stays with the Wallet after completion of the process.

### Authorization Endpoint

The Authorization Endpoint is used in the same manner as defined in [[ref: RFC 6749]].

#### Authorization Request

The Wallet MUST use OAuth 2.0 scopes to obtain an Access Token scoped to receiving Credentials from the Credential Issuer's Credential Endpoint. The Credential Issuer MAY also use OAuth 2.0 scopes to indicate specific types of credentials to issue. `scope` values can be obtained from the Credential Offer.

##### `client_id` of the Wallet

A Wallet that has pre-registered with the Authorization Server MUST use the `client_id` value obtained during that process.

However, because of the potentially large number of Credential Issuers and the Wallets, there may be cases when a Wallet has not pre-registered directly with the Authorization Server, or is unable to do so. 

When an Authorization Server accepts Wallets that are not pre-registered, it MUST define a `public_client_id` in its Credential Issuer metadata. Credential Issuer is responsible for pre-generating a valid `public_client_id` which can be used by the Wallets.

A Wallet that has not pre-registered with the Authorization Server MUST use a `public_client_id` value obtained from the Credential Issuer metadata as a `client_id` thus acting as a public client. Pre-registered Wallets MUST ignore `public_client_id` value even if it is present in the Credential Issuer metadata. 

Note that some Authorization Servers may only allow pre-registered Wallets, in which case `public_client_id` property MUST be absent from the Credential Issuer metadata.

For the security considerations, see [Using of `public_client_id` as a `client_id`](#using-of-public_client_id-as-a-client_id).

##### Scope Parameters

Credential Issuer MUST define additional scopes in its metadata document that the Wallet can use to obtain the Access Token valid at the Credential Endpoint. The Wallet MUST check for the scopes values in the Credential Issuer's metadata. 

Note: scope parameter values are case sensitive as defined in [[ref: RFC 6749]].

If Authorization Server does not understand the value in the scope, it MUST return an Authorization Error Response.

Authorization Servers MUST ignore unknown scopes in a request.

Multiple scope parameters MAY be present in a single Authorization Request whereby each occurrence MUST be interpreted individually.

#### Client Metadata Passed in the Authorization Request

Wallets MAY use the `scope` parameter of an Authorization Request to request the issuance of a specific credential type.

Below is a non-normative example using the scope `verifiedEmployee` to request a Verified Employee Credential (uses PKCE as defined in [RFC 7636](https://www.rfc-editor.org/rfc/rfc7636.html)).

```
GET/ https://server.example.com/authorize?
  response_type=code
  &scope=verifiedEmployee
  &client_id=s6BhdRkqt3
  &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
```

The `authorization_details` parameter is not supported.

#### Successful Authorization Response

Authorization Responses MUST be made as defined in [[ref: RFC 6749]].

Below is a non-normative example of a successful Authorization Response:

```
HTTP/1.1 302 Found
  Location: https://wallet.example.org/cb?
    code=SplxlOBeZQQYbYS6WxSbIA
```

#### Authorization Error Response

Authorization Error Response MUST be made as defined in [[ref: RFC 6749]].

When the requested scope value is invalid, unknown, or malformed, the AS should respond with the error code invalid_scope defined in Section 4.1.2.1 of [[ref: RFC 6749]].

Below is a non-normative example of an unsuccessful Authorization Response:

HTTP/1.1 302 Found
Location: https://client.example.net/cb?
    error=invalid_request
    &error_description=Unsupported%20response_type%20value

### Token Endpoint

The Token Endpoint issues an Access Token and in exchange for the authorization code that client obtained in a successful Authorization Response. It is used in the same manner as defined in [[ref: RFC 6749]] and follows the recommendations given in [[ref: OAuth 2.0 Security Best Current Practice]].

#### Token Request

Upon receiving a successful Authorization Response, a Token Request is made as defined in Section 4.1.3 of [[ref: RFC 6749]].

The token request MUST use the grant_type `authorization_code`. The requirement as described in Sections 4.1.3 and 3.2.1 of [[ref: RFC 6749]] MUST be followed.

Since the Client does not authenticate itself at the Token Endpoint, the Client Authentication method is none.

Below is a non-normative example of a Token Request:

```
POST /token HTTP/1.1
  Host: server.example.com
  Content-Type: application/x-www-form-urlencoded
  Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW

  grant_type=authorization_code
  &code=SplxlOBeZQQYbYS6WxSbIA
  &code_verifier=dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk
  &redirect_uri=https%3A%2F%2FWallet.example.org%2Fcb

```

#### Successful Token Response

Below is a non-normative example of a Token Response:

```
HTTP/1.1 200 OK
  Content-Type: application/json
  Cache-Control: no-store

  {
    "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6Ikp..sHQ",
    "token_type": "bearer",
  }
```

#### Token Error Response

If the Token Request is invalid or unauthorized, the Authorization Server constructs the error response as defined as in Section 5.2 of OAuth 2.0 [[ref: RFC 6749]].

### Credential Endpoint

#### Credential Request

Credential Request MUST be made by sending an HTTP POST request to the Credential Endpoint. 

Credential Request MUST include parameters as defined in Section 7.1 and Annex E.1.1.5. of [[ref: OpenID4VCI]] with the following requirements:
- `format` value MUST be `jwt_vc_json`
- `proof` parameter MUST be present
  - `proof_type` value MUST be `jwt` 
  - `jwt` parameter MUST be present 

In the JWT header of a proof JWT, `kid` Claim MUST be present, since the credential will be bound to a Holder's DID. `kid` MUST refer to a DID URL which identifies a particular key in the DID Document that the credential shall be bound to.

In the JWT body, `iss` Claim MUST be present.

Below is a non-normative example of a Credential Request (with line breaks for readability only):

```
POST /credential HTTP/1.1
Host: server.example.com
Content-Type: application/json
Authorization: BEARER czZCaGRSa3F0MzpnWDFmQmF0M2JW

{
  "format": "jwt_vc_json",
  "credential_definition": {
    "type": [
      "VerifiableCredential",
      "VerifiedEmployee"
    ]
  },
  "proof": {
    "proof_type": "jwt",
    "jwt": "eyJraWQiOiJkaWQ6ZXhhbXBsZTplYmZlYjFmNzEyZWJjNmYxYzI3NmUxMmVjMjEva2V5cy8
    xIiwiYWxnIjoiRVMyNTYiLCJ0eXAiOiJKV1QifQ.yJpc3MiOiJzNkJoZFJrcXQzIiwiYXVkI
    joiaHR0cHM6Ly9zZXJ2ZXIuZXhhbXBsZS5jb20iLCJpYXQiOiIyMDE4LTA5LTE0VDIxOjE5O
    jEwWiIsIm5vbmNlIjoidFppZ25zbkZicCJ9.wdkIkPV50iOeBUqMXCC_aZKPxgihac0aW9Ek
    L1nOzM“
  }
}

```

Note that the content-type MUST be `application/json`.

#### Credential Response

Credential Response is immediate meaning that the Credential Issuer MUST be able to immediately issue a requested credential and send it to the Wallet. Deferred credential response using the Deferred Credential Endpoint is NOT supported in this profile.

Each issued Credential MUST be a JWT encoded VC and formatted as a string as defined Annex E.1.1.6. of [[ref: OpenID4VCI]]. The parameter `format` MUST be `jwt_vc_json`.

`credential` parameter is REQUIRED.

Below is a non-normative example of a Credential Response:

```
HTTP/1.1 200 OK 
Content-Type: application/json 
Cache-Control: no-store 

{
 "format": "jwt_vc_json",
 "credential" : "LUpixVCWJk0eOt4CXQe1NXK....WZwmhmn9OQp6YxX0a2L",
} 
```

#### Credential Error Response

When the Credential Request is invalid or unauthorized, the Credential Issuer constructs the error response as defined in Section 7.3.1 of [[ref: OpenId4VCI]].

#### Batch Credential Endpoint

The batch credential endpoint as defined in Section 8 of [[ref: OpenID4VCI]] is NOT supported with this profile.

### Credential Issuer Server Metadata 

Credential Issuer's metadata MUST be compliant with Section 10.2.3 and Annex E.1.1.2 of [[ref: OpenID4VCI]]. This section extends that metadata with the following parameters:

* `public_client_id`: OPTIONAL for the Credential Issuer to include. REQUIRED for the Wallet to be able to understand this parameter when it is provided by the Credential Issuer and the Wallet is not pre-registered. A string representing a Client Identifier for a Wallet that has not pre-registered with the Authorization Server. This is a parameter introduced by this profile.

Credential Issuer's metadata MUST be signed using Credential Issuer's DID when the Credential Issuer is unable to host its metadata under its own domain.

To bind an owner of a DID to a controller of a certain origin, Well Known DID Configuration MUST be used as defined in [[ref: Well Known DID]]. For more details, see the Linked Domain Verification section in [[ref: JWT VC Presentation Profile]].

### End-user Consent

When the Wallet displays the consent screen to the user, it is RECOMMENDED to display the domain name obtained using [[ref: Well Known DID]]'s Linked Domain verification. Displaying details of the consent using `display` registration parameters such as `name`, `logo`, and `description` defined in the Section 10.2.3.1 of [[ref: OpenID4VCI]] is OPTIONAL.

Note that displaying the domain name of the Credential Issuer helps the End-Users identify malicious Issuers who have copied registration parameters of good Issuers and are impersonating them.

### Decentralized Identifiers

For requirements on Decentralized Identifiers, see Section 5.8 of [[ref: JWT VC Presentation Profile]. In particular, the description of short-form and long-form DIDs when using ION DID Method and which serviceEndpoints MUST be supported in the DID Document.

Implementations MUST support did:web, did:ion (long-form), and did:jwk as mandatory DID methods as defined in [[ref: did-web]], [[ref: did-ion]], and [[ref: did-jwk]].

### Cryptographic Signature

Issuers and the Wallets of this document MUST support JWT signature verification with the following Key Types and MUST support JWT signing with at least one.

|Key Type|JWT Algorithm|Support to validate the signature by the Credential Issuer and the Wallet|Support to sign by the Issuer|
|--------|-------------|-------------|-------------|
|secp256k1|ES256K|M|C*|
|Ed25519|EdDSA|M|C*|
|P-256 (secp256r1)|ES256|M|C*|
|P-384 (secp384r1)|ES384|M|C*|

`C*` Conditional: MUST support at least one.
`M` Must: MUST support.

Note: This profile leverages JWT for signature generation and verification only.

## Security Considerations

### Using of `public_client_id` as a `client_id`

By publishing `public_client_id` values in its metadata, the Credential Issuer is enabling any Wallet to use that public `public_client_id` value.

Authorization Servers should be aware of the risk of open redirects that allow Wallets to use the same `public_client_id` as a `client_id`.

## Use-Cases

Defining or referring to previously published use cases which this profile enables and supports is highly recommended.

## Credential type `VerifiedEmployee`

For a description of a `credentialSubject` for a credential type `VerifiedEmployee`, see [Credential type
VerifiedEmployee](https://identity.foundation/jwt-vc-presentation-profile/#credential-type-verifiedemployee) Section in the [[ref: JWT VC Presentation Profile]].

## Use-Cases

For a description of a Workplace credential use-case, see [Workplace Credential](https://identity.foundation/jwt-vc-presentation-profile/#workplace-credential) Section in [[ref: JWT VC Presentation Profile]].

## Examples

Embedded or referenced non-normative examples

## Implementations

- Mattr
- Ping Identity
- Gen Digital
- Spruce

## Test Vectors

Embedded or referenced test vectors.

## Test Suite

Embedded or referenced test suites.

## References

### Normative References

[[def: OpenID4VCI]]
~ [OpenID Connect for Verifiable Credentials Issuance](https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0-11.html). Torsten Lodderstedt, Kristina Yasuda, Tobias Looker. 2023.02. Status: Standards Track.

[[def: OIDC]]
~ [Open ID Connect](https://openid.net/specs/openid-connect-core-1_0.html). Nat Sakimura, John Bradley, Michael B. Jones, Breno de Medeiros, Chuck Mortimore. 2014.11. Status: Approved Specification.

[[def: VC Data Model v1.1]]
~ [Verifiable Credentials Data Model v1.1](https://www.w3.org/TR/vc-data-model/). Manu Sporny, Dave Longley, David Chadwick. 2021.08. Status: W3C Recommendation.

[[def: DID Core]]
~ [Decentralized Identifiers (DIDs) v1.0](https://www.w3.org/TR/2021/PR-did-core-20210803/). Manu Sporny, Dave Longley, Markus Sabadello, Drummond Reed, Orie Steele, Christopher Allen. 2021.08. Status: W3C Recommendation.

[[def: did-web]]
~ [Web DID Method](https://github.com/w3c-ccg/did-method-web). Oliver Terbu, Mike Xu, Dmitri Zagidulin, Amy Guy. Status: Registered in DID Specification Registry.

[[def: did-ion]]
~ [ION DID Method](https://github.com/decentralized-identity/ion-did-method). Various DIF contributors. Status: Registered in DID Specification Registry.

[[def: did-jwk]]
~ [JWT DID Method](https://github.com/quartzjer/did-jwk). Jeremie Miller, David Waite, Orie Steele. Status: Registered in DID Specification Registry.

[[def: Sidetree]]
~ [Sidetree v1.0.0](https://identity.foundation/sidetree/spec/). Daniel Buchner, Orie Steele, Troy Ronda. 2021.03. Status: DIF Ratified Specification.

[[def: Well Known DID]]
~ [Well Known DID Configuration](https://identity.foundation/.well-known/resources/did-configuration/). Daniel Buchner, Orie Steele, Tobias Looker. 2021.01. Status: DIF Working Group Approved Draft.

[[def: Identity Hub (0.0.1 Predraft)]]
~ [Identity Hub - Decentralized Web Node 0.0.1 Predraft](https://identity.foundation/decentralized-web-node/spec/0.0.1-predraft/). Daniel Buchner, Tobias Looker. Status: Draft.

[[def: Status List 2021]]
~ [Status List 2021](https://w3c.github.io/vc-status-list-2021/). Manu Sporny, Dave Longley, Orie Steele, Mike Prorock, Mahmoud Alkhraishi. 2022.04. Status: Draft Community Group Report.

[[def: JWT VC Presentation Profile]]
~ [JWT VC Presentation Profile](https://identity.foundation/jwt-vc-presentation-profile/). Daniel McGrogan, Kristina Yasuda, Jen Schreiber. 2022.10. Status: Draft.

[[def: RFC 6749]]
~ [The OAuth 2.0 Authorization Framework](https://www.rfc-editor.org/rfc/rfc6749.html). Dick Hardt, Ed. 2012.10. Status: Proposed Standard.

[[def: RFC 8414]]
~ [OAuth 2.0 Authorization Server Metadata](https://www.rfc-editor.org/rfc/rfc8414.html). Mike Jones, Nat Sakimura, John Bradley. 2018.06. Status: Proposed Standard.

[[def: OAuth 2.0 Security Best Current Practice]]
~[OAuth 2.0 Security Best Current Practice](https://www.ietf.org/archive/id/draft-ietf-oauth-security-topics-21.txt). Torsten Lodderstedt, John Bradley, Andrey Labunets, Daniel Fett. 2022.09. Status: Internet-Draft

### Non-Normative References

[[def: JWP]]
~ [JSON Web Proof](https://github.com/json-web-proofs/json-web-proofs/blob/main/draft-jmiller-json-web-proof.md). Jeremie Miller, David Waite, Michael B. Jones. Status: Internet-Draft.

[[def: JPA]]
~ [JSON Proof Algorithms](https://github.com/json-web-proofs/json-web-proofs/blob/main/draft-jmiller-json-proof-algorithms.md) Jeremie Miller, Michael B. Jones. Status: Internet-Draft.

[[def: SD-JWT]]
~ [JSON Web Proof](hhttps://datatracker.ietf.org/doc/html/draft-fett-selective-disclosure-jwt). Daniel Fett, Kristina Yasuda. Status: Internet-Draft.
