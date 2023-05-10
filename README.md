# .bit DID Method Specification

## Status

⚠️  This document is a work in progress draft

## Summary

[.bit](https://did.id) is a [blockchain-based](https://www.nervos.org/), open source, decentralized cross-chain account system that provides a worldwide unique naming system with a .bit suffix that can be used in different scenarios, such as cryptocurrency transfer, domain name resolution, identity authentication, etc.

This .bit DID method specification describes a new DID method and how to do CRUD operations on .bit DID documents.

This specification conforms to the requirements specified in the [DIDs specification](https://www.w3.org/TR/did-core/) currently published by the W3C Credentials Community Group.

## DID Method

The name string that shall identify this DID method is: `bit`.

A DID that uses this method MUST begin with the following prefix: `did:bit`. Per the DID specification, this string MUST be in lowercase. The remainder of the DID, after the prefix, is specified below.

## DID Method-Specific Identifier

.bit DID Method-Specific Identifier have the following format:

```
DID Method-Specific Identifier := [:<network>]:<name>
network                        := mainnet | pudge | ...
name                           := <.bit-name>
```

If the network is omitted, the network defaults to mainnet, so a `did:bit:example.bit` is equivalent to `did:bit:mainnet:example.bit`. However, the canonical form is `did:bit:mainnet:example.bit`.

### Examples

Example with full .bit DID format:

```
- did:bit:example.bit
- did:bit:my.example.bit
- did:bit:mainnet:my.example.bit
- ...
```

## CRUD Operations

.bit names can have CUSTOM records. This specification defines CUSTOM record names that will have an impact on DID resolution of .bit DIDs.

The following named CUSTOM records are defined for now:

- `org_w3c_did_service`

  OPTIONAL. A set of [services](https://www.w3.org/TR/did-core/#services) as per W3C DID Core specification. The service `id` property MAY be omitted. In that case the DID resolver will generate a canonical value for the specific service entry.

  > NOTE: the .bit Service will be automatically propagated as a service during DID resolution.

- `org_w3c_did_verificationMethod`
  
  OPTIONAL. A set of [verification methods](https://www.w3.org/TR/did-core/#verification-methods) as per W3C DID Core specification. Verification method `id` property values MUST be relative DID URIs. 
  
  > NOTE: the owner of the .bit name will be automatically propagated as a verification method during DID resolution.

- `org_w3c_did_verificationRelationship`
  
  OPTIONAL. A map of [verification relationship](https://www.w3.org/TR/did-core/#verification-relationships) as per W3C DID Core specification to a set of verification relationship identifiers (`id` property).
  
  > NOTE: the verification method that relates to the owner of the .bit name can be used for all verification relationships.

### Create

See [.bit doc](https://docs.did.id/) or [.bit official website](https://did.id) on how to register .bit names.

### Read

DID resolution will perform .bit resolution for a given .bit name. Additionally, the DID resolver will then retrieve the DID specific CUSTOM records for the .bit name and add default values for service endpoints and verification methods.

The default verification method will always include the owner of the .bit name as follows:

```json
{
  "id": "<.bit-name>#<sha256-of-blockchainAccountId>",
  "type": "EcdsaSecp256k1RecoveryMethod2020",
  "controller": "<.bit-name>",
  "blockchainAccountId": "<algorithmIndex-payload>"
}
```

- The `id` of the default verification method is the concatenation of the .bit DID followed by the `#` and the hex representation of `sha256(blockchainAccountId)`. Additional verification methods that MAY be added MUST not use that verification method `id` and will be ignored in the DID Document.

- .bit's `blockchainAccountId` consists of two parts: `algorithmIndex` and `payload`. Now .bit supports the following algorithms:

| index | algorithm        | curve     | payload               | Scenario             |
| ----- | ---------------- | --------- | --------------------- | -------------------- |
| 03    | ECDSA            | secp256k1 | address               | EVM compatible chain |
| 04    | ECDSA            | secp256k1 | hex format on address | TRON                 |
| 05    | ECDSA ( [EIP712](https://eips.ethereum.org/EIPS/eip-712) ) | secp256k1 | address               | EVM compatible chain |
| 06    | EdDSA            | ed25519   | publick key           | Mixin                |
| 07    | ECDSA            | secp256k1 | hex format on address | DOGE                 |
| 08    | ECDSA            | secp256r1 | hash(cid)+hash(pk)    | WebAuthn             |

The default service will always include the .bit service as follows:

```json
{
  "id":"<DID-.bit>#Web3PublicProfile-<sha256-of-blockchainAccountId>",
  "type": "Web3PublicProfile", 
  "serviceEndpoint": { 
    "profileService": ".bit",
    "name": "<.bit-name>",
    "network": "mainnet" 
  }
}
```

The `id` of the default service is the concatenation of the .bit DID followed by the `#Web3PublicProfile-` and the hex representation of `sha256(blockchainAccountId)`. Additional services that MAY be added MUST not use that service `id` and will be ignored in the DID Document.

If the DID specific CUSTOM records are malformed, the entire CUSTOM record will be ignored in the DID resolution process.

#### Example (no CUSTOM records)

For `did:bit:example.bit` (with no CUSTOM records added), the DID Document would look as follows:

```json
{
  "@context": [
    "https://www.w3.org/ns/did/v1",
    "https://w3id.org/ens/v1",
    "https://w3id.org/casa/profile-services/v1"
  ],
  "id": "did:bit:example.bit",
  "canonicalId": "did:bit:mainnet:example.bit",
  "verificationMethod": [{
    "id": "did:bit:mainnet:example.bit#85c4952fd6252377167e5de56017724e2687fa153bbd4b53b10bb3cac482978b",
    "type": "EcdsaSecp256k1RecoveryMethod2020",
    "controller": "did:bit:mainnet:example.bit",
    "blockchainAccountId": "03ab16a96d359ec26a11e2c2b3d8f8b8942d5bfcdb"
  }],
  "service": [{
    "id":"did:bit:mainnet:example.bit#Web3PublicProfile-85c4952fd6252377167e5de56017724e2687fa153bbd4b53b10bb3cac482978b",
    "type": "Web3PublicProfile", 
    "serviceEndpoint": { 
      "profileService": ".bit",
      "name": "example.bit",
      "network": "mainnet" 
    }
  }],
  "authentication": [
    "did:bit:mainnet:example.bit#85c4952fd6252377167e5de56017724e2687fa153bbd4b53b10bb3cac482978b"
  ],
  "assertionMethod": [
    "did:bit:mainnet:example.bit#85c4952fd6252377167e5de56017724e2687fa153bbd4b53b10bb3cac482978b"
  ],
  "capabilityInvocation": [
    "did:bit:mainnet:example.bit#85c4952fd6252377167e5de56017724e2687fa153bbd4b53b10bb3cac482978b"
  ],
  "capabilityDelegation": [
    "did:bit:mainnet:example.bit#85c4952fd6252377167e5de56017724e2687fa153bbd4b53b10bb3cac482978b"
  ]
}
```

#### Example (with keyAgreement)

For `did:bit:example.bit` with DID specific CUSTOM records added, the DID Document would look as follows:

```json
{
  "@context": [
    "https://www.w3.org/ns/did/v1",
    "https://w3id.org/ens/v1",
    "https://w3id.org/casa/profile-services/v1"
  ],
  "id": "did:bit:example.bit",
  "canonicalId": "did:bit:mainnet:example.bit",
  "verificationMethod": [{
      "id": "did:bit:mainnet:example.bit#85c4952fd6252377167e5de56017724e2687fa153bbd4b53b10bb3cac482978b",
      "type": "EcdsaSecp256k1RecoveryMethod2020",
      "controller": "did:bit:mainnet:example.bit",
      "blockchainAccountId": "03ab16a96d359ec26a11e2c2b3d8f8b8942d5bfcdb"
    },
    {
      "id": "did:bit:mainnet:example.bit#zC9ByQ8aJs8vrNXyDhPHHNNMSHPcaSgNpjjsBYpMMjsTdS",
      "type": "X25519KeyAgreementKey2019", 
      "controller": "did:bit:mainnet:example.bit",
      "publicKeyMultibase": "z9hFgmPVfmBZwRvFEyniQDBkz9LmV7gDEqytWyGZLmDXE" 
    }
  ],
  "service": [{
    "id":"did:bit:mainnet:example.bit#Web3PublicProfile-85c4952fd6252377167e5de56017724e2687fa153bbd4b53b10bb3cac482978b",
    "type": "Web3PublicProfile", 
    "serviceEndpoint": { 
      "profileService": ".bit",
      "ensName": "example.bit",
      "network": "mainnet" 
    }
  }],
  "authentication": [
    "did:bit:mainnet:example.bit#85c4952fd6252377167e5de56017724e2687fa153bbd4b53b10bb3cac482978b"
  ],
  "assertionMethod": [
    "did:bit:mainnet:example.bit#85c4952fd6252377167e5de56017724e2687fa153bbd4b53b10bb3cac482978b"
  ],
  "capabilityInvocation": [
    "did:bit:mainnet:example.bit#85c4952fd6252377167e5de56017724e2687fa153bbd4b53b10bb3cac482978b"
  ],
  "capabilityDelegation": [
    "did:bit:mainnet:example.bit#85c4952fd6252377167e5de56017724e2687fa153bbd4b53b10bb3cac482978b"
  ],
  "keyAgreement": [
    "did:bit:mainnet:example.bit#zC9ByQ8aJs8vrNXyDhPHHNNMSHPcaSgNpjjsBYpMMjsTdS"  
  ]
}
```

The following CUSTOM records have to be set:

- `org_w3c_did_verificationRelationship`
- `org_w3c_did_verificationMethod`

### Update

See [related documents](https://docs.did.id/technical-details/data-container#record) on how to add/update CUSTOM records.

### Delete

When an account expires and is not renewed for a long time, it will be reclaimed by the system. Once it is reclaimed, the DID information is essentially deleted and will remain deleted until a new user registers it. More details see [related documents](https://docs.did.id/technical-details/lifecycle#account-logical-status) to know the lifecycle of a .bit name.

## Privacy Considerations

When any data (e.g. W3C Verifiable Credentials) is associated with .bit DIDs, sharing that data would also impose sharing the onchain data graph (e.g. transaction history, NFTs etc.) of the payload that owns the .bit name.

Using personal identifiable information as DID Method specific identifiers discloses personal information every time the DID is shared with a counter party. This specification DOES NOT endorse the use of .bit names that correlate directly with real world human beings.

## Security Considerations

.bit names are non-fungible and transferrable. When the owner of the .bit name changes, the authorative keys will also change. This needs to be considered when used in conjunction with verifiable data where the DID is embedded, e.g., W3C Verifiable Credentials.

