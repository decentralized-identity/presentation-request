# Addition of Presentation Request/Response to the W3C Verifiable Credentials specification
- Authors:
  - Daniel McGrogan [daniel.mcgrogan@workday.com](mailto:daniel.mcgrogan@workday.com)
  - Jonathan Reynolds [jonathan.reynolds@workday.com](mailto:jonathan.reynolds@workday.com)
- Last updated: 2019-12-18
## Status
- Status: **PROPOSAL**
- Status Date: 2019-12-18
- Status Note: Draft proposal for initial feedback

## Abstract
The [W3C Verifiable Credentials (hereafter VC) specification](https://www.w3.org/TR/vc-data-model) does not currently outline how credential data should be requested by a Verifier. This document outlines the approach taken at Workday and proposes it as an addition to the VC spec.

This document deals only with format of Presentation messages. Presentation protocol should be decoupled from its format and, as such, is dealt with in a separate document.

## Contents
  * [Proposal](#proposal)
    + [Introduction](#introduction)
    + [Selective Disclosure](#selective-disclosure-via-individually-signed-attributes)
    + [Self Attestation](#self-attestation)
    + [Zero Knowledge Proofs](#zero-knowledge-proofs)
    + [JSON and JSON-LD](#json-and-json-ld)
    + [Presentation Request](#presentation-request)
      - [Presentation Request Message Structure](#presentation-request-message-structure)
        * [JSON-LD](#json-ld)
        * [JSON](#json)
      - [Presentation Request Message Example](#presentation-request-message-example)
        * [JSON-LD](#json-ld-1)
        * [JSON](#json-1)
      - [Presentation Request JSON Schema](#presentation-request-json-schema)
      - [Criteria Message Structure](#criteria-message-structure)
        * [JSON-LD](#json-ld-2)
        * [JSON](#json-2)
      - [Criteria Message Example](#criteria-message-example)
        * [JSON-LD](#json-ld-3)
        * [JSON](#json-3)
      - [Criteria JSON Schema](#criteria-json-schema)
    + [Presentation Response](#presentation-response)
      - [Presentation Response Message Structure](#presentation-response-message-structure)
        * [JSON](#json-4)
      - [Presentation Response Message Example](#presentation-response-message-example)
      - [Presentation Response JSON Schema](#presentation-response-json-schema)
  * [Drawbacks/Limitations](#drawbackslimitations)
  * [Alternatives](#alternatives)
  * [Prior Art](#prior-art)
  * [Unresolved Questions](#unresolved-questions)
  * [Suggested Next Steps](#suggested-next-steps)
  * [Privacy Considerations](#privacy-considerations)
  * [References](#references)
  * [Glossary](#glossary)

## Proposal

### Introduction
It is the opinion of the authors that in order to have an interoperable VC flow the question of how to request Credential data must be addressed. We believe that leaving the solution out of the standards will result in competing formats which will reduce interoperability.

A Presentation Request is here defined as a document produced by a Verifier which lists the data criteria for a verification event. Similarly, Presentation Response is the proposed format with which to respond to a Presentation Request. By having Presentation Request as part of the VC specification, [software agents](#glossary) can have a common understanding of the data being requested.

It is expected that by using a Presentation Request, a Verifier can request multiple pieces of data from a Holder such that each:
- Is held by the Holder in a Verifiable Credential or self attested
- Is described in a known schema or other linked data
- Facilitates selective disclosure
    - Can be a single attribute from the known schema
    - Can be all data from a schema
- Is from one of a supplied list of valid Issuers

The concept of Presentation Request already defined as part of [Indy/Sovrin](https://github.com/sovrin-foundation/protocol/blob/master/themis/proof-request.md) as follows:

```The message sent by the relying party to the holder describing the verifiable attributes and appropriate conditions (predicates, issuer of attributes, schema of the credentials used, etc) that the holder need to satisfy.```

This proposal adjusts the definition slightly to move away from the concept of exclusively sending a message. Instead the Presentation Request can be sent or otherwise resolved and includes an optional Presentation URL to which a presentation response can be sent.

### Selective Disclosure via Individually Signed Attributes
Selective disclosure of attributes from a VC is desirable and should be supported by implementations of this Presentation Request specification. The W3C VC specification lists CL Signatures as the means to achieve selective disclosure. 

For solutions which do not use Zero Knowledge Proofs, this document proposes that individually signing attributes on a VC is a valid alternative for selective disclosure. This solution has the advantage of simpler crypto which anyone can reason about. In addition the approach of individually signing attributes allows an issuer to fully create a credential without requiring an additional issuer/holder interaction as required when using a blinded linked/master secret.

A separate proposal to update the W3C VC spec to support individually signed attributes on a credential will also be submitted.

### Self Attestation

Some use cases require a user to send some information which they self attest. The [Credential Manifest](https://github.com/decentralized-identity/credential-manifest/blob/master/explainer.md) project in DIF is one such example.

To specify that self attestation is valid, add a keyword to the Issuer array for a given claim:
* `SELF`
* `did:self`

### Zero Knowledge Proofs
This document does not currently address Zero Knowledge Proofs. A later version of this document will describe how a Verifier requests a predicate proof.

### JSON and JSON-LD
Both JSON and JSON-LD are listed in the VC specification as valid formats for Credentials. This implies that a Presentation Request must:
- Support JSON and JSON-LD formats
- Allow a Credential of either format to be included as a criterion

### Presentation Request

In order to allow both JSON and JSON-LD credentials to be requested and to support JSON and JSON-LD messages, the following structure sections will deal with Criteria separately from the main Presentation Request message.

#### Presentation Request Message Structure

* `id`: Id of the presentation request 
* `PresentationURL`: URL to send presentation response to (optional since protocol specific)
* PresentationRequest
  * `Accept`: A pair list of syntax types and proof types the verifier will process
  * `Criteria`: Array of requested [Criteria](#criteria-message-structure)
  * `Description`: Presentation description
  * `Verifier`: DID of Verifier
* Proof
  * `Created`: Created date
  * `Creator`: Verifier key reference
  * Nonce
  * `Signature Value`: signature string
  * `Type`: Signature Type

#### Presentation Request Message Example

##### JSON-LD
<details>
  <summary>Show/Hide Example</summary>

```
{
  "@context": [
    "https://www.w3.org/2018/credentials/v1"
  ],
  "id": "eedeec15-c364-4e4c-b7a7-871413e412f9",
  "@type": ["PresentationRequest"],
  "presentationURL": "https://example.com/v1/presentation-response",
  "presentationRequest": {
    "accept": ["json/RsaSignature2018", "json-ld/CLSignature2019"]
    "criteria": [...See Criteria section],
    "description": "Credit card application information",
    "verifier": "did:work:WrgHL2icHVN8vwZtY9YmjY"
  },
  "proof": {
    "type": "RsaSignature2018",
    "created": "2018-06-17T10:03:48Z",
    "presentationPurpose": "assertionMethod",
    "verificationMethod": "https://example.edu/issuers/14/keys/234",
    "jws": "pY9...Cky6Ed = "
  }
}
```
</details>

##### JSON
<details>
  <summary>Show/Hide Example</summary>

```
{
  "id": "eedeec15-c364-4e4c-b7a7-871413e412f9",
  "presentationURL": "https://example.com/v1/presentation-response",
  "presentationRequest": {
    "criteria": [...See Criteria section],
    "description": "Credit card application information",
    "verifier": "did:work:WrgHL2icHVN8vwZtY9YmjY"
  },
  "proof": {
    "created": "2019-04-01T11:38:12Z",
    "creator": "key-1",
    "nonce": "70b5a966-e818-44a5-8d0f-3873bb10980a",
    "signatureValue": "3FQAREE3D9YCgHjqj54gQ9BZnRhkCur1tTUoovBUuFRhD34mkusNLRuwi4QVRkwc7cEiichmHx7U2957iK4BgVss",
    "type": "Ed25519VerificationKey2018"
  }
}
```
</details>

#### Presentation Request JSON Schema
<details>
  <summary>Show/Hide Example</summary>

```
{
  "$schema": "http://json-schema.org/draft-04/schema#",
  "type": "object",
  "properties": {
    "id": {
      "type": "string"
    },
    "presentationURL": {
      "type": "string"
    },
    "presentationRequest": {
      "type": "object",
      "properties": {
        "criteria": {//see below
        },
        "description": {
          "type": "string"
        },
        "verifier": {
          "type": "string"
        }
      },
      "required": [
        "criteria",
        "description",
        "verifier"
      ]
    },
    "proof": {
      "type": "object",
      "properties": {
        "created": {
          "type": "string"
        },
        "creator": {
          "type": "string"
        },
        "nonce": {
          "type": "string"
        },
        "signatureValue": {
          "type": "string"
        },
        "type": {
          "type": "string"
        }
      },
      "required": [
        "created",
        "creator",
        "nonce",
        "signatureValue",
        "type"
      ]
    }
  },
  "required": [
    "id",
    "presentationURL",
    "presentationRequest",
    "proof"
  ]
}
```
</details>

#### Criteria Message Structure

* `Description`: Criterion description
* Issuers
    * `Dids`: Array of accepted Issuer DIDs
* `Max`: (Optional) Maximum number of credentials required (e.g. 6 months worth of Payslips)
* `Min`: (Optional) Minimum number of credentials required
* `Reason`: The reason that this data has been requested
* Schema
    * `Id`: DID of Schema 
    * Attributes
      * `Name`: name of attribute
      * `Required`: boolean. Is this attribute required
* Subject
    * `Context` establish the terms used in the type
    * `Type` the type of credential requested
    * Attributes
      * `Name`: name of attribute
      * `Required`: boolean. Is this attribute required

#### Criteria Message Example

##### JSON, JSON-LD
The following is an example with 4 criteria:
- An email address conforming to a schema specified by DID from one of three specified issuers
- First Name, Last Name, Date of Birth as defined in `https://stateagency.gov/DrivingLicenceCredential` from a single specified issuer 
- Address information conforming to a schema specified by DID from one of three specified issuers
- Six payslips conforming to a schema specified by DID from one of three specified issuers

<details>
  <summary>Show/Hide Example</summary>

```json
    "criteria": [{
        "description": "Contact Information",
        "issuers": {
          "dids": [
            "did:work:Tn1XD2QPskjDCztVGuJC2w",
            "did:work:M1dAMR7zmHD18wELKQTcTU",
            "did:work:7SWNtygraxEPqNKhuWpw8f"
          ]
        },
        "max": 1,
        "min": 1,
        "reason": "Send information regarding your application",
        "schema": "did:work:D7jC1h6LibDbZiF6E9uzhz;schema=1234;version=1.0", [schema is used when the properties are described on a schema]
        "properties": [{
          "name": "emailAddress",
          "required": true
        }]
      },
      {
        "description": "Date of Birth",
        "issuers": {
          "dids": [
            "did:work:2LwMkQadbzQVTGAqUBqWUV"
          ]
        },
        "max": 1,
        "min": 1,
        "reason": "Age Eligibility",
        "@context": "https://stateagency.gov", [context and type are used when the properties are described via JSON-LD]
        "@type": "DrivingLicenceCredential",
        "properties": [{
            "name": "FirstName",
            "required": true
          },
          {
            "name": "LastName",
            "required": true
          },
          {
            "name": "Date Of Birth",
            "required": true
          }
        ]
      },
      {
        "description": "Billing Address",
        "issuers": {
          "dids": [
            "did:work:PZ5vbLVYjC7c6vjC8w7UM",
            "did:work:W4Qi2D1DpBZig513ztvCFC",
            "did:work:7SWNtygraxEPqNKhuWpw8f"
          ]
        },
        "max": 1,
        "min": 1,
        "reason": "Regulations require a billing address",
        "schema": "did:work:3CRANqJJNw2uU3ZxyBBSwX;schema=1333;version=1.0",
        "properties": [{
            "name": "city",
            "required": true
          },
          {
            "name": "country",
            "required": true
          },
          {
            "name": "postalCode",
            "required": true
          },
          {
            "name": "street1",
            "required": true
          },
          {
            "name": "street2",
            "required": false
          }
        ]
      },

      {
        "description": "6 Months of payslips",
        "issuers": {
          "dids": [
            "did:work:EgLY8mysgbLmj542ygMKWo"
          ]
        },
        "max": 6,
        "min": 6,
        "reason": "Payslips are used to determine credit worthiness",
        "schema": "did:work:KypRcDbLYAUwaA7nPuP3Tf;schema=1333;version=1.0",
        "properties": [{
            "name": "currency",
            "required": true
          },
          {
            "name": "grossPay",
            "required": true
          },
          {
            "name": "payPeriodEnd",
            "required": true
          },
          {
            "name": "payPeriodStart",
            "required": true
          }
        ]
      }
    ]
```
</details>


#### Criteria JSON Schema
Note that we enforce `schema` or `@context` and `@type`

<details>
  <summary>Show/Hide Example</summary>

```
"criteria": {
  "type": "array",
  "items": 
    {
      "type": "object",
      "properties": {
        "description": {
          "type": "string"
        },
        "issuers": {
          "type": "object",
          "properties": {
            "dids": {
              "type": "array",
              "items": [
                {
                  "type": "string"
                },
                {
                  "type": "string"
                },
                {
                  "type": "string"
                }
              ]
            }
          },
          "required": [
            "dids"
          ]
        },      
        "schema": {
         "type": "string"
        },      
        "@context" : {
            "type" : "string"
        },
        "@type" : {
            "type" : "string"
        },
        "max": {
          "type": "integer"
        },
        "min": {
          "type": "integer"
        },
        "reason": {
          "type": "string"
        },
        "properties": {
          "type": "array",
          "items": [
            {
              "type": "object",
              "properties": {
                "name": {
                  "type": "string"
                },
                "required": {
                  "type": "boolean"
                }
              },
              "required": [
                "name",
                "required"
              ]
            }
          ]
        }
      },
      "not" : {
        "anyOf": [{"required": ["@context","schema"]},{"required": ["type","schema"]}]
      },
      "anyOf":[
        { "required": ["description","schema","issuers","max","min","reason","properties"]},
        { "required": ["description","@context", "@type","issuers","max","min","reason","properties"]}
      ]
    }
  
}
```
</details>


### Presentation Response

#### Presentation Response Message Structure

##### JSON

+ FulfilledCriteria
  - `Criterion`: See [above](#criteria-message-structure)
  - `Presentations`: W3C Verifiable Presentations for the requested Criteria
+ Proof
+ `PresentationRequestId`: Id to link the Presentation Response to the Presentation Request to aid Verifier correlation 

#### Presentation Response Message Examples

##### Example with single Criterion and Presentation

<details>
  <summary>Show/Hide Example</summary>

```
{
  "id": "672841fc-10a6-4c13-b3e0-349da08ae9f3",
  "presentationRequestId": "752e4391-2111-4ab6-b174-67fd88d51e9f",
  "FulfilledCriteria": [
      {
          "Criterion": {
              "description": "Email",
              "issuers": {
                  "dids": [
                      "did:work:RWmzj4ftrGrjwMgNPhYRQ2"
                  ]
              },
              "max": 1,
              "min": 1,
              "reason": "Send information regarding your application",
              "schema": {
                  "attributes": [
                      {
                          "name": "emailAddress",
                          "required": true
                      }
                  ],
                  "id": "did:work:DkAENQnV4998K7CDkZAbHC;schema=1333;version=1.0"
              }
          },
          "Presentations": [
              {
                  "@context": [
                      "https://w3.org/2018/credentials/v1"
                  ],
                  "@type": [
                      "VerifiablePresentation"
                  ],
                  "created": "2019-08-23T14:29:48Z",
                  "id": "b9cd5414-7c36-4c59-91f3-11a8bbca6b9e",
                  "proof": [
                      {
                          "created": "2019-08-23T14:29:48Z",
                          "creator": "did:work:93kmikE6QnaaNSpA3Fp8Sm#key-1",
                          "nonce": "8ac448d4-6977-447c-9e58-b24ba2a2b07e",
                          "signatureValue": "2qGNdUuEGPQZnajxSCm3wFedRcq47Bzf2dq8WJskYnz5LS9iTVMPcXasXrDB7pHdjAe4wqBV2vKxbi3Gt4wFRHAc",
                          "type": "Ed25519VerificationKey2018"
                      }
                  ],
                  "verifiableCredential": [
                      {
                          "@context": [
                              "https://w3.org/2018/credentials/v1"
                          ],
                          "@type": [
                              "VerifiableCredential"
                          ],
                          "credentialSubjects": [
                              {
                                  "id": "did:work:847yjhrkgjferkrgebsd",
                                  "key": "emailAddress",
                                  "proof": [
                                      {
                                          "created": "2019-08-23T14:27:45Z",
                                          "creator": "did:work:RWmzj4ftrGrjwMgNPhYRQ2#key-1",
                                          "nonce": "47103c6b-34e2-4567-b896-dedab2cf3865",
                                          "signatureValue": "YwU389NKemM2p8nzEj7wYyVgANcTse2RUTnPbMtbj2kHZqXDEfnfdM2Q76BqYJAk1HS95DYWaPebbKr4zzgvYVu",
                                          "type": "Ed25519VerificationKey2018"
                                      }
                                  ],
                                  "value": "noreply@workday.com"
                              }
                          ],
                          "id": "6adb968b-52df-4da0-a720-4a0a6ac8f8db",
                          "issuanceDate": "2019-08-23T14:27:45Z",
                          "issuerDid": "did:work:RWmzj4ftrGrjwMgNPhYRQ2",
                          "schemaId": "did:work:DkAENQnV4998K7CDkZAbHC;schema=1333;version=1.0"
                      }
                  ]
              }
          ]
      }
  ],
  "proof": [
      {
          "created": "2019-08-23T14:29:48Z",
          "creator": "did:work:93kmikE6QnaaNSpA3Fp8Sm#key-1",
          "nonce": "d9e9dc30-e284-41d5-84ff-9d39d8e96b6c",
          "signatureValue": "s9diCs3k52eRut4J3id4ZW1KwKLBcGNDFtArZHmW3MNBL1XKariPwYjtmoMjmnoekdfTzLYg22iDAuHcf8B3WGy",
          "type": "Ed25519VerificationKey2018"
      }
  ]
}
```
</details>

##### Longer Example

<details>
  <summary>Show/Hide Example</summary>

```
{
	"id": "672841fc-10a6-4c13-b3e0-349da08ae9f3",
	"presentationRequestId": "752e4391-2111-4ab6-b174-67fd88d51e9f",
    "FulfilledCriteria": [
        {
            "Criterion": {
                "description": "Email",
                "issuers": {
                    "dids": [
                        "did:work:RWmzj4ftrGrjwMgNPhYRQ2"
                    ]
                },
                "max": 1,
                "min": 1,
                "reason": "Send information regarding your application",
                "schema": {
                    "attributes": [
                        {
                            "name": "emailAddress",
                            "required": true
                        }
                    ],
                    "id": "did:work:DkAENQnV4998K7CDkZAbHC;spec:b70f9e7e-bd6f-429e-8e08-250d0c4da909"
                }
            },
            "Presentations": [
                {
                    "@context": [
                        "https://w3.org/2018/credentials/v1"
                    ],
                    "created": "2019-08-23T14:29:48Z",
                    "id": "b9cd5414-7c36-4c59-91f3-11a8bbca6b9e",
                    "proof": [
                        {
                            "created": "2019-08-23T14:29:48Z",
                            "creator": "did:work:93kmikE6QnaaNSpA3Fp8Sm#key-1",
                            "nonce": "8ac448d4-6977-447c-9e58-b24ba2a2b07e",
                            "signatureValue": "2qGNdUuEGPQZnajxSCm3wFedRcq47Bzf2dq8WJskYnz5LS9iTVMPcXasXrDB7pHdjAe4wqBV2vKxbi3Gt4wFRHAc",
                            "type": "Ed25519VerificationKey2018"
                        }
                    ],
                    "@type": [
                        "VerifiablePresentation"
                    ],
                    "verifiableCredential": [
                        {
                            "@context": [
                                "https://w3.org/2018/credentials/v1"
                            ],
                            "credentialSubjects": [
                                {
                                    "id": "did:work:847yjhrkgjferkrgebsd",
                                    "key": "emailAddress",
                                    "proof": [
                                        {
                                            "created": "2019-08-23T14:27:45Z",
                                            "creator": "did:work:RWmzj4ftrGrjwMgNPhYRQ2#key-1",
                                            "nonce": "47103c6b-34e2-4567-b896-dedab2cf3865",
                                            "signatureValue": "YwU389NKemM2p8nzEj7wYyVgANcTse2RUTnPbMtbj2kHZqXDEfnfdM2Q76BqYJAk1HS95DYWaPebbKr4zzgvYVu",
                                            "type": "Ed25519VerificationKey2018"
                                        }
                                    ],
                                    "value": "noreply@workday.com"
                                }
                            ],
                            "id": "6adb968b-52df-4da0-a720-4a0a6ac8f8db",
                            "issuanceDate": "2019-08-23T14:27:45Z",
                            "issuerDid": "did:work:RWmzj4ftrGrjwMgNPhYRQ2",
                            "schemaId": "did:work:DkAENQnV4998K7CDkZAbHC;schema=1333;version=1.0",
                            "@type": [
                                "VerifiableCredential"
                            ]
                        }
                    ]
                }
            ]
        },
        {
            "Criterion": {
                "description": "Billing Address",
                "issuers": {
                    "dids": [
                        "did:work:KBBj3fsCwYLbPfmiKJgkYx"
                    ]
                },
                "max": 1,
                "min": 1,
                "reason": "Regulations require a billing address",
                "schema": {
                    "attributes": [
                        {
                            "name": "city",
                            "required": true
                        },
                        {
                            "name": "country",
                            "required": true
                        },
                        {
                            "name": "postalCode",
                            "required": true
                        },
                        {
                            "name": "street1",
                            "required": true
                        },
                        {
                            "name": "street2",
                            "required": false
                        }
                    ],
                    "id": "did:work:DkAENQnV4998K7CDkZAbHC;schema=1333;version=1.0"
                }
            },
            "Presentations": [
                {
                    "@context": [
                        "https://w3.org/2018/credentials/v1"
                    ],
                    "created": "2019-08-23T14:29:48Z",
                    "id": "1f92319e-57aa-48e8-8395-e1ed166e84b1",
                    "proof": [
                        {
                            "created": "2019-08-23T14:29:48Z",
                            "creator": "did:work:93kmikE6QnaaNSpA3Fp8Sm#key-1",
                            "nonce": "34c90fcb-eb20-4e59-9aff-cd697c080453",
                            "signatureValue": "41W5pMaXiihim3esbtLgvYQTVnBtFhRYquHqMqm5csdWLstmXPfM4q1npwnESgBHsVVqWH2LfDZySthPQEAkdYqX",
                            "type": "Ed25519VerificationKey2018"
                        }
                    ],
                    "@type": [
                        "VerifiablePresentation"
                    ],
                    "verifiableCredential": [
                        {
                            "@context": [
                                "https://w3.org/2018/credentials/v1"
                            ],
                            "credentialSubjects": [
                                {
                                    "id": "did:work:847yjhrkgjferkrgebsd",
                                    "key": "city",
                                    "proof": [
                                        {
                                            "created": "2019-08-23T14:27:54Z",
                                            "creator": "did:work:KBBj3fsCwYLbPfmiKJgkYx#key-1",
                                            "nonce": "527cdb66-186c-49c6-8149-859ece265c00",
                                            "signatureValue": "3CR4EmY4GAgkEsXyscRnvVHnJLyymFAHBdYCQgiYobbEhHXnrpwhVUT9Pfp5QG6dV1GesPYzrHBpBWPfj1LACGHd",
                                            "type": "Ed25519VerificationKey2018"
                                        }
                                    ],
                                    "value": "Pleasanton"
                                },
                                {
                                    "id": "did:work:847yjhrkgjferkrgebsd",
                                    "key": "country",
                                    "proof": [
                                        {
                                            "created": "2019-08-23T14:27:54Z",
                                            "creator": "did:work:KBBj3fsCwYLbPfmiKJgkYx#key-1",
                                            "nonce": "f7190bdd-12f4-43d0-a34e-3954964c2724",
                                            "signatureValue": "rEtk8Dk1YaTH5RCvmg5mATPqyrn5wwT2AEdPMzquHrDpAoyXdUajNzjSS14yhD3LQGZz6QRqLZLznU7rwTfFwRq",
                                            "type": "Ed25519VerificationKey2018"
                                        }
                                    ],
                                    "value": "US"
                                },
                                {
                                    "id": "did:work:847yjhrkgjferkrgebsd",
                                    "key": "postalCode",
                                    "proof": [
                                        {
                                            "created": "2019-08-23T14:27:54Z",
                                            "creator": "did:work:KBBj3fsCwYLbPfmiKJgkYx#key-1",
                                            "nonce": "103c85c8-1aaa-4521-98ae-1b1b3403bc2e",
                                            "signatureValue": "2x57F9nAFNcpB6NV6PxUAkxzJkWUkbZptMR5sehuMBAjZo4cWT2Skoz9DUHhCYb27AU2fmRYWDxC1n6MQ6vzWu8i",
                                            "type": "Ed25519VerificationKey2018"
                                        }
                                    ],
                                    "value": "94588"
                                },
                                {
                                    "id": "did:work:847yjhrkgjferkrgebsd",
                                    "key": "street1",
                                    "proof": [
                                        {
                                            "created": "2019-08-23T14:27:54Z",
                                            "creator": "did:work:KBBj3fsCwYLbPfmiKJgkYx#key-1",
                                            "nonce": "ade3be00-1831-467d-8b1e-ca27650c1371",
                                            "signatureValue": "nc81qwL56Ua5YkvdUP58nNE58cEphmZtAkQeuTCRXXsGhPDDgcNtuYSzJuyBpRntmy3pwkP9HZMz584yzmWSbfq",
                                            "type": "Ed25519VerificationKey2018"
                                        }
                                    ],
                                    "value": "5928 Stoneridge Mall Rd"
                                }
                            ],
                            "id": "4dbc6ddd-d0ee-4d40-9efc-d51ac2eb9550",
                            "issuanceDate": "2019-08-23T14:27:54Z",
                            "issuerDid": "did:work:KBBj3fsCwYLbPfmiKJgkYx",
                            "schemaId": "did:work:DkAENQnV4998K7CDkZAbHC;schema=1333;version=1.0",
                            "@type": [
                                "VerifiableCredential"
                            ]
                        }
                    ]
                }
            ]
        },
        {
            "Criterion": {
                "description": "Payslips",
                "issuers": {
                    "dids": [
                        "did:work:YXMykiTuPju9nEZDxvUKx6"
                    ]
                },
                "max": 2,
                "min": 1,
                "reason": "Payslips are used to determine credit worthiness",
                "schema": {
                    "attributes": [
                        {
                            "name": "currency",
                            "required": true
                        },
                        {
                            "name": "grossPay",
                            "required": true
                        },
                        {
                            "name": "payPeriodEnd",
                            "required": true
                        },
                        {
                            "name": "payPeriodStart",
                            "required": true
                        }
                    ],
                    "id": "did:work:TGribEAw1pEJxr4agrBPrE;schema=1333;version=1.0"
                }
            },
            "Presentations": [
                {
                    "@context": [
                        "https://w3.org/2018/credentials/v1"
                    ],
                    "created": "2019-08-23T14:29:48Z",
                    "id": "b3c648af-69a5-4073-afe1-ee2d8fed2a56",
                    "proof": [
                        {
                            "created": "2019-08-23T14:29:48Z",
                            "creator": "did:work:93kmikE6QnaaNSpA3Fp8Sm#key-1",
                            "nonce": "944665e8-f353-40f1-840e-76721e09cb34",
                            "signatureValue": "2GKCVRtcHgRKvZ5dVHr81E5XvgWdRb6FZ3LWfw5zAdPKAs15Z83FSmMN1sDJJQoZo9QCbNY3oGaFeNDkVdZqJra8",
                            "type": "Ed25519VerificationKey2018"
                        }
                    ],
                    "@type": [
                        "VerifiablePresentation"
                    ],
                    "verifiableCredential": [
                        {
                            "@context": [
                                "https://w3.org/2018/credentials/v1"
                            ],
                            "credentialSubjects": [
                                {
                                    "id": "did:work:847yjhrkgjferkrgebsd",
                                    "key": "currency",
                                    "proof": [
                                        {
                                            "created": "2019-08-23T14:28:03Z",
                                            "creator": "did:work:YXMykiTuPju9nEZDxvUKx6#key-1",
                                            "nonce": "11fd9bad-f5f6-41f9-8c33-ee37948d4697",
                                            "signatureValue": "54WmKb13krcbW5fSofZ38D6oCZxdgwUEcBNGC38suLJqhpEh8pfP9A457HkR48iEWX5goB7eyRgMqUet72KFCKQw",
                                            "type": "Ed25519VerificationKey2018"
                                        }
                                    ],
                                    "value": "USD"
                                },
                                {
                                    "id": "did:work:847yjhrkgjferkrgebsd",
                                    "key": "grossPay",
                                    "proof": [
                                        {
                                            "created": "2019-08-23T14:28:03Z",
                                            "creator": "did:work:YXMykiTuPju9nEZDxvUKx6#key-1",
                                            "nonce": "ae744339-eb71-49fe-a365-16bcc8d81d21",
                                            "signatureValue": "2EKUjp1c9zTHoCrMyyVuEjXC5XFmxpDBtjFwfN6tjDzbNgj67Taj5fKuUpZoJghpS41d4THpB2aiveT98dsX7uUA",
                                            "type": "Ed25519VerificationKey2018"
                                        }
                                    ],
                                    "value": "5000"
                                },
                                {
                                    "id": "did:work:847yjhrkgjferkrgebsd",
                                    "key": "payPeriodEnd",
                                    "proof": [
                                        {
                                            "created": "2019-08-23T14:28:03Z",
                                            "creator": "did:work:YXMykiTuPju9nEZDxvUKx6#key-1",
                                            "nonce": "2ff537b7-6901-43a3-9964-bf7bc10b6282",
                                            "signatureValue": "54Y8TgTh1TTjR5WmsrrnQyf236cYAssQEyuMaWPB7qmCt4y7keFuTb2d61AEf7m61HZzfoBEiwTwWkyhMECGQh8b",
                                            "type": "Ed25519VerificationKey2018"
                                        }
                                    ],
                                    "value": "2006-01-02T15:04:05+04:00"
                                },
                                {
                                    "id": "did:work:847yjhrkgjferkrgebsd",
                                    "key": "payPeriodStart",
                                    "proof": [
                                        {
                                            "created": "2019-08-23T14:28:03Z",
                                            "creator": "did:work:YXMykiTuPju9nEZDxvUKx6#key-1",
                                            "nonce": "75f5beab-9e50-4881-9c46-2a8e2565a08e",
                                            "signatureValue": "62zXHZpSnEXaXudJesZn5hEWth5WUTWuvFJnuvkPSKRCZokmXLXx191xCbbpLC8846JDqYva2wK3PckJQHbwnvsR",
                                            "type": "Ed25519VerificationKey2018"
                                        }
                                    ],
                                    "value": "2006-01-02T15:04:05+04:00"
                                }
                            ],
                            "id": "12c21009-7c13-42da-9cb8-01ea805ed324",
                            "issuanceDate": "2019-08-23T14:28:03Z",
                            "issuerDid": "did:work:YXMykiTuPju9nEZDxvUKx6",
                            "schemaId": "did:work:TGribEAw1pEJxr4agrBPrE;schema=1333;version=1.0",
                            "type": [
                                "VerifiableCredential"
                            ]
                        }
                    ]
                },
                {
                    "@context": [
                        "https://w3.org/2018/credentials/v1"
                    ],
                    "created": "2019-08-23T14:29:48Z",
                    "id": "63ccf1f3-82ba-4ffa-a7ec-b7ee659e8243",
                    "proof": [
                        {
                            "created": "2019-08-23T14:29:48Z",
                            "creator": "did:work:93kmikE6QnaaNSpA3Fp8Sm#key-1",
                            "nonce": "11aa2b4a-ed2a-404b-9215-30d1dafdaee1",
                            "signatureValue": "3A31QNyopJLSbowgAih2KVPHznEeNvr6iCkrUqnHnvYgGSU5UQ4XE8xEzrX8bee2TgvJYG39v9Rcnw7aAHszgNfQ",
                            "type": "Ed25519VerificationKey2018"
                        }
                    ],
                    "type": [
                        "VerifiablePresentation"
                    ],
                    "verifiableCredential": [
                        {
                            "@context": [
                                "https://w3.org/2018/credentials/v1"
                            ],
                            "credentialSubjects": [
                                {
                                    "id": "did:work:847yjhrkgjferkrgebsd",
                                    "key": "currency",
                                    "proof": [
                                        {
                                            "created": "2019-08-23T14:28:03Z",
                                            "creator": "did:work:YXMykiTuPju9nEZDxvUKx6#key-1",
                                            "nonce": "11fd9bad-f5f6-41f9-8c33-ee37948d4697",
                                            "signatureValue": "54WmKb13krcbW5fSofZ38D6oCZxdgwUEcBNGC38suLJqhpEh8pfP9A457HkR48iEWX5goB7eyRgMqUet72KFCKQw",
                                            "type": "Ed25519VerificationKey2018"
                                        }
                                    ],
                                    "value": "USD"
                                },
                                {
                                    "id": "did:work:847yjhrkgjferkrgebsd",
                                    "key": "grossPay",
                                    "proof": [
                                        {
                                            "created": "2019-08-23T14:28:03Z",
                                            "creator": "did:work:YXMykiTuPju9nEZDxvUKx6#key-1",
                                            "nonce": "ae744339-eb71-49fe-a365-16bcc8d81d21",
                                            "signatureValue": "2EKUjp1c9zTHoCrMyyVuEjXC5XFmxpDBtjFwfN6tjDzbNgj67Taj5fKuUpZoJghpS41d4THpB2aiveT98dsX7uUA",
                                            "type": "Ed25519VerificationKey2018"
                                        }
                                    ],
                                    "value": "5000"
                                },
                                {
                                    "id": "did:work:847yjhrkgjferkrgebsd",
                                    "key": "payPeriodEnd",
                                    "proof": [
                                        {
                                            "created": "2019-08-23T14:28:03Z",
                                            "creator": "did:work:YXMykiTuPju9nEZDxvUKx6#key-1",
                                            "nonce": "2ff537b7-6901-43a3-9964-bf7bc10b6282",
                                            "signatureValue": "54Y8TgTh1TTjR5WmsrrnQyf236cYAssQEyuMaWPB7qmCt4y7keFuTb2d61AEf7m61HZzfoBEiwTwWkyhMECGQh8b",
                                            "type": "Ed25519VerificationKey2018"
                                        }
                                    ],
                                    "value": "2006-01-02T15:04:05+04:00"
                                },
                                {
                                    "id": "did:work:847yjhrkgjferkrgebsd",
                                    "key": "payPeriodStart",
                                    "proof": [
                                        {
                                            "created": "2019-08-23T14:28:03Z",
                                            "creator": "did:work:YXMykiTuPju9nEZDxvUKx6#key-1",
                                            "nonce": "75f5beab-9e50-4881-9c46-2a8e2565a08e",
                                            "signatureValue": "62zXHZpSnEXaXudJesZn5hEWth5WUTWuvFJnuvkPSKRCZokmXLXx191xCbbpLC8846JDqYva2wK3PckJQHbwnvsR",
                                            "type": "Ed25519VerificationKey2018"
                                        }
                                    ],
                                    "value": "2006-01-02T15:04:05+04:00"
                                }
                            ],
                            "id": "12c21009-7c13-42da-9cb8-01ea805ed324",
                            "issuanceDate": "2019-08-23T14:28:03Z",
                            "issuerDid": "did:work:YXMykiTuPju9nEZDxvUKx6",
                            "schemaId": "did:work:TGribEAw1pEJxr4agrBPrE;schema=1333;version=1.0",
                            "type": [
                                "VerifiableCredential"
                            ]
                        }
                    ]
                }
            ]
        }
    ],
    "proof": [
        {
            "created": "2019-08-23T14:29:48Z",
            "creator": "did:work:93kmikE6QnaaNSpA3Fp8Sm#key-1",
            "nonce": "d9e9dc30-e284-41d5-84ff-9d39d8e96b6c",
            "signatureValue": "s9diCs3k52eRut4J3id4ZW1KwKLBcGNDFtArZHmW3MNBL1XKariPwYjtmoMjmnoekdfTzLYg22iDAuHcf8B3WGy",
            "type": "Ed25519VerificationKey2018"
        }
    ]
}
```

</details>


#### Presentation Response JSON Schema
<details>
  <summary>Show/Hide Example</summary>

```
{
  "$schema": "http://json-schema.org/draft-04/schema#",
  "type": "object",
  "properties": {
    "id": {
      "type": "string"
    },
    "presentationRequestId": {
      "type": "string"
    },
    "FulfilledCriteria": {
      "type": "array",
      "items": [
        {
          "type": "object",
          "properties": {
            "Criterion": {...See Criteria section},
            "Presentations": {
              "type": "array",
              "items": [
                {
                  "type": "object",
                  "properties": {
                    "@context": {
                      "type": "array",
                      "items": [
                        {
                          "type": "string"
                        }
                      ]
                    },
                    "created": {
                      "type": "string"
                    },
                    "id": {
                      "type": "string"
                    },
                    "proof": {
                      "type": "array",
                      "items": [
                        {
                          "type": "object",
                          "properties": {
                            "created": {
                              "type": "string"
                            },
                            "creator": {
                              "type": "string"
                            },
                            "nonce": {
                              "type": "string"
                            },
                            "signatureValue": {
                              "type": "string"
                            },
                            "type": {
                              "type": "string"
                            }
                          },
                          "required": [
                            "created",
                            "creator",
                            "nonce",
                            "signatureValue",
                            "type"
                          ]
                        }
                      ]
                    },
                    "type": {
                      "type": "array",
                      "items": [
                        {
                          "type": "string"
                        }
                      ]
                    },
                    "verifiableCredential": {
                      "type": "array",
                      "items": [
                        {
                          "type": "object",
                          "properties": {
                            "@context": {
                              "type": "array",
                              "items": [
                                {
                                  "type": "string"
                                }
                              ]
                            },
                            "credentialSubjects": {
                              "type": "array",
                              "items": [
                                {
                                  "type": "object",
                                  "properties": {
                                    "id": {
                                      "type": "string"
                                    },
                                    "key": {
                                      "type": "string"
                                    },
                                    "proof": {
                                      "type": "array",
                                      "items": [
                                        {
                                          "type": "object",
                                          "properties": {
                                            "created": {
                                              "type": "string"
                                            },
                                            "creator": {
                                              "type": "string"
                                            },
                                            "nonce": {
                                              "type": "string"
                                            },
                                            "signatureValue": {
                                              "type": "string"
                                            },
                                            "type": {
                                              "type": "string"
                                            }
                                          },
                                          "required": [
                                            "created",
                                            "creator",
                                            "nonce",
                                            "signatureValue",
                                            "type"
                                          ]
                                        }
                                      ]
                                    },
                                    "value": {
                                      "type": "string"
                                    }
                                  },
                                  "required": [
                                    "id",
                                    "key",
                                    "proof",
                                    "value"
                                  ]
                                }
                              ]
                            },
                            "id": {
                              "type": "string"
                            },
                            "issuanceDate": {
                              "type": "string"
                            },
                            "issuerDid": {
                              "type": "string"
                            },
                            "schemaId": {
                              "type": "string"
                            },
                            "type": {
                              "type": "array",
                              "items": [
                                {
                                  "type": "string"
                                }
                              ]
                            }
                          },
                          "required": [
                            "@context",
                            "credentialSubjects",
                            "id",
                            "issuanceDate",
                            "issuerDid",
                            "schemaId",
                            "type"
                          ]
                        }
                      ]
                    }
                  },
                  "required": [
                    "@context",
                    "created",
                    "id",
                    "proof",
                    "type",
                    "verifiableCredential"
                  ]
                }
              ]
            }
          },
          "required": [
            "Criterion",
            "Presentations"
          ]
        }
      ]
    },
    "proof": {
      "type": "array",
      "items": [
        {
          "type": "object",
          "properties": {
            "created": {
              "type": "string"
            },
            "creator": {
              "type": "string"
            },
            "nonce": {
              "type": "string"
            },
            "signatureValue": {
              "type": "string"
            },
            "type": {
              "type": "string"
            }
          },
          "required": [
            "created",
            "creator",
            "nonce",
            "signatureValue",
            "type"
          ]
        }
      ]
    }
  },
  "required": [
    "id",
    "presentationRequestId",
    "FulfilledCriteria",
    "proof"
  ]
}
```
</details>

##### JSON-LD
The JSON-LD example differs only in the addition of context and type
```
{
  "@context":"https://www.w3.org/2018/credentials/v1"
  "@type":["PresentationResponse"]
  "FulfilledCriteria": [
    {
      "Criterion": {....
```

## Drawbacks/Limitations
- Verbose
- Issuers is a limited subset of DIDs. We should also allow for a Presentation from all members of `Issuer trusted by X`. 

## Alternatives
- Bespoke exchange mechanism per implementation between clients and agents
- Semantic query language which addresses a common way to query across a credential graph. A separate proposal will outline this.

## Prior Art
- As mentioned above, Proof Requests are a concept which is already present in Hyperledger Indy and therefore Sovrin which has a number of live implementations.
- Hyperledger Aries is [proposing Proof Request negotation protocol](https://github.com/hyperledger/aries-rfcs/tree/master/features/0037-present-proof)
- A more complex negotiation protocol is proposed as [W3C Credential Handler API](https://w3c-ccg.github.io/credential-handler-api) and covers some similar ground to this proposal.

## Unresolved Questions
- We are aware of the projects mentioned in above in Prior Art. Of these, we know that Sovrin is using Proof Requests in live systems. We do not know of other formats currently being used by live or nearly live implementations of VC.

## Suggested Next Steps 
- Specifying issuer identity through semantic assertion on that identity
- Request zero knowledge proof and derived credential as a value response

## Privacy Considerations
- Lack of ZKP predicate requests diminish privacy for certain use cases. For many of the initial uses in enterprise anonymity is not possible given that attributes need to be shared directly to satisfy use-cases. Since it is the intention to add ZKP predicate requests to this specification there is no medium term issue here.

## References
- W3C Verifiable Credentials Specification: https://www.w3.org/TR/vc-data-model
- Sovrin defintion of Proof Request: https://github.com/sovrin-foundation/protocol/blob/master/themis/proof-request.md
- Proof Request in Indy/Sovrin: https://github.com/sovrin-foundation/protocol/blob/master/themis/proof-request.md
- Hyperledger Aries Proposal: https://github.com/hyperledger/aries-rfcs/tree/master/features/0037-present-proof
- W3C Credential Handler API: https://w3c-ccg.github.io/credential-handler-api

## Glossary
- `Agent`: Software used by or acting on behalf of an identity in a VC flow.