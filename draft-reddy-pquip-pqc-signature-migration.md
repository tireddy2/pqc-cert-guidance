---
title: Guidance for Migration to Composite, Dual, or PQC-Only Authentication
abbrev: PQC Signature Migration Guidance
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
    email: "k.tirumaleswar_reddy@nokia.com"
  -
    fullname: Dan Wing
    organization: Citrix
    abbrev: Citrix
    country: United States of America
    email: danwing@gmail.com
  -
    ins: "Y. Rosomakho"
    fullname: Yaroslav Rosomakho
    organization: Zscaler
    email: yrosomakho@zscaler.com


normative:


informative:

--- abstract

This document provides guidance for migration from traditional digital
signature algorithms to post-quantum cryptographic (PQC) signature
algorithms. It compares three models under discussion in the IETF for
PKI-based protocols: composite certificates, dual
certificates, and PQC-only certificates. The goal is to help operators
and engineers working on cryptographic libraries, network security, and
PKI/key management infrastructure select an approach that balances
interoperability, security, and operational efficiency during the
transition to post-quantum authentication.

--- middle

# Introduction {#intro}

The emergence of cryptographically relevant quantum computer (CRQC) poses a
threat to widely deployed public-key algorithms such as RSA and elliptic-curve
cryptography (ECC). Post-quantum algorithms are being standardized by NIST
and other bodies, but migration is not immediate. In the meantime, protocols
need to ensure that authentication mechanisms remain secure against both
classical and quantum adversaries.

For data authentication, the primary concern is that adversaries who obtain a
CRQC will be able to forge digital signatures produced by traditional
public-key algorithms (e.g., RSA, ECDSA). Such forgeries enable a
range of attacks, including on-path man-in-the-middle (MitM)
attacks, and off-path attacks such as software-artifact forgery, and client
impersonation in mutual TLS when a client private key is compromised. In addition,
on-path adversaries can attempt active downgrade techniques (for example,
suppressing PQC-only or hybrid signature schemes during negotiation) to force reliance on
broken traditional algorithms. PQC-only or Hybrid certificates do not by themselves
prevent downgrade attack when relying parties continue to accept traditional-only
certificates. These risks motivate a transition of certificate-based authentication
toward post-quantum security.

The IETF has defined two hybrid transition models for use in TLS, IKEv2/IPsec,
JOSE/COSE, and PKIX:

* Composite certificates: A single X.509 certificate that contains a composite
  public key and a composite signature, combining a traditional and a PQC algorithm.
  Certificates using composite ML-DSA are specified in {{!COMPOSITE-ML-DSA=I-D.ietf-lamps-pq-composite-sigs}}.

* Dual-certificate model: A deployment model in which two separate certificates, one using a traditional
  algorithm and one using a PQC algorithm, issued for the same identity, presented and validated together
  during authentication. Some protocols may require these certificates to include the RelatedCertificate extension {{?RELATED-CERTS=RFC9763}} to ensure that both refer to the same identity and binding.

Another approach is to use a PQC-only certificate which contains only a post-quantum
public key and produces signatures using a PQC algorithm. Examples include {{!ML-DSA=I-D.ietf-lamps-dilithium-certificates}}
and {{!SLH-DSA=I-D.ietf-lamps-x509-slhdsa}}.

This document provides guidance on selecting among the two hybrid
certificate models and the PQC-only model depending on the deployment
context, the readiness of the supporting ecosystem, and security
requirements.

It is important to note that the use of PQC-only certificates, composite
certificates, or the dual-certificate model alone does not guarantee
post-quantum security. As long as relying parties continue to trust or
accept traditional-only certificates, an attacker equipped with a CRQC
can forge traditional certificates and impersonate an authenticated
party, even if that party does not use a traditional certificate. Post-quantum security
is achieved only when relying parties enforce policies that reject
traditional-only authentication.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

This document uses the terms "composite certificates" and "PQC-only certificates"
as defined in {{intro}}.

The term "dual certificates" in this document refers to the
dual-certificate model as defined in {{intro}}.

Composite: A key, certificate, or signature that merges traditional
and PQC algorithms into one object.

The terms hybrid signature scheme and hybrid signature are used as
defined in {{!HYBRID-SPECTRUMS=I-D.ietf-pquip-hybrid-signature-spectrums}}.

Relying Party:  An endpoint which validates the certificate of a remote peer.  
With classic HTTPS authentication, this is the HTTPS client.  With mutual TLS 
authentication, this is both TLS endpoints.

Authenticated Party: An endpoint which provides its certificate for a 
remote peer to validate.  With classic HTTPS authentication, this is the HTTPS 
server.  With mutual TLS authentication, this is both TLS endpoints.

# Motivation for PQC Signatures

Unlike "Harvest Now, Decrypt Later" attacks (see {{Section 7 of ?PQC-ENGINEERS=I-D.ietf-pquip-pqc-engineers}})
that target the confidentiality of encrypted data, the threat to authentication arises
only from the moment a CRQC becomes available. Compromise of authentication
is therefore not retrospective: previously established identities and signatures
cannot be forged in hindsight, but all future authentications using traditional
algorithms become insecure once a CRQC exists.

Once a CRQC is available, continued reliance on traditional public-key
algorithms (e.g., RSA, ECDSA) becomes untenable, as an attacker could forge
digital signatures and impersonate legitimate entities. In practice, the
availability of a CRQC may not be publicly disclosed. Similar to a zero-day
vulnerability, an adversary could exploit quantum capabilities privately to
compromise traditional certificates without alerting the wider ecosystem.

Addressing this risk requires replacing traditional signatures with
post-quantum (PQC) signatures.  Doing so entails ecosystem-wide upgrades across:

* Software components: cryptographic libraries and protocol
  implementations;
* Hardware security devices: Hardware Security Modules (HSMs) and
  Trusted Platform Modules (TPMs);
* Public Key Infrastructure (PKI): Certification Authorities (CAs),
  intermediate CAs, and trust anchors;
* Dependent protocols: TLS ({{?TLS=I-D.ietf-tlsrfc8446bis}}, {{?DTLS=RFC9147}}), {{?IKEv2=RFC5996}}, and JOSE/COSE.

Because these transitions require years of planning, coordination, and
investment, preparations must begin well before a CRQC is publicly known.

PQC-only or hybrid certificates provide post-quantum security only when relying parties
reject traditional-only certificates. The implications of this requirement differ
across deployment environments:

- Open environments (e.g., the Web):
  Enforcing rejection of traditional-only certificates would cause substantial disruption
  because of the diversity of clients and servers.
  In such ecosystems, it is unlikely that relying parties will stop accepting traditional
  certificates until PQC-only or hybrid certificate deployment becomes significantly high
  or there is credible evidence that CRQCs exist.

- Closed or enterprise-managed environments:
  In deployments where both the authenticating party and the relying party
  are managed by the same organization, enforcing PQC-only or hybrid authentication
  policies is operationally feasible. Organizations can coordinate certificate issuance
  and validation policies centrally, enabling earlier transition to PQC-only or hybrid
  models without affecting interoperability.

# Composite certificates

A composite certificate contains a composite public key and a composite
signature, each combining a traditional and a post-quantum (PQC) algorithm
within a single X.509 structure. Both the key and the signature use new
encodings defined in {{!I-D.ietf-lamps-pq-composite-sigs}}, and
therefore composite certificates do not offer interoperability with
legacy PKI deployments. The goal is of the composite approach is
defense-in-depth: the traditional component preserves authentication
security if a flaw is found in the PQC algorithm before a CRQC exists,
while the PQC component preserves security after CRQCs can break traditional
algorithms. Verification succeeds only if all component signatures validate
over the same canonical message.

ML-DSA composite certificates are defined in
{{!I-D.ietf-lamps-pq-composite-sigs}}, which defines the use of ML-DSA
in combination with one or more traditional algorithms such as
RSA-PKCS#1v1.5, RSA-PSS, ECDSA, Ed25519, or Ed448. The framework in that
document is designed to be extensible and is expected to accommodate
additional post-quantum algorithms in future specifications.

Protocol-specific drafts describe how composite certificates are used in
different environments, including:
{{?TLS-COMPOSITE-ML-DSA=I-D.reddy-tls-composite-mldsa}} for TLS,
{{?IKEv2-COMPOSITE-ML-DSA=I-D.hu-ipsecme-pqt-hybrid-auth}} for IKEv2,
and {{?JOSE-COSE-COMPOSITE-ML-DSA=I-D.prabel-jose-pq-composite-sigs}} for JOSE and COSE.
In each case, the relying party validates a single certification path
anchored in a multi-algorithm trust anchor, avoiding the need for
parallel certificate chains.

## Advantages

A key benefit of the composite model is single-path operation. Because both
algorithms are embedded in one certificate chain, the relying party validates
only one path, which reduces chain-management complexity compared to dual-chain
deployments. Conveying a single certificate and signature object can also
reduce message size relative to transmitting two independent chains. From a
protocol perspective, composite certificates typically require minimal changes
to handshakes, since authentication still relies on one certificate and one signature.

## Disadvantages

The main challenge with composite certificates is ecosystem readiness.
Clients, servers, and Certification Authorities must support composite
public keys and composite signature verification, which are not yet
widely deployed. The new certificate encodings and multi-algorithm
signing introduce updates across PKI components, libraries, and Hardware
Security Modules. Once these components support the composite structures,
using a composite signature algorithm is no more complex than adopting
any new PQC algorithm.

Another operational limitation is the need for algorithm-set
coordination: all participants in a composite ecosystem must agree on
the specific and acceptable combinations of post-quantum and traditional
algorithms (for example, ML-DSA-44 + ECDSA P-256 or ML-DSA-65 + EdDSA Ed448).
A composite certificate can only be validated if both endpoints and all
intermediate CAs recognize the same algorithm identifiers and policy.
Disagreement on permitted combinations can lead to handshake failures,
certificate re-issuance delays, or policy fragmentation across vendors.
This is primarily a policy and interoperability issue during early deployment:
once endpoints and CAs recognize the same algorithm identifiers and policies,
a composite algorithm behaves like any other registered signature algorithm.

Composite deployments are also an intermediate step: once traditional
algorithms are deprecated due to CRQCs, operators will still need to
transition from composite to PQC-only certificates. This requires
deploying new PQC trust anchors, issuing PQC-only certificates, and
revoking composite certificates. While automated mechanisms such as
ACME or CMP can streamline end-entity certificate issuance, trust anchors are
typically distributed through OS, Browser, or device
update mechanisms, and their replacement generally requires
platform-specific processes. As a result, for some organizations, this
two-stage path may lengthen the overall migration.

# Dual Certificates

Dual certificates rely on issuing two separate certificates for the same
identity: one using a traditional algorithm (for example, RSA or ECDSA)
and one using a post-quantum algorithm (for example, ML-DSA). Both
certificates are presented and validated during authentication,
providing hybrid assurance without introducing new certificate formats
or encodings.

## Advantages

A major advantage of the dual-certificate model is its negotiation
flexibility. Because each certificate contains only a single algorithm,
endpoints do not need to agree in advance on a specific combination of
traditional and post-quantum algorithms. The server can select which
certificate (or both) to present based on the client's advertised
capabilities, and the client can validate whichever chain it supports.
This enables smoother incremental deployment and interoperation between
implementations that support different PQC algorithms or security
policies.

Dual certificates also use standard X.509 structures and single-algorithm
chains, maximizing compatibility with existing PKI and avoiding changes
to certificate parsing or signature verification logic. The clear separation
between traditional and PQC keys simplifies operational control, audit,
and incident response. Deployments can move from traditional-only to dual
certificates, and later retire the traditional certificate when PQC support
is ubiquitous, without redefining certificate formats or introducing composite
encodings. The model also fits well in multi-tenant environments where
different tenants or business units may adopt different combinations of
traditional and PQC algorithms without requiring global agreement on a
composite set.

## Disadvantages

The dual-certificate model increases protocol overhead, since both
certificate chains and signatures must be transmitted and validated.
Protocols that traditionally authenticate a single certificate chain,
such as {{?TLS=I-D.ietf-tlsrfc8446bis}} and {{?IKEv2=RFC5996}}, require
extensions to support validation of two end-entity certificates and to
ensure that both are cryptographically bound to the same identity.
This adds implementation complexity and may increase handshake latency.

Managing two distinct certificate chains introduces operational cost and
new failure modes. Debugging becomes more difficult, as validation
errors may originate from either chain or from inconsistent identity
binding. Operators must also obtain and renew two certificates from
Certification Authorities, which can be significant in large-scale
deployments.

Finally, while dual certificates avoid the need for a fixed algorithm
pairing, they require explicit binding and coordination between the
two chains. Each relying party must verify that the traditional and PQC
certificates correspond to the same entity, typically using mechanisms
such as the RelatedCertificate extension {{?RELATED-CERTS=RFC9763}}.
Lack of consistent binding policies can lead to interoperability issues
and potential downgrade risks if only one chain is validated.

# PQC-Only Certificates

PQC-only certificates represent the final stage of migration.
They use exclusively post-quantum cryptographic algorithms for both
public keys and signatures, providing no fallback to traditional
algorithms.  Once adopted at scale, they eliminate hybrid complexity and
rely entirely on quantum-resistant primitives for authentication.

## Advantages

The PQC-only model offers the simplest and most forward-looking
architecture. It removes all dependency on classical algorithms, thus
avoiding future deprecation or phased-out support for RSA and ECC.
Certificate management is streamlined, as there is only one algorithm
family to provision, monitor, and renew. Operational overhead decreases
compared to hybrid or dual deployments, since each entity maintains a
single certificate chain and consistent cryptographic policy.

PQC-only certificates also enable long-term assurance: the entire
certificate path is verifiable using post-quantum signatures, ensuring
uniform resistance against quantum adversaries.

## Disadvantages

The primary risk of PQC-only deployments is algorithmic fragility. If a
vulnerability or cryptanalytic weakness is discovered in a deployed PQC
scheme, there is no classical fallback for continued authentication.
Protocols and infrastructures must therefore maintain strong
crypto-agility and be prepared to replace algorithms rapidly if needed.

Backward compatibility can be maintained if the authenticating party also
holds a traditional certificate and presents it to relying parties
that have not yet deployed PQC support. While this approach preserves
interoperability during the transition, it also introduces downgrade risk:
an attacker could suppress PQC options and force peers to authenticate
using the traditional certificate.

PQC-only operation where traditional algorithms are completely removed
eliminates this downgrade vector, but it is feasible only once relying
parties enforce PQC–only authentication.

Adoption may also be uneven across jurisdictions. Regulatory frameworks
and certification programs may not recognize the same PQC
algorithms at the same time. Divergent compliance regimes could delay
global deployment or require organizations operating in multiple
regions to maintain mixed trust infrastructures until regulatory
alignment is achieved.

Finally, PQC-only deployments remain feasible only once PQC algorithms
are fully standardized, broadly implemented, and supported by hardware
security modules, operating systems, and major application ecosystems.

# Operational and Ecosystem Considerations

Migration to post-quantum authentication requires addressing broader
ecosystem dependencies, including trust anchors, hardware security modules,
and constrained devices.

## Trust Anchors and Transitions

Trust anchors represent the ultimate root of trust in a PKI. If existing
trust anchors are RSA or ECC-based, then new PQC-capable trust anchors will
need to be distributed. Operators will have to plan for a phased introduction of
PQC trust anchors, which may involve:

* Rolling out composite trust anchors that support both traditional and PQC
  signatures.
* Establishing parallel trust anchor hierarchies and phasing out the
  traditional hierarchy once PQC adoption is universal.
* Ensuring secure and authenticated distribution of updated trust anchors
  to clients, especially devices that cannot be easily updated.

Deployments migrating from traditional to post-quantum authentication
may have to operate with multiple trust anchors for a period of time.
A new PQC or composite root may be introduced, or alternatively a PQ
intermediate may be added beneath an existing traditional root,
leading to different trust chain models:

- Traditional chain: anchored in a Traditional root (e.g., RSA/ECDSA),
  which may issue a PQC intermediate.
- PQC chain: anchored in a PQC root (e.g., ML-DSA, SLH-DSA).
- Parallel roots: both a traditional root and a PQC root are distributed as
  trust anchors, with separate hierarchies operating in parallel until the
  traditional root can be phased out.
- Composite chain: anchored in a composite root and using composite
  algorithms, with a single certificate chain that combines traditional and
  PQC public keys and signatures. This forms a distinct chain, rather
  than two parallel ones.

During this coexistence phase, clients generally fall into five categories:

1. Legacy-only: trust only traditional roots and support only
   traditional algorithms.
2. Mixed: trust only traditional roots but support both traditional and
   PQC algorithms. These clients can validate PQC certificates only if a PQC
   intermediate is cross-signed by a traditional root.
3. Dual-trust: trust both traditional and PQC roots, supporting both
   algorithm families.
4. Composite-trust: trusts composite root and support composite
   algorithms, validating a single chain that integrates traditional and PQ
   signatures.
5. PQC-only: trust only PQC roots and support only PQC algorithms.

The main challenge is that servers cannot easily distinguish between mixed
clients (2) and dual-trust clients (3), since both advertise PQC algorithms,
but only dual-trust clients actually recognize PQC roots. To ensure
compatibility with mixed clients (2), servers may default to sending longer
PQC chains that include a cross-signed PQC root (i.e., a PQC root certificate
signed by a traditional root). However, this is unnecessary and even counterproductive
for dual-trust clients (3), which already trust the PQC root directly;
such clients will fail to validate the cross-signed PQC root. For dual-trust clients,
including the cross-signed PQC root only increases message size and introduces
validation errors.

{{!I-D.ietf-tls-trust-anchor-ids}} (TAI) addresses this problem by allowing
clients to indicate, on a per-connection basis, which trust anchors they
recognize. Servers can use that information to select a compatible certificate
chain, reducing unnecessary chain elements and providing operators with better
telemetry on PQC adoption. TAI also enables PQC-capable clients to tell PQC-aware
servers exactly which PQC trust anchors they recognize, while still supporting
traditional roots for compatibility with legacy servers.

In all cases, the long-term goal is a transition to PQC-only roots and
certificate chains. Hybrid signature schemes help bridge
the gap, but operators will have to plan carefully for the eventual retirement of
traditional and composite roots once PQC adoption is widespread.

## Multiple Transitions and Crypto-Agility

Post-quantum migration is not a single event. There may be multiple
transitions over time, as:

* Traditional signature algorithms are gradually retired.
* Initial PQC signature algorithms are standardized and deployed.
* New PQC signature algorithms may replace early ones due to cryptanalysis or
  efficiency improvements.

Protocols and infrastructures will have to be designed with crypto-agility in mind,
supporting:

* Negotiation of standalone PQC algorithms and hybrid signature schemes.
* Phased migration paths, including initial use of hybrid signature schemes,
  eventual transition to PQC-only certificates, and later migration
  to new PQC algorithms as cryptanalysis or security policy guidance evolves.
* Downgrade protection is critical throughout the migration period,
  since relying parties may otherwise be tricked into accepting weaker
  traditional authentication even when PQC-only or composite credentials exist.
  For open environments (for example, the Web), one possible mitigation is
  the X.509 Post-Quantum/Composite Hosting Continuity (PQCHC) extension {{!PQCHC=I-D.reddy-lamps-x509-pq-commit-latest}}, which enables a certificate subject to
  convey an intent to continue presenting PQC or composite credentials
  for a configured continuity period beyond the certificate’s notAfter date.

## Support from Hardware Security Modules (HSMs)

Many organizations rely on HSMs for secure key storage and operations.
Challenges include:

* HSMs must be upgraded to support PQC algorithms and, where relevant,
  composite or dual key management models.
* PQC algorithms often have larger key sizes and signatures, requiring
  sufficient memory and processing capability in HSMs.
* For dual certificate deployments, HSMs can manage the underlying
  traditional and PQC private keys independently, and no API changes are
  required. The security protocol is responsible for coordinating how
  signatures from both keys are used. By contrast, supporting composite
  keys and composite signing operations will require HSM and API extensions
  to represent composite private keys and perform multi-algorithm signing
  atomically.

Without HSM vendor support for PQC, migration may be delayed or require
software-based fallback solutions, which will weaken security.

## Constrained Devices and IoT Environments

Constrained environments, such as IoT devices, present unique challenges
for PQC deployment due to limited processing, memory, and bandwidth.
Guidance is provided in {{!I-D.ietf-pquip-pqc-hsm-constrained}}, including
the use of seeds for efficient key generation, PQC-protected firmware
updates, and other techniques for enabling PQC in lightweight HSMs and
resource-constrained devices.

# Transition Considerations

Migration to post-quantum authentication will proceed gradually across
protocols, products, and organizations. During this period, endpoints
may support multiple authentication models (traditional, composite,
dual, or PQC-only) depending on their stage of deployment. The
transition requires careful coordination of certificate management,
protocol negotiation, and policy enforcement to maintain security and
interoperability throughout the migration.

## Transition Logic Overview

The migration to post-quantum authentication will occur in phases as
organizations adopt PQC algorithms and update their infrastructures.
Because CRQCs may be deployed without public disclosure, continued
reliance on traditional algorithms will become increasingly risky.
During the transition, dual certificates enable interoperability between
PQC-capable and legacy systems, while composite certificates provide
hybrid authentication within upgraded ecosystems. These approaches serve
as intermediate steps toward PQC-only deployments. Post-quantum security
is achieved only when relying parties stop accepting traditional-only
authentication. At that point, authenticated parties can also stop
issuing or presenting traditional-only certificates.

## Negotiation and Interoperability

During coexistence, endpoints must be able to discover which
authentication mechanisms the peer supports. In most protocols, this is
achieved through existing negotiation mechanisms such as, the
`signature_algorithms` extension in {{?TLS=RFC8446}}. Clients
advertise their supported algorithms and certificate types, and servers
select the strongest mutually supported option or fail authentication if
no common algorithm is found.

In hybrid or PQC-capable deployments, there is no security benefit if
authentication using only traditional algorithms continues to be
accepted, since an attacker can always downgrade to that option.
The specific choice between PQC-only and hybrid mechanisms may be influenced
by regulatory guidance, national cryptography policies, or the organization's
appetite for defense-in-depth during early adoption.

Negotiation mechanisms must also include downgrade protection so
that an adversary cannot suppress PQC or hybrid options and force a
fallback to traditional signatures. TLS already provide such
protection through transcript binding of the handshake messages that
carry the algorithm negotiation results, but new or proprietary protocols
have to ensure similar safeguards.

A deployment will typically adopt one of three models, PQC-only certificates,
dual certificates, or composite certificates.

The choice depends on several factors, including:

- Frequency and duration of system upgrades
- The expected timeline for CRQC availability
- Operational flexibility to deploy, enable, and retire PQC algorithms
- Availability of automated certificate provisioning mechanisms such as {{?ACME=RFC8555}} and {{?CMP=RFC9810}}

Deployments with limited flexibility benefit from hybrid signature schemes.
These approaches mitigate risks associated with delays in
transitioning to PQC and provide an immediate safeguard against zero-day
vulnerabilities. Both approaches improve resilience during migration, but
they do so in different ways and carry different operational trade-offs.

Hybrid signature schemes enhance resilience during the adoption of
PQC by:

- Providing defense in depth: security is maintained as long as either
  the PQC or traditional algorithm remains unbroken.
- Reducing exposure to unforeseen vulnerabilities: immediate protection
  against weaknesses in PQC algorithms.

However, each approach comes with long-term implications.

## Composite Certificates

Composite certificate embeds both a traditional and a PQC algorithm into a
single certificate and signature. However, once a traditional algorithm is no
longer secure against CRQCs, it will have to be deprecated. For discussion
of the security impact in security protocols, such as TLS and IKEv2,
versus artifact-signing use cases, see {{suf}}.

To complete the transition to a fully quantum-resistant authentication model,
operators will need a PQC CA root and CA intermediates, resulting in PQC-only
end-entity certificates.

Protocol configurations will likewise need to be updated to negotiate only
PQC-based authentication, ensuring that the entire certification path and
protocol handshake are cryptographically resistant to quantum attacks and
no longer depend on any traditional algorithms.

## Dual Certificates

When CRQCs become available, the traditional certificate chain will no
longer provide secure authentication. At that point, relying parties
must stop accepting or requesting traditional certificate chains and
validate only PQC-based chains. Authenticated parties will automatically
cease using traditional chains once relying parties no longer request
them. Dual-certificate deployments therefore defer, but
do not avoid, the eventual migration to a PQC-only environment.

## Loss of Strong Unforgeability in Composite and Dual Certificates {#suf}

A deployment may choose to continue using a composite or dual certificate
configuration even after a traditional algorithm has been broken by the
advent of a CRQC. While this may simplify operations by avoiding
re-provisioning of trust anchors, it introduces a significant risk:
security properties degrade once one component of the hybrid is no longer
secure.

In composite certificates, the composite signature will no longer achieve Strong
Unforgeability under chosen message attack (SUF-CMA) (see {{Section 10.1.1 of ?PQC-ENGINEERS=I-D.ietf-pquip-pqc-engineers}}
and {{Section 10.2 of !I-D.ietf-lamps-pq-composite-sigs}}). A CRQC can forge the
broken traditional signature component (s1*) over a message (m). That forged
component can then be combined with the valid post-quantum component (s2) to
produce a new composite signature (m, (s1*, s2)) that verifies successfully,
thereby violating SUF-CMA.

In dual certificate deployments where the client requires both a
traditional and a PQC chain, the SUF-CMA property is likewise not achieved once
the traditional algorithm is broken.

In protocols such as TLS and IKEv2, a composite signature remains
secure against impersonation as long as at least one component algorithm
remains unbroken, because verification succeeds only if every
component signature validates over the same canonical message defined
by the authentication procedure. However, in artifact signing
use cases, the break of a single component does not enable forgery of a
composite signature but does enable "repudiation": multiple distinct
composite signatures can exist for the same artifact, undermining the
"one signature, one artifact" guarantee. This creates ambiguity about
which composite signature is authentic, complicating long-term
non-repudiation guarantees.

Hybrid signature schemes should not be used for artifact signing (such as software packages),
since the loss of SUF-CMA makes them unsuitable for long-term non-repudiation.
In security protocols, hybrid signature schemes may continue to
function for a limited time after a CRQC is realized, since they still provide
impersonation resistance as long as one component algorithm remains secure.
This situation does not constitute a zero-day vulnerability requiring an
immediate upgrade. However, operators will have to plan an orderly migration
to PQC-only certificates in order to restore SUF-CMA security guarantees.

# Migration Guidance

* Long-term to adopt and deploy:: Dual certificates have been standardized in
  {{!RFC9763}}. However, at the time of writing, none of
  the security protocols (e.g., TLS, IKEv2, JOSE/COSE) have
  adopted this mechanism. The proposals are being discussed in IKEv2
  ({{?I-D.hu-ipsecme-pqt-hybrid-auth}}), TLS
  ({{?I-D.yusef-tls-pqt-dual-certs}}), and in the form of paired
  certificates with a single certificate
  ({{?I-D.bonnell-lamps-chameleon-certs}}).

* Medium-term to adopt and deploy: Composite certificates become viable once
  ecosystem support across PKIX, IPsec, JOSE/COSE, and TLS is mature.
  Composite ML-DSA is already being standardized in the LAMPS WG
  ({{!I-D.ietf-lamps-pq-composite-sigs}}) and leveraged in
  {{?I-D.reddy-tls-composite-mldsa}} for TLS,
  {{?I-D.hu-ipsecme-pqt-hybrid-auth}} for IPsec/IKEv2, and
  {{?I-D.prabel-jose-pq-composite-sigs}} for JOSE/COSE.

* Long-to-medium term to adopt and deploy: PQC-only certificates are the final goal, once PQ
  algorithms are well-established, trust anchors have been updated,
  HSMs and devices support PQC operations, and traditional
  algorithms are fully retired. Work to enable PQC signatures is already
  underway in JOSE/COSE {{!I-D.ietf-cose-dilithium}}, TLS {{!I-D.ietf-tls-mldsa}},
  and IPsec {{!I-D.ietf-ipsecme-ikev2-pqc-auth}}.

# Use of SLH-DSA in PQC-Only Deployments

SLH-DSA does not introduce any new hardness assumptions beyond those inherent
to its underlying hash functions. It builds upon established cryptographic
foundations, making it a reliable and robust digital signature scheme for a
post-quantum world. While attacks on lattice-based schemes such as ML-DSA are
currently hypothetical, if realized they could compromise the security of those
schemes. SLH-DSA would remain unaffected by such attacks due to its distinct
mathematical foundations, helping to ensure the ongoing security of systems and
protocols that rely on it for digital signatures. Unlike ML-DSA, SLH-DSA is not
defined for use in composite certificates and is intended to be deployed directly
in PQC-only certificate hierarchies.

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

Hybrid signature schemes are designed to provide defense in depth during the migration to PQC.
Their goal is to ensure that authentication remains secure as long as at least one of the algorithms
in use remains unbroken. However, several important security considerations arise.

## Downgrade Attacks {#downgrade}

Implementations must ensure downgrade protection so that an adversary cannot
suppress PQC or hybrid schemes and force reliance solely on traditional
algorithms. This is especially important in scenarios where a CRQC is
available but not publicly disclosed. Without downgrade protection, a MitM
attacker could impersonate servers by presenting only traditional
certificates even when PQC certificates are supported.

## Strong Unforgeability versus Existential Unforgeability

In hybrid signature schemes, once one component algorithm
is broken (e.g., the traditional algorithm under a CRQC), the overall scheme
no longer achieves SUF-CMA. While Existential Unforgeability under chosen message attack
(EUF-CMA) (see Section 10.1.1 of {{?I-D.ietf-pquip-pqc-engineers}}) is still
preserved by the PQC component, meaning that an adversary who can obtain signatures on
arbitrary messages still cannot forge a valid PQC signature on any new message that
was not previously signed. The loss of SUF-CMA means that hybrid mechanisms will
have be eventually retired once traditional algorithms are no longer secure.

## Operational Risks

Managing multiple certificate paths (composite, dual, and PQC-only) increases
the risk of misconfiguration and operational errors. For example, a server
might continue using a hybrid signature scheme after the traditional algorithm
is broken, fail to revoke traditional certificates that are no longer secure,
or select the wrong chain for a given client, resulting in clients receiving a
certificate path they cannot validate.

Clear operational guidance and automated monitoring are essential to minimize
these risks. Operators need best practices for certificate lifecycle and
migration planning, along with automated checks to ensure PQC chains remain
present, valid, and not replaced by weaker alternatives.

# IANA Considerations

This document has no IANA actions.

# Acknowledgments

Thanks to Martin McGrath, Suresh P. Nair, Eric Rescorla, and German Peinado for the detailed review.

