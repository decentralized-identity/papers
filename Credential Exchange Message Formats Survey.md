# Credentials / Claims Exchange message formats

**Table of Contents**

- [About this document/approach](#about-this-document-approach)
- [Categories](#categories)
- [Expressiveness/Features](#expressiveness-features)
- [Design Questions](#design-questions)
  * [Filters/Constraints](#filters-constraints)
  * [Should we consider Verifiable Credential Exchange a special case of data exchange?](#should-we-consider-verifiable-credential-exchange-a-special-case-of-data-exchange-)
  * [Vocabulary](#vocabulary)
- [Category 1 examples](#category-1-examples)
  * [uPort Verifiable Credential Request/Response](#uport-verifiable-credential-request-response)
  * [uPort Selective Disclosure Request/Response (used in DID Auth)](#uport-selective-disclosure-request-response--used-in-did-auth-)
  * [Jolocom Credential Request/Response](#jolocom-credential-request-response)
  * [Credential Handler API](#credential-handler-api)
  * [ID Hub Search and Read Requests](#id-hub-search-and-read-requests)
    + [Query Request/Response](#query-request-response)
    + [Read Request/Response](#read-request-response)
- [ID Hub Write (Store) Request/Response (TODO: Categorize)](#id-hub-write--store--request-response--todo--categorize-)
- [Sidetree VCs (TODO: categorize)](#sidetree-vcs--todo--categorize-)
- [HIPE (TODO: categorize)](#hipe--todo--categorize-)
- [Civic](#civic)
  * [Credential Commons](#credential-commons)
  * [DSR - Dynamic Scope Request](#dsr---dynamic-scope-request)
  * [ID Validator Toolkit (e.g. Issuer Toolkit)](#id-validator-toolkit--eg-issuer-toolkit-)
- [Questions/TODO](#questions-todo)
- [General References](#general-references)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>


## About this document/approach

Previously, the DIF Claims/Credentials Working Group proposed a general [Credentials Exchange Manifest](https://github.com/decentralized-identity/credential-manifest/blob/master/explainer.md); however, before going too far along that path, we wanted to better understand existing approaches. That will help ensure our format is sufficiently expressive and flexible, and meet our goals of promoting interop.

This document is simply an effort to gather "what's out there" so we can categorize and generalize. 

Currently, efforts like DID auth are having to focus on message formats in addition to protocols -- even message formats dealing with claims exchange (see uPort selective disclosure/DID auth). We'd also like to identify when this is happening so we can iterats on them in the Claims/Credentials group.

Because of that, we'll be reviewing existing claims exchange protocols/formats that member companies have documented, including where they appear as part of protocols like DID Auth. References are at the end.


## Categories

Based on review so far, it looks like we need interoperable message formats for Claims manifests/Claims (request/response) associated with:
1. Credential exchange 
    - Example: RP requesting VCs/VPs (or data?) from holder
    - In some cases, this may be part of a DID Auth flow
2. Subject/holder requesting claims issuance


## Expressiveness/Features
Still deciding scope for initial interop POC, but eventually design should accommodate all. 

Initial list:
- Selection of fields / entire claim
- Verifiable Presentation, or just VCs?
- ZKP (out of scope for initial phase)
- Filter to issuers
- Revocation actions
- Icons

## Design Questions

### Filters/Constraints

There will be a need for claims filters (by type, fields, etc), as shown in a couple of examples below, but we should be careful to not re-invent the wheel and create something restrictive. 

We could consider something LISP S-expressions. 
```
["AND",
    {"var1" : "value1"},
    ["OR",
        { "var2" : "value2" },
        { "var3" : "value3" }
    ]
]
```

A JSON library attempting something similar is  [JSONLogic](http://jsonlogic.com/)

**Note** Jolocom appears to use a similar style. DIF's previously-published Claim Metadata format looked home-grown...

### Should we consider Verifiable Credential Exchange a special case of data exchange?

Identity hub examples are included below, because there is a fuzzy area between requesting *unverified* claim data and other user data.

A possible approach is for claims exchange to be a special case of data exchange.

### Vocabulary

How do we let a requestor specify desired vocabulary/types? Current approaches seem ambiguous/confusing.

I think it is just a special case of Filters/Constraints

## Category 1 examples

Note that responses take the form of Verifiable Credentials/Presentations, but examples are included to the demonstrate different wrappers/envelopes

### uPort Verifiable Credential Request/Response

Format: JWT
Details: https://github.com/uport-project/specs/blob/develop/messages/verificationreq.md

About: The issuer is requesting the holder's wallet/agent to present a signed claim containing _exactly_ the field `name` and respond with a payload like the following

Example request:
```
{
  "type":"verReq",
  "iss":"did:uport:REQUESTING_APP_OR_USER",
  "aud":"did:uport:VERIFYING_APP_OR_USER",
  "sub":"did:uport:SUBJECT_OF_VERIFIED_CLAIM",
  "riss":"did:uport:IDENTITY_THAT_WILL_SIGN_THE_CLAIM",
  "unsignedClaim": {
    "name":"Bob Smith"
  },
  "callback":"https://example.com",
  "rexp": 123456789
}
```

Example response:

```
{
  "iss":"did:uport:VERIFYING_APP_OR_USER",
  "sub":"did:uport:SUBJECT_OF_VERIFIED_CLAIM",
  "claim": {
    "name":"Bob Smith"
  },
  "exp": 123456789
}
```


Observations:
- Potential for reuse:
    - `iss`, `iat`, `exp` are shared with Share Request, and _should_ be fairly standard across JWTs
- Concerns:
    - It's not clear what's happening if the holder has a claim containing a superset of the requested field. uPort mentions a best practice of narrow claims, but we can't carry this assumption
    - Asking for a field independent of types; is there an assumption of shared context? 

### uPort Selective Disclosure Request/Response (used in DID Auth)

**Note**: I would argue any claims-based aspects of DID auth belong with Claims/Credentials effort

Format: JWT
Details: 
- https://github.com/uport-project/specs/blob/develop/messages/sharereq.md
- https://github.com/uport-project/specs/blob/develop/messages/shareresp.md

```
{
  iss: 'did:web:somesite.com',
  type: 'shareReq',
  claims: {
    verifiable: {
      email: {
        iss: [
          {
            did: 'did:web:uport.claims',
            url: 'https://uport.claims/email'
          },
          {
            did: 'did:web:sobol.io',
            url: 'https://sobol.io/verify'
          }
        ],
        reason: 'Whe need to be able to email you'
      },
      nationalIdentity: {
        essential: true,
        iss: [
          {
            did: 'did:web:idverifier.claims',
            url: 'https://idverifier.example'
          }
        ],
        reason: 'To legally be able to open your account'
      }
    },
    user_info: {
      name: { essential: true, reason: "Show your name to other users"},
      country: null
    }
  }
}
```

Response example:
```
{
  "iss":"did:ethr:0x012abcd...",
  "vc": ["Verified Claim JWT 1", "Verified Claim JWT 2"]
}
```


Observations:
- Inconsistency with the above example; i.e. `claims` as requested claims vs `unsignedClaim` in previous example. Also they should allow similar options, like filtering by issuer
- `essential` -- is a boolean ok, or do we need multi-valued. If booleam rename to required? what should default be?
- `callback` not shown here but available as optional field, like previous example


### Jolocom Credential Request/Response


Format: JWT

Reference: https://jolocom-lib.readthedocs.io/en/latest/interactionFlows.html

Example request:

```
{
  "typ": "JWT",
  "alg": "ES256K"
}
{
  "iat": 1528997842275,
  "requestedCredentials": [
    {
      "type": ["Credential", "ProofOfEmailCredential"],
      "constraints": {
        "and": [
          { "==": [ true, true ] },
          { "!=": [ { "var": "issuer" }, { "var": "claim.id" } ] }
        ]
      }
    }
  ],
  "requesterIdentity": "did:jolo:b310d293aeac8a5ca680232b96901fe85988fde2860a1a5db69b49762923cc88",
  "callbackURL": "https://demo-sso.jolocom.com/proxy/authentication/aws6i"
}
```

Example response:
```
{
  '@context': [ ... ],
  ...
  issuer: 'did:jolo:b2d5d8d6cc140033419b54a237a5db51710439f9f462d1fc98f698eca7ce9777',
  claim: {
    email: 'example@example.com',
    id: 'did:jolo:6d6f636b207375626a656374206469646d6f636b207375626a65637420646964'
  },
  proof: EcdsaLinkedDataSignature {
    ...
    creator: 'did:jolo:b2d5d8d6cc140033419b54a237a5db51710439f9f462d1fc98f698eca7ce9777#keys-1'
    ...
}
```

Observations:
- can express constraints, logical operators
- In the request, `type` appears to be requested type; that's a little confusing.
- Response looks like a straightforward VC. It's out of date, but easily portable


### Credential Handler API

Format: JSON-LD

Details: https://w3c-ccg.github.io/credential-handler-api/

Credential Storage (Registration) and Request. Uses notion of credential "hints"
```
{
  {
    name: "My social account: pat@example.com",
    enabledTypes: ["VerifiableProfile"],
    icons: [{
      src: "icon/lowres.webp",
      sizes: "48x48",
      type: "image/webp"
    }],
    match: {
      VerifiableProfile: {
        id: 'did:method1:1234-1234-1234-1234'
      }
    }
  }
  ```
  
  ```
  {
        name: "My business account: pat@business.example.com",
        enabledTypes: ["VerifiableProfile"],
        match: {
          VerifiableProfile: {
            id: 'did:method1:1234-1234-1234-1235'
          }
        }
      }
```

About/Scope: imperative API enabling a website to request a user’s credentials from a user agent, and to help the user agent correctly store user credentials for future use. User agents implementing that API prompt the user to select a way to handle a credential request, after which the user agent returns a credential to the originating site. This specification defines capabilities that enable third-party Web applications to handle credential requests and storage.

### ID Hub Search and Read Requests

If you squint enough, VC exchange requests look like a special case of data exchange requests that occur, e.g. in Identity Hub operations

Identity Hub has 2 relevant requests (search and read). Consider whether there are opportunities to unify/align these requests and how these can be shared with credential exchange.



#### Query Request/Response

```
{
  "@context": "https://schema.identity.foundation/0.1",
  "@type": "ObjectQueryRequest",
  "iss": "did:foo:123abc",
  "sub": "did:bar:456def",
  "aud": "did:baz:789ghi",
  "query": {
      "interface": "Collections",
      "context": "http://schema.org",
      "type": "MusicPlaylist",
      
      // Optional object_id filters
      "object_id": ["3a9de008f526d239..", "a8f3e7..."]
  }
}
```
```
{
  "@context": "https://schema.identity.foundation/0.1",
  "@type": "ObjectQueryResponse",
  "developer_message": "completely optional",
  "objects": [
    {
      // object metadata
      "interface": "Collections",
      "context": "http://schema.org",
      "type": "MusicPlaylist",
      "id": "3a9de008f526d239...",
      "created_by": "did:foo:123abc",
      "created_at": "2018-10-24T18:39:10.10:00Z",
      "sub": "did:foo:123abc",
      "commit_strategy": "basic",
      "meta": {
        "tags": ["classic rock", "rock", "rock n roll"],
        "cache-intent": "full"
      }
    },
    // ...more objects
  ]

  // potential pagination token
  "skip_token": "ajfl43241nnn1p;u9390",
}
```

#### Read Request/Response

```
{
  "@context": "https://schema.identity.foundation/0.1",
  "@type": "CommitQueryRequest",
  "iss": "did:foo:123abc",
  "sub": "did:bar:456def",
  "aud": "did:baz:789ghi",
  "query": {
    "object_id": ["3a9de008f526d239..."],
    "revision": ["abc", "def", ...]
  },
}
```

```
{
  "@context": "https://schema.identity.foundation/0.1",
  "@type": "CommitQueryResponse",
  "developer_message": "completely optional",
  "commits": [
    {
      protected: "ewogICJpbnRlcmZhY2UiO...",
      header: {
        "iss": "did:foo:123abc",
        // Hubs may add additional information to the unprotected headers for convenience
        "rev": "aHashOfTheCommit",
      },
      payload: "ewogICJAY29udGV4dCI6ICdo...",
      signature: "b7V2UpDPytr-kMnM_YjiQ3E0J2..."
    },
    // ...
  ],

  // potential pagination token
  "skip_token": "ajfl43241nnn1p;u9390",
}
```

TODO: See also Profiles, and more in ID Hub docs




## ID Hub Write (Store) Request/Response (TODO: Categorize)

```
// JWT headers
{
  "alg": "RS256",
  "kid": "did:foo:123abc#key-abc",
  "interface": "Collections",
  "context": "https://schema.org",
  "type": "MusicPlaylist",
  "operation": "create",
  "committed_at": "2018-10-24T18:39:10.10:00Z",
  "commit_strategy": "basic",
  "sub": "did:bar:456def",

  // Example metadata about the object that is intended to be "public"
  "meta": {
    "tags": ["classic rock", "rock", "rock n roll"],
    "cache-intent": "full"
  }
}

// JWT body
{
  "@context": "http://schema.org/",
  "@type": "MusicPlaylist",
  "description": "The best rock of the 60s, 70s, and 80s",
  "tracks": ["..."],
}
```

```
{
  "@context": "https://schema.identity.foundation/0.1",
  "@type": "WriteResponse",
  "developer_message": "completely optional message from the hub",
  "revisions": ["aHashOfTheCommitSubmitted"]
}
```



## Sidetree VCs (TODO: categorize)

Details: https://hackmd.io/tx8Z0mIRS-aK84Gx4xIzfg?view


- Create operation: payload is a Verifiable Credential
- Update operations: expressed as JSON Patch operations on the VC

## HIPE (TODO: categorize)
https://github.com/hyperledger/indy-hipe/blob/c761c583b1e01c1e9d3ceda2b03b35336fdc8cc1/text/anoncreds-protocol/README.md

## Civic
Civic utilized draft specifications of DIDs and Verifiable Credentials in their product pipeline. Civic Technologies makes the open-source components of their implementation accessible to interested parties with an MIT License via identity.com. Generally the following Libraries exists:

### Credential Commons
URL: https://github.com/identity-com/credential-commons
This Javascript Library provides functionality around Verifiable Credentials (VC), a W3C standard. Enables Validators to issue, Credential Wallets to verify, filter and Requesters to verify credentials.

Civic does not provide a Proof implementation for the issued Verifiable Credentials, but abstracts that via an interface implementation. E.g. interested parties are able to use Credential-Commons and provide their own implementation around JWT token proofs or JSON-LD Proofs. Internally Civic uses a proprietary implementation of VC Proofs declared via
```
...
  "proof": {
    "type": "CivicMerkleProof2018",
...
```
Credential Commons furthermore defines a global definitions file for claims and credentials that can be referenced in Presentation Requests or Credential Manifests. (https://github.com/identity-com/credential-commons/blob/master/src/claim/definitions.js)

### DSR - Dynamic Scope Request


### ID Validator Toolkit (e.g. Issuer Toolkit)
URL: This component is not public yet.

The IDV Toolkit is a modular framework that allows an issuer of Verifiable Credential to host an endpoint to validate an users input and issue verifiable credentials to an interested party.
At Identity.com potential issuers are discovered via the Identity.com Marketplace, but this is not a prerequisite for hosting and issuing credentials. The discovery of Issuers and their respective Toolkits can be agnostic from the Identity.com Marketplace. 

The IDV Validator Toolkit announces requirements for issuing an Verifiable Credential to a user via a Verification Process that looks similar to this:

```
{
  "id": "3e51e979-bcdd-4eda-ba78-ee8fbccb6ace",
  "processUrl": "/processes/3e51e979-bcdd-4eda-ba78-ee8fbccb6ace",
  "state": {
    "status": "COMPLETE",
    "ucaVersion": "1",
    "credential": "credential-cvc:IdDocument-v1",
    "ucas": {
      "email": {
        "name": "cvc:Type:email",
        "status": "ACCEPTED",
        "value": {
          "username": "martin-test",
          "domain": {
            "tld": "com",
            "name": "civic"
          }
        },
        "retriesRemaining": 0,
        "suggestedIndex": 0
      },
      "document": {
        "name": "cvc:Document:flow",
        "parameters": {
          "applicantId": "XXX",
          "secrets": {
            "apiKey": "XXX"
          },
          "front": "{\"Bucket\":\"ugimages-dev\",\"ContentMD5\":\"3765ac6d6fb8a2edfdec6863de4552b2\",\"Key\":\"docs/2019/5/2/45bd574b-088e-4d87-bab3-85a1f96b6ba0.jpg\"}"
        },
        "status": "ACCEPTED",
        "value": "upload",
        "retriesRemaining": 0,
        "suggestedIndex": 1
      },
      ...
```

In a Validation Process an ID Validator expresses the requirements to an external entity by exposing UCA "User Collectible Attributes" that are defined in an Identity.com specific Library. Entities can submit these UCA (e.g. an E-Mail, a document, a phone number, a validation token) in order to validate information dynamically with the ID Validator.

After a successful Validation process the ID Validator can issue the requested Credential to the User.

Entity Authentication is done via JWT HTTP Bearer Tokens of the respective DID Keys.







## Questions/TODO

- From DIF meeting: revocation in scope? 
- VC Statuses https://w3c.github.io/vc-data-model/#status
- Credential Disputes: https://w3c.github.io/vc-data-model/#disputes
- "Refresh" Service: https://w3c.github.io/vc-data-model/#refreshing
- Filter by terms-of-use?
- Also see these, and consider framing categories in light of these: https://w3c.github.io/vc-data-model/#subject-holder-relationships

## General References
- [DID Auth](https://github.com/WebOfTrustInfo/rwot6-santabarbara/blob/master/final-documents/did-auth.md)
- [SSI Layers and Implementations](https://docs.google.com/spreadsheets/d/1IZjsriCsSgERIg29sR-NEQhHNT2n1Td3I3YsuHtVUHo/edit#gid=1201785801)
