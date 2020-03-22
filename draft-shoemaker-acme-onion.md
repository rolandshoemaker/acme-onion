%%%
title = "Automated Certificate Management Environment (ACME) Onion v3 Identifier Validation Extension"
ipr= "trust200902"
area = "General"
workgroup = "ACME Working Group"
submissiontype = "IETF"
keyword = []

[seriesInfo]
status = "standard"
name = "Internet-Draft"
value = "draft-shoemaker-acme-onion-00"
stream = "IETF"

[[author]]
initials = "R.B."
surname = "Shoemaker"
fullname = "Roland Bracewell Shoemaker"
organization = "Internet Security Research Group"
abbrev = "ISRG"
  [author.address]
  email = "roland@letsencrypt.org"
%%%

.# Abstract

This document specifies identifiers and challenges required to enable the Automated Certificate Management Environment (ACME) to issue certificates for Onion Addresses as specified in Tor Rendezvous Specification - Version 3.

{mainmatter}

# Introduction

Currently the Automated Certificate Management Environment (ACME) [@RFC8555] only specifies how DNS identifiers and IP address identifiers [@RFC8738] may be validated for inclusion in x.509 certificates [@RFC5280]. This document extends the protocol to include a validation mechanism for Tor version 3 Onion Addresses[@TOR-REND-V3].

# Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 [@RFC2119] [@RFC8174] when, and only when, they appear in all capitals, as shown here.

# Onion Address Identifier

Version 3 Onion address identifier objects MUST use the type "onion-v3". The value field of the identifier MUST contain the textual encoding of the address as defined as onion_address in [@TOR-REND-V3] section 6. ACME servers MUST verify that the value field contains a properly encoded address by checking that it contains only two labels, that first label contains a valid checksum, and that the last byte of the first label is \x03. [TODO: this could probably be specified better?]

An identifier for the version 3 Onion address address for the following Ed25519 public key would be formatted like so:

~~~~~~~~~~
-----BEGIN PUBLIC KEY-----
MCowBQYDK2VwAyEAAJhLnbvXNWu8WXre3Y0HU+1FErU13zcbO7pEqkI38+Q=
-----END PUBLIC KEY-----
~~~~~~~~~~

~~~~~~~~~~
{"type": "onion-v3", "value": "acmexhn3242wxpczplpn3dihkpwukevvgxptogz3xjckuqrx6pscepad.onion"}
~~~~~~~~~~

# Onion CSR Challenge

This document specifies a single new challenge type that can be used for validation of onion-v3 identifiers. This challenge demonstrates control of the private key associated with the public key contained within the Onion address by signing a CSR containing special attributes.

type (required, string):
: The string "onion-v3-csr"

nonce (required, string):
: A random value that uniquely identifies the challenge. This value MUST have at least 64 bits of entropy. It MUST NOT contain any characters outside the base64url alphabet as described in [@RFC4648] Section 5. Trailing '=' padding characters MUST be stripped. See [@!RFC4086] for additional information on randomness requirements.

The client prepares for validation by constructing a Certificate Signing Request (CSR) [@RFC2986]. This CSR MUST contain two attributes, a caSigningNonce attribute containing the nonce provided in the challenge object, and a applicantSigningNonce attribute containing a random value picked by the client. This random value MUST have at least 64 bits of entropy. The CSR MUST be signed by the private key associated with the public key contained within the Onion address.

The caSigningNonce and applicantSigningNonce attributes are defined as follows in [@CABF-BR] Appendix F

```
cabf-caSigningNonce OBJECT IDENTIFIER ::= { cabf 41 }

caSigningNonce ATTRIBUTE ::= {
    WITH SYNTAX             OCTET STRING
    EQUALITY MATCHING RULE  octetStringMatch
    SINGLE VALUE            TRUE
    ID                      { cabf-caSigningNonce }
}

cabf-applicantSigningNonce OBJECT IDENTIFIER ::= { cabf 42 }

applicantSigningNonce ATTRIBUTE ::= {
    WITH SYNTAX             OCTET STRING
    EQUALITY MATCHING RULE  octetStringMatch
    SINGLE VALUE            TRUE
    ID                      { cabf-applicantSigningNonce }
}
```

# IANA Considerations

## Identifier Types

Adds a new type to the "ACME Identifier Types" registry defined in Section 9.7.7 of [@RFC8555] with Label "onion-v3" and Reference "I-D.shoemaker-acme-onion".

## Challenge Types

Adds one new entry to the "ACME Validation Methods" registry defined in Section 9.7.8 of [@RFC8555] as defined below.

Label        | Identifier Type | ACME | Reference
-------------|-----------------|------|---------------------------
onion-v3-csr | onion-v3        | Y    | I-D.shoemaker-acme-onion

# Security Considerations

Probably should consider something here (may want to reference [@TOR-REND-V3] Section 2.2.7.).

# Acknowledgments

Thank people (Wayne Thayer for his work on the CABF ballot).

{backmatter}

<reference anchor='TOR-REND-V3' target='https://spec.torproject.org/rend-spec-v3'>
    <front>
        <title>Tor Rendezvous Specification - Version 3</title>
        <author>
          <organization>Tor Project</organization>
        </author>
        <date year='2020'/>
        <format type='HTML' target='https://spec.torproject.org/rend-spec-v3'/>
    </front>
</reference>
<reference anchor='CABF-BR' target='https://cabforum.org/wp-content/uploads/CA-Browser-Forum-BR-1.6.8.pdf'>
    <front>
        <title>Baseline Requirements for the Issuance and Management of Publicly-Trusted Certificates, Version 1.6.8</title>
        <author>
          <organization>CA/Browser Forum</organization>
        </author>
        <date year='2020'/>
        <format type='PDF' target='https://cabforum.org/wp-content/uploads/CA-Browser-Forum-BR-1.6.8.pdf'/>
    </front>
</reference>
