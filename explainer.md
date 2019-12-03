# Presentation Request Standardization

## Motivation
At IIW19 there were multiple sessions around data minimization when sharing Verifiable Credentials (VCs) between a Credential Holder and a Credential Verifier.

- On Tuesday, Workday presented their solutions to selectively share individual claims (as a substructure of the credentialSubject object) by additionally providing JSON-LD Signatures for every claim in addition to the overall credential.
- On Wednesday, as a result of the vivid discussion the day before, Nathan George, myself and representatives of Workday did a generalized discussion around the design of Presentation Request Protocol and the implications for VCs and associated schema / overlay definitions.

## Requirements
There exists a general consenus in the community that the information content transferred between a Credential Holder and a Credential Verifier should be kept to a minimal in order to satisfy the (regulatory) requirements of the Verifier. Regulation like GDPR and the California Consumer Privacy Act further support this requirement.

In order to archieve that goal, the credential holder must be able to construct Verifiable Presentations with a minimal amount of information that is decoupled from the personal information contained inside issued Verifiable Credentials.

Furthermore, Credential Verifiers must understand the taxonomy of supported credentials **and** their queryable substructures and/or capabilities (like Zero Knowledge Proofs, ZKP).

## Issuer vs. Verifier Interests

As discussed, Verifiers are interested in data minimalization, however, this is not necessarily true for the Issuer - Subject relationship. The Credential Issuer does not have an interest in separating issued credentials into minimal substructures (to support the Holder-Verifier Requirements), but rather wants to logically group Verifiable Information together into a single "issue event".

This allows the Credential Issuer to put logical restrictions on the joint information (e.g. like forcing the Holder to only present certain information in conjunction).

## Presentation Requests Capabilities

Generally, a Presentation Request should support the following Use-Cases (from the view of the) Verifier.

1. Allow to query "CredentialItems" which could be one of the following:
    - "Complete" Verifiable Credential (e.g. JSON-LD signed credential)
    - Selective Substructures fo a Verifiable Credentials ("Claims") that are idenpendently verifiable. "Selective Disclosure"
    - Other proofing technologies to convey information to the Verifier (e.g. ZKP)
2. Add Constraints to individual CredentialItems that the Holder can test client-side
    - Example A: "CredentialItem Date of Birth. Constraint. GT 05/20/84"
    - Important: Constraints can only be added to Information that is ALSO shared.

## Technical Considerations

### "Solve (Sovrin) / Resolve (Identity.com)"
The Holder needs to be able to match a given Presentation Request to their existing database of issued Verifiable Credentials.

- (Re)solve can be by
- Fully Satisfied
- Partially Satisfied
- Not Satisfied

If (Re)solve is not able to satisfy the Request fully, the ecosystem may allow the Credential Subject to discover appropiate Issuers.



