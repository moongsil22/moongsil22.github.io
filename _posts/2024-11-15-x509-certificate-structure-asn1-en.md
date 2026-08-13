---
layout: post
tags: [posts-en]
title: Taking Apart an X.509 Certificate (ASN.1 DER Analysis)
description: >
  Notes on X.509 v3 fields and ASN.1 DER encoding from a graduate-school Modern Cryptography assignment that analyzed a public-key certificate byte by byte. (Values adapted for illustration.)
---

While working with the OAM (Oracle Access Manager) authentication/authorization system, I realized I'd never actually broken a certificate down field by field to see what's really inside it. A graduate school assignment for my Modern Cryptography course happened to require analyzing a personal public-key certificate down to the byte level, so I've written up the notes here. (The values below have been adapted into illustrative examples rather than showing an actual certificate.)

## 1. What Is a Certificate?

A certificate bundles "someone's public key" together with a signature that says "a CA vouches this public key really belongs to that person." X.509 is the standard that defines that structure, and the actual byte encoding follows the ASN.1 DER rules.

## 2. Subject Public Key Info: RSA Public Key Structure

An RSA public key ultimately comes down to two numbers: the **modulus (n)** and the **public exponent (e)**.

- **Modulus (n)**: the product of two primes p and q (`n = p × q`). This value determines the key length (e.g., 2048 bits).
- **Public exponent (e)**: conventionally fixed at 65537 (`0x10001`), chosen by convention for a balance of computational efficiency and security.

Encryption and signature verification both boil down to modular exponentiation:

- Encryption: `C = M^e mod n`
- Signature verification: `M = S^e mod n`

### What It Looks Like Encoded in ASN.1 DER

```
30 82 01 0a          SEQUENCE, length 266 bytes   ← wraps the whole public key
  02 82 01 01         INTEGER, length 257 bytes   ← modulus (n)
    00 xx xx ... xx    (the modulus value; a leading 00 is prepended if the top byte is 0x80 or higher, so it isn't misread as negative)
  02 03               INTEGER, length 3 bytes     ← public exponent (e)
    01 00 01           e = 65537
```

`30` is the ASN.1 tag for SEQUENCE, and `02` is the tag for INTEGER. These two fields (a modulus and an exponent wrapped in a SEQUENCE) are the entirety of an RSA public key — it looks complicated at first glance, but underneath it's just "two integers packed together in order."

## 3. Key X.509 v3 Fields

| Field | Example value | Description |
|---|---|---|
| Version | V3 | The version that supports X509v3 extension fields. Most certificates today are V3. |
| Serial Number | `01A2B3C4` | A unique identifier assigned by the CA at issuance. Used to point to a specific certificate when it's revoked. |
| Signature Algorithm | sha256WithRSAEncryption | The hash + signature algorithm combination the CA used to sign the certificate. |
| Issuer | CN=ExampleCA, O=ExampleCorp, C=KR | Identifies the CA that issued this certificate. |
| Validity | 2024-XX-XX – 2025-XX-XX | The validity period. Falling outside this range means immediate rejection during verification. |
| Subject | CN=John Doe, O=ExampleOrg, C=KR | Identifies the certificate's owner. For TLS, this gets compared against the server name. |
| Subject Public Key Info | RSA 2048 bit | The modulus/exponent structure broken down above. |

## 4. Extension Fields

The extensions supported starting with V3 are, in practice, the more interesting part.

- **Key Usage**: restricts what this certificate can be used for. For example, `Digital Signature, Non Repudiation` limits it to generating/verifying digital signatures and can enforce that it not be used for other purposes like key exchange. If it's flagged `critical`, this restriction must be honored.
- **Certificate Policies**: specifies which issuance policy (OID) this certificate follows and where its CPS (Certificate Practice Statement) document lives. Systems use this as a basis for deciding whether the certificate meets a policy level they're willing to trust.
- **Subject Alternative Name (SAN)**: carries additional identifiers that a single Subject CN can't express — domains, emails, or custom OID values defined by a country-specific standard. In Korea's accredited-certificate system, for instance, a hash used for real-name verification is sometimes embedded as an "Other Name" inside the SAN.
- **CRL Distribution Point**: the location of the CRL (Certificate Revocation List) where you can check whether this certificate has been revoked, typically given as an LDAP or HTTP URL.
- **Authority Info Access (AIA)**: the address of an OCSP (Online Certificate Status Protocol) server. Instead of downloading the entire CRL, you can ask this URL in real time, "is this certificate valid right now?"

## 5. Fingerprint and Signature Value

- **Fingerprint**: a hash-algorithm summary of the certificate's entire data. Since even a tiny tamper changes the fingerprint, browsers and systems use it to quickly check "is this the same certificate I already know about?"
- **Signature Value**: the value the CA produced by signing the certificate's contents with its own private key. A verifier decrypts this signature with the CA's public key and checks it against the hash of the certificate's contents, which is how it confirms "was this certificate really issued by that CA, and has it been left untampered with since?"

## Wrap-Up

Taking a certificate apart field by field made it much clearer why certain values matter in the OAM/mTLS configuration work I do day to day. In particular, extensions like Key Usage and SAN — which I'd normally just skim past — turned out to be core pieces of information that actually determine how far a given certificate should be trusted and what it's restricted to being used for.
