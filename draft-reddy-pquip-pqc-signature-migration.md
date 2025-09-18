---
title: Guidance for Migration to Composite, Dual-Certificate, or PQ-Only Authentication
abbrev: PQ Signature Migration Guidance
docname: draft-reddy-pquip-pqc-signature-migration-latest
category: info
consensus: true
submissiontype: IETF

ipr: trust200902
area: Security
workgroup: PQUIP
keyword: Internet-Draft

stand_alone: yes
pi:
  rfcedstyle: yes
  toc: yes
  tocindent: yes
  sortrefs: yes
  symrefs: yes
  strict: yes
  comments: yes
  inline: yes
  text-list-symbols: o-*+
  compact: yes
  subcompact: yes
  consensus: false

author:
  -
    fullname: Tirumaleswar Reddy
    organization: Nokia
    city: Bangalore
    region: Karnataka
    country: India
    email: "kondtir@gmail.com"


normative:
  RFC2119:


informative:

--- abstract

This document provides guidance for migration from traditional digital
signature algorithms to post-quantum cryptographic (PQC) signature
algorithms. It compares three models under discussion in the IETF for
PKI-based protocols: composite signatures, dual
certificates, and PQ-only certificates. The goal is to help operators
and engineers working on cryptographic libraries, network security, and
PKI/key management infrastructure select an approach that balances interoperability,
security, and operational efficiency during the transition to
post-quantum authentication.

--- middle

# Introduction

The emergence of cryptographically relevant quantum computer (CRQC) poses a
threat to widely deployed public-key algorithms such as RSA and elliptic-curve
cryptography (ECC). Post-quantum algorithms are being standardized by NIST and other bodies,
but migration is not immediate. In the meantime, protocols need to ensure
that authentication mechanisms remain secure against both classical and
quantum adversaries.

For data authentication, the primary concern is the risk of on-path
attackers equipped with CRQCs. Such adversaries could break
certificate-based mechanisms that rely on traditional algorithms
(e.g., RSA, ECDSA), allowing them to impersonate servers and mount
man-in-the-middle (MitM) attacks. In this scenario, attackers could also
suppress PQC certificates and present only traditional ones, enabling
downgrades. These risks highlight the need to transition away from
traditional signatures and adopt post-quantum algorithms for
certificate-based authentication.

The IETF has developed several transition models for use in TLS, IKEv2/IPsec,
JOSE/COSE, and PKIX:

* Composite signatures – A single certificate, key, and signature that
  combines traditional and PQ algorithms {{!I-D.ietf-lamps-pq-composite-sigs}}.
* Dual certificates – Two separate certificates, one classical and one PQ,
  that are bound together and validated jointly. (e.g., {{!RFC9763}})
* PQ-only certificates – A certificate or signature that uses only a PQ
  algorithm ({{!I-D.ietf-lamps-dilithium-certificates}} for ML-DSA and {{!I-D.ietf-lamps-x509-slhdsa}} for SLH-DSA).

This document provides guidance on selecting among these approaches,
depending on the deployment context, the readiness of the supporting
ecosystem , and the applicable security requirements.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

Composite: A key, certificate, or signature that merges traditional and PQ algorithms into one object.
Dual certificates: Two independent certificates (traditional and PQ) issued for the same identity, presented and validated together.
PQ-only: A certificate or signature using only a PQ algorithm.

# Motivation for PQC Signatures

Unlike "Harvest Now, Decrypt Later" attacks that target confidentiality,
this risk directly impacts authentication and trust. Once a CRQC is
available, the continued use of traditional certificates becomes untenable.
In practice, however, the availability of a CRQC may not be publicly disclosed.
Similar to a zero-day vulnerability, an adversary could secretly exploit CRQC
capabilities to compromise traditional certificates without alerting the wider
ecosystem.

# Composite Signatures

A composite certificate contains both a traditional public key algorithm (e.g., ECDSA) and a post-quantum algorithm (e.g., ML-DSA) within a single X.509 certificate. This design enables both algorithms to be used in parallel, the traditional component ensures compatibility with existing infrastructure, while the post-quantum component introduces resistance against future quantum attacks.

Composite certificates are defined in {{!I-D.ietf-lamps-pq-composite-sigs}}. These combine Post-Quantum algorithms like ML-DSA with traditional algorithms such as RSA-PKCS#1v1.5, RSA-PSS, ECDSA, Ed25519, or Ed448, to provide additional protection against vulnerabilities or implementation bugs in a single algorithm. {{!I-D.reddy-tls-composite-mldsa}} specifies how composite signatures, including ML-DSA, are used for TLS 1.3 authentication. Composite-trust clients validate a single chain anchored in a composite root, without needing parallel chains.

## Advantages

* A single certificate chain is used for both traditional and post-quantum keys,
  simplifying certificate management.
* A single composite signature, rooted in one intermediate certificate chain,
  reduces protocol message size compared to transmitting multiple separate
  signatures, each of which would require its own certificate chain.
* No need to manage or validate multiple parallel certificate chains.
* No major changes to the base protocol are required to support the composite
  signature approach.

## Disadvantages

* Requires endpoints, relying parties, and CAs to support composite public keys
  and composite signature verification, which are not widely deployed at the
  time of writing of the specification.
* Introduces new certificate formats and verification logic that will need
  updates to PKI infrastructure.
* Expands the migration landscape to three transition paths:
  1. Traditional-only,
  2. Composite (classical + PQ),
  3. PQ-only.
  In contrast, the dual certificate approach requires only two paths
  (traditional and PQ).

# Dual Certificates

Dual certificates rely on issuing two separate certificates for the same
identity: one with a traditional algorithm (e.g., RSA or ECDSA) and one with
a post-quantum algorithm (e.g., ML-DSA). Both certificates are presented
and validated during authentication, providing hybrid assurance without
requiring new certificate formats.

## Advantages

* Uses standard, single-algorithm X.509 certificates and chains, maximizing
  compatibility with existing PKI infrastructures.
* Maintains clear separation between traditional and post-quantum keys.
* Requires no changes to certificate validation logic.

## Disadvantages

* Increases protocol message size due to the transmission of multiple
  certificate chains and signatures.
* Requires management of multiple certificates.
* Requires major changes to the base protocol to support dual certificate
  binding and validation.

# PQ-Only Certificates

PQ-only certificates represent the final stage of migration. They use
only a post-quantum algorithm and provide no fallback to traditional algorithms.

## Advantages

* Simpler model without the complexity of hybrid mechanisms.
* Forward-looking design, avoiding eventual deprecation of traditional
  algorithms.
* Reduced operational burden compared to managing dual or composite certificates.

## Disadvantages

* Risk if a deployed PQ algorithm is broken due to a bug.
* No interoperability with legacy systems that only support traditional
  algorithms.
* Deployment is only feasible once PQ algorithms are standardized and
  broadly supported across the ecosystem.

# Negotiation of Authentication Schemes

During the transition, endpoints may support multiple authentication
schemes (e.g., traditional, composite, dual, or PQ-only). Clients
advertise their supported schemes using the protocol’s negotiation
mechanism (for example, the `signature_algorithms` extension in TLS
{{!RFC8446}}), and servers select from the client’s list or fail the
authentication if no common option is available. In practice,
deployments are expected to prefer PQ-only or hybrid (composite or
dual) schemes over traditional ones, with the choice between PQ-only
and hybrid influenced by regulatory mandates or by whether
defense-in-depth is prioritized.

# Operational and Ecosystem Considerations

Migration to post-quantum authentication requires addressing broader
ecosystem dependencies, including trust anchors, hardware security modules,
and constrained devices.

## Trust Anchors and Transitions

Trust anchors represent the ultimate root of trust in a PKI. If existing
trust anchors are RSA or ECC-based, then new PQ-capable trust anchors will
need to be distributed. Operators will have to plan for a phased introduction of
PQ trust anchors, which may involve:

* Rolling out hybrid trust anchors that support both traditional and PQC
  signatures.
* Establishing parallel trust anchor hierarchies and phasing out the
  traditional hierarchy once PQ adoption is universal.
* Ensuring secure and authenticated distribution of updated trust anchors
  to clients, especially devices that cannot be easily updated.

Deployments migrating from traditional to post-quantum authentication
may have to operate with multiple trust anchors for a period of time. A new PQ
or composite root may be introduced while the existing traditional root
remains in place, leading to different trust chain models:

- Traditional chain: anchored in a Traditional root (e.g., RSA/ECDSA).
- PQ chain: anchored in a PQ root (e.g., ML-DSA, SLH-DSA).
- Composite chain: anchored in a composite root, with intermediates
  signed using composite algorithms that combine traditional
  and PQ. This forms a distinct new chain, rather than two parallel ones.

During this coexistence phase, clients generally fall into five categories:

1. Legacy-only: trust only traditional roots and support only
   traditional algorithms.
2. Mixed: trust only traditional roots but support both classical and
   PQ algorithms. These clients can validate PQ certificates only if a PQ
   intermediate is cross-signed by a traditional root.
3. Dual-trust: trust both traditional and PQ roots, supporting both
   algorithm families.
4. Composite-trust: trusts composite root and support composite
   algorithms, validating a single chain that integrates traditional and PQ
   signatures.
5. PQ-only: trust only PQ roots and support only PQ algorithms.

The main challenge is that servers cannot easily distinguish between mixed
clients (2) and dual-trust clients (3), since both advertise PQ algorithms,
but only dual-trust clients actually recognize PQ roots. To ensure
compatibility with mixed clients (2), servers may default to sending longer
PQ chains that include a cross-signed PQ root (i.e., a PQ root certificate
signed by a traditional root). However, this is unnecessary for dual-trust
clients (3), which already trust the PQ root directly, and it results in
increased message size.

{{!I-D.ietf-tls-trust-anchor-ids}} (TAI) addresses this problem by
allowing clients to explicitly indicate which trust anchors they recognize.
This enables servers to select the appropriate chain more efficiently,
reduce message size, and gain better visibility into clients readiness for
PQ-only operation. TAI also allows PQ-capable clients to enforce use of PQ
chains while still advertising broader trust sets to other servers.

{{!I-D.ietf-tls-trust-anchor-ids}} (TAI) addresses this problem by allowing
clients to indicate, on a per-connection basis, which trust anchors they
recognize. Servers can use that information to select a compatible certificate
chain, reducing unnecessary chain elements and providing operators with better
telemetry on PQC adoption. TAI also enables PQ-capable clients to tell PQ-aware
servers exactly which PQ trust anchors they recognize, while still supporting
traditional roots for compatibility with legacy servers.

In all cases, the long-term goal is a transition to PQ-only roots and
certificate chains. Composite and dual-certificate approaches help bridge
the gap, but operators will have to plan carefully for the eventual retirement of
traditional and composite roots once PQ adoption is widespread.

## Multiple Transitions and Crypto-Agility

Post-quantum migration is not a single event. There may be multiple
transitions over time, as:

* Traditional signature algorithms are gradually retired.
* Initial PQC signature algorithms are standardized and deployed.
* New PQC signature algorithms may replace early ones due to cryptanalysis or
  efficiency improvements.

Protocols and infrastructures will have to be designed with crypto-agility in mind,
supporting:

* Negotiation of both individual PQC algorithms and hybrid combinations
  with traditional algorithms.
* Phased migration paths, including initial use of PQ/T mechanisms,
  eventual transition to PQ-only certificates, and later migration
  to new PQ algorithms as cryptanalysis or policy guidance evolves.
* Protection against downgrade attacks across all transition phases.

## Support from Hardware Security Modules (HSMs)

Many organizations rely on HSMs for secure key storage and operations.
Challenges include:

* HSMs must be upgraded to support PQ algorithms and, where relevant,
  composite or dual key management models.
* PQC algorithms often have larger key sizes and signatures, requiring
  sufficient memory and processing capability in HSMs.
* For dual-certificate deployments, HSMs can manage the underlying
  traditional and PQ private keys independently, and no API changes are
  required. The application layer is responsible for coordinating how
  signatures from both keys are used. By contrast, supporting composite
  keys and composite signing operations may require HSM and API extensions
  to represent composite key objects and perform multi-algorithm signing
  atomically.

Without HSM vendor support for PQC, migration may be delayed or require
software-based fallback solutions, which will weaken security.

## Constrained Devices and IoT Environments

Constrained environments, such as IoT devices, present unique challenges
for PQC deployment. A broad set of challenges and potential mitigations is
described in {{!I-D.ietf-pquip-pqc-hsm-constrained}}, which analyzes PQC
deployment in resource-constrained devices and hardware security modules.

# Transition Considerations

Determining whether and when to adopt PQC-only certificates, hybrid
certificates, or dual certificates depends on several factors, including:

- Frequency and duration of system upgrades
- The expected timeline for CRQC availability
- Operational flexibility to enable or disable algorithms

Deployments with limited flexibility benefit from composite or dual
certificates. These approaches mitigate risks associated with delays in
transitioning to PQC and provide an immediate safeguard against zero-day
vulnerabilities. Both approaches improve resilience during migration, but
they do so in different ways and carry different operational trade-offs.

Composite or dual certificates enhance resilience during the adoption of
PQC by:

- Providing defense in depth: security is maintained as long as either
  the PQ or traditional algorithm remains unbroken.
- Reducing exposure to unforeseen vulnerabilities: immediate protection
  against weaknesses in PQC algorithms.

However, each approach comes with long-term implications.

## Composite Certificates

Composite certificates embed both a traditional and a PQC algorithm into a
single certificate and signature. However, once a traditional algorithm is no
longer secure against CRQCs, it must be deprecated. For discussion
of the security impact in security protocols (e.g., TLS, IKEv2)
versus artifact-signing use cases, see Section {{suf}}.

To complete the transition to a fully quantum-resistant authentication model,
operators will need to provision a new root CA certificate that uses only a
PQC signature algorithm and public key. This new root CA would issue a hierarchy
of intermediate certificates, each also signed using a PQC algorithm, ultimately
leading to end-entity certificates that contain only PQC public keys and are signed with
PQC algorithms.

Protocol configurations (e.g., TLS, IKEv2) will likewise
need to be updated to negotiate only PQC-based authentication, ensuring that
the entire certification path and protocol handshake are cryptographically
resistant to quantum attacks and no longer depend on any traditional
algorithms.

## Dual Certificates

When CRQCs become available, the traditional certificate chain will no
longer be secure. At that point, the traditional chain must be removed,
and the protocol configuration updated so that only the PQC certificate
chain is presented and validated. This requires careful coordination
during the transition, since legacy clients that cannot process PQC
certificates will lose access once the traditional chain is withdrawn.
Dual-certificate deployments therefore defer, but do not avoid, the need
to update protocol configurations and move to a PQ-only environment.

## Loss of Strong Unforgeability in Composite and Dual Certificates {#suf}

A deployment may choose to continue using a composite or dual-certificate
configuration even after a traditional algorithm has been broken by the
advent of a CRQC. While this may simplify operations by avoiding
re-provisioning of trust anchors, it introduces a significant risk:
security properties degrade once one component of the hybrid is no longer
secure.

In composite certificates, the composite signature will no longer achieve Strong
Unforgeability (SUF) (see Section 10.1.1 of {{?I-D.ietf-pquip-pqc-engineers}}
and Section 10.2 of {{!I-D.ietf-lamps-pq-composite-sigs}}). A CRQC can forge the
broken traditional signature component (s1*) over a message (m). That forged
component can then be combined with the valid post-quantum component (s2) to
produce a new composite signature (m, (s1*, s2)) that verifies successfully,
thereby violating SUF.

In dual-certificate deployments where the client requires both a
traditional and a PQ chain, the SUF property is likewise not achieved once
the traditional algorithm is broken.

In protocols such as TLS and IKEv2/IPsec, a composite signature remains
secure against impersonation as long as at least one component algorithm
remains unbroken, because verification succeeds only if every
component signature validates over the same canonical message defined
by the authentication procedure. However, in artifact signing
use cases, the break of a single component does not enable forgery of a
composite signature but does enable "repudiation": multiple distinct
composite signatures can exist for the same artifact, undermining the
“one signature, one artifact” guarantee. This creates ambiguity about
which composite signature is authentic, complicating long-term
non-repudiation guarantees.

# Migration Guidance

* Experimental (individual drafts): Dual certificates have been
  proposed as a migration mechanism (e.g., {{?I-D.hu-ipsecme-pqt-hybrid-auth}},
  {{?I-D.yusef-tls-pqt-dual-certs}}, {{?I-D.bonnell-lamps-chameleon-certs}}),
  but at the time of writing none of the relevant IETF working groups
  have adopted this approach. Dual certificates require significant
  protocol changes to bind and validate both certificates correctly
  (e.g., extensions in TLS, multiple CERT payload handling in IKEv2).

* Medium-term (standards-track): Composite signatures enable tighter
  integration once ecosystem support (PKIX, IPSec, JOSE/COSE, TLS) is mature.
  Composite signatures are already being standardized in the LAMPS WG
  ({{!I-D.ietf-lamps-pq-composite-sigs}}) and leveraged in
  {{?I-D.reddy-tls-composite-mldsa}} for TLS,
  {{?I-D.hu-ipsecme-pqt-hybrid-auth}} for IPsec/IKEv2, and
  {{?I-D.prabel-jose-pq-composite-sigs}} for JOSE/COSE.

* Long-term: PQ-only certificates are the final goal, once PQ
  algorithms are well-established, trust anchors have been updated,
  HSMs and devices natively support PQ operations, and traditional
  algorithms are fully retired.

# Use of SLH-DSA in PQ-Only Deployments

SLH-DSA does not introduce any new hardness assumptions beyond those inherent
to its underlying hash functions. It builds upon established cryptographic
foundations, making it a reliable and robust digital signature scheme for a
post-quantum world. While attacks on lattice-based schemes such as ML-DSA are
currently hypothetical, if realized they could compromise the security of those
schemes. SLH-DSA would remain unaffected by such attacks due to its distinct
mathematical foundations, helping to ensure the ongoing security of systems and
protocols that rely on it for digital signatures. Unlike ML-DSA, SLH-DSA is not
defined for use in composite certificates and is intended to be deployed directly
in PQ-only certificate hierarchies.

SLH-DSA may be used for both end-entity and CA certificates. It provides strong
post-quantum security but produces larger signatures than ML-DSA or traditional
algorithms. At security levels 1, 3, and 5, two parameter sets are available:

* "Small" (s) variants minimize signature size, ranging from 7856 bytes
  (128-bit) to 29792 bytes (256-bit).
* "Fast" (f) variants optimize key generation and signing speed, with
  signature sizes from 17088 bytes (128-bit) to 29792 bytes (256-bit),
  but slower verification performance.

Because of these large signatures, SLH-DSA will increase handshake size in
protocols such as TLS 1.3 or IKEv2. However, the impact on performance is
minimal for long-lived connections or large data transfers, where handshake
overhead is amortized over session duration (e.g., DTLS-in-SCTP in 3GPP N2
interfaces, or signature authentication in IKEv2 using PQC
{{!I-D.ietf-ipsecme-ikev2-pqc-auth}}).
In deployments where minimizing handshake size is critical, operators may
prefer SLH-DSA for root and intermediate certificates while using smaller-
signature algorithms (e.g., ML-DSA) in end-entity certificates or in the
"CertificateVerify" message.

Mechanisms such as Abridged TLS Certificate Chains {{?I-D.ietf-tls-cert-abridge}} and
Suppressing CA Certificates {{?I-D.kampanakis-tls-scas-latest}} reduce handshake size
by limiting certificate exchange to only end-entity certificates. In such cases,
intermediate certificates are assumed to be known to the peer, allowing the use of
larger signature algorithms like SLH-DSA for those certificates without adding
overhead to the handshake.

# Security Considerations

Hybrid approaches (composite or dual certificates) are designed to provide
defense in depth during the migration to PQC. Their goal is to ensure that
authentication remains secure as long as at least one of the algorithms in
use remains unbroken. However, several important security considerations
arise.

## Downgrade Attacks

Implementations must ensure downgrade protection so that an adversary cannot
suppress PQ or hybrid certificates and force reliance solely on traditional
algorithms. This is especially important in scenarios where a CRQC is
available but not publicly disclosed. Without downgrade protection, a MitM
attacker could impersonate servers by presenting only traditional
certificates even when PQC certificates are supported.

## Strong Unforgeability versus Existential Unforgeability

In composite and dual-certificate deployments, once one component algorithm
is broken (e.g., the traditional algorithm under a CRQC), the overall scheme
no longer achieves SUF. While Existential Unforgeability (EUF) is still
preserved by the PQ component, the loss of SUF means that hybrid mechanisms
will have be retired once traditional algorithms are no longer secure.

## Operational Risks

Managing multiple certificate paths (composite, dual, and PQ-only) increases
the risk of misconfiguration and operational errors. For example, a server
might continue using a hybrid scheme after the traditional algorithm
is broken, fail to revoke traditional certificates that are no longer secure,
or misconfigure chain selection so that clients receive an incompatible path.
Clear operational guidance and automated monitoring are essential to minimize
these risks. Operators need best practices for certificate lifecycle and
migration planning, along with automated checks to ensure PQC chains remain
present, valid, and not replaced by weaker alternatives.

# IANA Considerations

This document has no IANA actions.

# Acknowledgments

TODO

