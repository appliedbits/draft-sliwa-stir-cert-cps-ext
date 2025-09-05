---
###
# Call Placement Service (CPS) URI Certificate Extension for STIR/SHAKEN Certificates
###
title: "Call Placement Service (CPS) URI Certificate Extension for STI Certificates"
abbrev: "CPS URI Certificate Extension"
category: std

docname: draft-sliwa-stir-cert-cps-ext-00
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Applications and Real-Time"
workgroup: "Secure Telephone Identity Revisited"
keyword:
- stir
- certificates
- delegate certificates
venue:
  group: "Secure Telephone Identity Revisited"
  type: "Working Group"
  mail: "stir@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/stir/"
  github: "appliedbits/draft-sliwa-stir-cert-cps-ext"
  latest: "https://github.com/appliedbits/draft-sliwa-stir-cert-cps-ext"

author:
 -
    fullname: Rob Sliwa
    organization: Somos Inc.
    email: robjsliwa@gmail.com
 -
    fullname: Chris Wendt
    organization: Somos Inc.
    email: chris@appliedbits.com

normative:
  RFC3986:
  RFC5280:
  RFC8224:
  RFC8225:
  RFC8226:
  RFC8816:
  RFC9060:
  I-D.wendt-stir-certificate-transparency:
  X.509:    
    title: "Information technology - Open Systems Interconnection - The Directory: Public-key and attribute certificate frameworks"
    author: 
      - org: International Telecommunication Union
    date: October 2016
    target: https://www.itu.int/rec/T-REC-X.509
    seriesinfo:
      ITU-T: Recommendation X.509, ISO/IEC 9594-8
  X.680:
    title: "Information Technology - Abstract Syntax Notation One (ASN.1): Specification of basic notation"
    author: 
      - org: International Telecommunication Union
    date: August 2015
    target: https://www.itu.int/rec/T-REC-X.680
    seriesinfo:
      ITU-T: Recommendation X.680, ISO/IEC 8824-1
  X.681:
     title: "Information Technology - Abstract Syntax Notation One (ASN.1): Information object specification"
     author: 
      - org: International Telecommunication Union
     date: August 2015
     target: https://www.itu.int/rec/T-REC-X.681
     seriesinfo:
      ITU-T: Recommendation X.681, ISO/IEC 8824-2
  X.682:    
    title: "Information Technology - Abstract Syntax Notation One (ASN.1): Constraint specification"
    author: 
      - org: International Telecommunication Union
    date: August 2015
    target: https://www.itu.int/rec/T-REC-X.682
    seriesinfo:
      ITU-T: Recommendation X.682, ISO/IEC 8824-3
  X.683:    
    title: "Information Technology - Abstract Syntax Notation One (ASN.1): Parameterization of ASN.1 specifications"
    author: 
      - org: International Telecommunication Union
    date: August 2015
    target: https://www.itu.int/rec/T-REC-X.683
    seriesinfo:
      ITU-T: Recommendation X.683, ISO/IEC 8824-4

informative:

--- abstract

This document specifies a non-critical X.509 v3 certificate extension that conveys the HTTPS URI of a Call Placement Service (CPS) associated with the telephone numbers authorized in a STIR Delegate Certificate. The extension enables originators and verifiers of STIR PASSporTs to discover, with a single certificate lookup, where Out-of-Band (OOB) PASSporTs can be retrieved. The mechanism removes bilateral CPS provisioning, allows ecosystem-scale discovery backed by STI Certificate Transparency (STI-CT), and is fully backward compatible with existing STIR certificates and OOB APIs.

--- middle

# Introduction

The STIR (Secure Telephone Identity Revisited) framework provides a means of cryptographically asserting the identity of the calling party in a telephone call by using PASSporTs carried in SIP requests, as defined in {{RFC8224}} and {{RFC8225}}. To support deployment in environments where SIP Identity headers may be removed or are not end-to-end transmittable, such as in non-IP or hybrid telephony networks, the STIR Out-of-Band (OOB) mechanism was introduced in {{RFC8816}}. In OOB scenarios, PASSporTs are published to a Call Placement Service (CPS) where they may be retrieved independently of the SIP signaling path.

To enable discovery of the appropriate CPS for a given telephone number or SPC, this document defines a certificate extension that binds a CPS URI to the identity resources listed in the TNAuthList of the STI certificate. This CPS URI extension provides a verifiable association between a number resource and its corresponding CPS, enabling relying parties to discover CPS endpoints by observing STI Certificate Transparency (STI-CT) logs defined in {{I-D.wendt-stir-certificate-transparency}}.

This specification defines the syntax and semantics of the CPS URI certificate extension, describes how it is encoded in {{X.509}} certificates also defined in {{RFC5280}}, and outlines validation procedures for Certification Authorities and relying parties. This extension is intended to be used in conjunction with existing STIR certificates defined in {{RFC8226}} and delegate certificates defined in {{RFC9060}} infrastructure, and supports enhanced transparency and automation in OOB PASSporT routing.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# The id-pe-cpsURI Certificate Extension

This {{X.509}} extension is non-critical, applicable only to end-entity certificates, and defined with ASN.1 {{X.680}} {{X.681}} {{X.682}} {{X.683}} later in this section.

This extension is intended for use in end-entity STI certificates {{RFC8226}} and delegate certificates {{RFC9060}} that include TNAuthList values authorizing the use of specific telephone numbers or Service Provider Codes (SPCs). The CPS URI extension provides a means for the certificate holder to declare the HTTPS endpoint of a Call Placement Service (CPS) defined in {{RFC8816}} that can be used to publish or retrieve PASSporTs for the covered resources.

The presence of this extension allows relying parties to discover the CPS associated with a given telephone number without relying on static configuration or bilateral agreements. This facilitates scalable and verifiable Out-of-Band PASSporT delivery as defined in {{RFC8816}}, using information already published in publicly logged STI certificates.

The extension is encoded as an IA5String containing an absolute HTTPS URI and is identified by an object identifier (OID) assigned in the PKIX id-pe arc. Additional details about the encoding, semantics, and validation rules for the CPS URI are defined in the sections below.

## ASN.1 Module Syntax

The extension ASN.1 module is defined as follows:

~~~~~~~~~~~~~
CPS-CERT-EXTENSION DEFINITIONS EXPLICIT TAGS ::=
BEGIN

id-pe OBJECT IDENTIFIER ::=
  { iso(1) identified-organization(3) dod(6) internet(1)
    security(5) mechanisms(5) pkix(7) 1 }

id-pe-cpsURI OBJECT IDENTIFIER ::= { id-pe TBD }

CPSURI ::= IA5String

END
~~~~~~~~~~~~~

Certificates containing a CPSURI that is not an absolute HTTPS URI as defined in {{RFC3986}} MUST be considered invalid by relying parties.

Note: The numeric assignment TBD is temporary. IANA will allocate a permanent arc under "PKIX SubjectPublicKeyInfo Certificate Extensions" during RFC publication.

## Extension Semantics

The IA5String value MUST be an absolute URI {{RFC3986}} that:

- Uses the "https" scheme.
- Identifies the root of the CPS HTTPS API interface (e.g., "https://cps.example.net/oob/v1").

## Criticality

The extension MUST be marked non-critical so that implementations that do not understand it can still validate the certificate.

## Processing Rules

- A STIR Authentication Service (AS), defined in {{RFC8224}}, that holds a Delegate Certificate containing id-pe-cpsURI SHOULD publish OOB PASSporTs to the indicated CPS.
- A STIR Verification Service (VS), defined in {{RFC8224}} that receives a PASSporT signed by such a certificate MAY derive the CPS endpoint by reading the extension, or MAY query an external discovery directory that is populated by monitoring the STI-CT logs.
- If the extension and an external directory disagree, verifiers SHOULD treat the call as unverifiable unless local policy states otherwise.

Relying parties SHOULD ensure that the certificate containing the CPS URI is present in a trusted Certificate Transparency log before using the URI for OOB operations.

# Use with Out-of-Band

Figure 1 shows the message flow when the extension is present:

~~~~~~~~~~~~~
   +------------+ (1) ACME w/ Authority Tokens +-----------+
   | Enterprise |----------------------------->|  CA / CT  |
   +------------+ Delegate Cert w/ CPS URI ext +-----------+
          |                                       |    |
          |      (2) SIP INVITE + PASSporT        |    |
          v                                       |    |
   +-----------+ (3) GET CPS/PASSporT via HTTPS   |    |
   |  VS/CPS   |<---------------------------------+    |
   +-----------+                                       |
          ^                                            |
          |                                   +------------+ 
          +-----------------------------------| CT Monitor |
                                              +------------+ 
Figure 1
~~~~~~~~~~~~~

1. The enterprise obtains a Delegate Certificate containing the CPS URI. The CA submits the certificate to STI-CT.
2. On each call, the AS signs a PASSporT with Delegate Certificate containing SCT.
3. The terminating VS reads the CPS URI from the certificate and fetches the PASSporT.

# Operational Considerations

- Logging: CAs issuing certificates with id-pe-cpsURI MUST submit the certificate to STI-CT logs.
- Rotation: Changing a CPS hostname or path requires certificate re-issuance. Operators SHOULD minimize TTLs on old URIs during migration.
- Monitoring: Relying parties and CPS discovery services SHOULD monitor trusted STI-CT logs for new or updated CPS URI declarations to ensure timely access and detect misconfiguration.

# Security Considerations

The CPS URI certificate extension introduces a mechanism for associating telephone number resources with CPS endpoints through STI certificates. The following considerations apply:

- Misuse or Misissuance: A malicious or misconfigured entity may include a CPS URI in a certificate without authorization for the corresponding TNAuthList resources. Certification Authorities (CAs) MUST validate that the entity requesting the certificate has authority over the listed numbers or SPCs before issuing the certificate.
- URI Integrity: The CPS URI is not digitally signed independently of the certificate. Relying parties MUST validate the entire certificate chain and ensure the certificate is properly logged in a Certificate Transparency log before using the URI.
- Certificate Expiry and Revocation: CPS URI information may become outdated due to certificate expiration or revocation. Relying parties SHOULD evaluate certificate validity and revocation status when interpreting CPS mappings.
- Log Availability and Monitoring: Relying parties that depend on CT log monitoring for CPS discovery SHOULD monitor multiple trusted logs to ensure timely detection of CPS declarations and prevent omission attacks.
- Information Exposure: The publication of CPS URIs in publicly logged certificates may reveal deployment metadata. This exposure is consistent with existing STIR delegate certificate practices and does not introduce additional privacy risk beyond what is already present in TNAuthList usage.

# IANA Considerations

IANA is requested to assign a new object identifier (OID) for the CPS URI certificate extension in the "PKIX Extension Registry" as follows:

- Name: id-pe-cpsURI
- OID: to be assigned
- Description: Certificate extension for specifying a Call Placement Service (CPS) URI for STIR Out-of-Band PASSporTs
- Reference: This document

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
