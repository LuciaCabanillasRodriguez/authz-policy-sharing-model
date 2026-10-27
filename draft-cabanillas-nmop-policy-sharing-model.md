---
title: "Model for distributed policy sharing"
abbrev: "policy-sharing-model"
category: info

docname: draft-cabanillas-nmop-policy-sharing-model-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 04
area: "Operations and Management"
workgroup: "Network Management Operations"
keyword:
 - policy
 - distribution
 - lifecycle
venue:
  group: "Network Management Operations"
  type: "Working Group"
  mail: "nmop@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/nmop/"
  github: "LuciaCabanillasRodriguez/policy-sharing-model"
  latest: "https://LuciaCabanillasRodriguez.github.io/authz-policy-sharing-model/draft-cabanillas-nmop-authz-policy-sharing-model.html"

author:
 - name: Lucía Cabanillas Rodríguez
   organization: Telefónica
   email: "lucia.cabanillasrodriguez@telefonica.com"
 - name: Diego López
   organization: Telefónica
   email: "diego.r.lopez@telefonica.com"
 - name: Ana Méndez
   organization: Telefónica
   email: "ana.mendezperez@telefonica.com"
 - name: Pedro Martínez-Julia
   organization: NICT
   email: "pedro@nict.go.jp"

normative:
 RFC2904:
 RFC9052:
 I-D.ietf-opsawg-yang-provenance: I-D.ietf-opsawg-yang-provenance

informative:
 Rego:
  title: A Policy Language for Open Policy Agent
  target: https://www.openpolicyagent.org/docs/latest/policy-language/
 Cedar:
  title: Cedar Policy Language
  target: https://docs.cedarpolicy.com/
 ALFA:
  title: ALFA (Abbreviated Language For Authorization)
  target: https://alfa.guide/alfa-authorization-language/
 authzen:
  title: AuthZEN Policy Store Format
  target: https://datatracker.ietf.org/doc/draft-schwartz-authzen-policy-store/
 OCgNSI:
  title: OpenConfig gNSI Authorization Model
  target: https://www.netconfcentral.org/modules/openconfig-gnsi-authz/2024-02-13/source/raw/
...

--- abstract
This document defines mechanisms and conventions for the representation, lifecycle management, and distribution of  policies in distributed and automated environments. It specifies a consistent, machine-readable, and interoperable framework that enables policies to be validated, exchanged, updated, and removed across heterogeneous systems and administrative domains.

The framework defines how policies, expressed using Policy-as-Code (PaC) languages or other policy languages, are encapsulated and managed using YANG as a canonical representation format. By separating policy lifecycle management from policy semantics, enforcement architectures, and trust frameworks, the model enables a common policy artifact representation that can be reused across diverse implementations and deployment environments.

The model represents policy languages using YANG identities and identity references, allowing new policy languages to be incorporated without modifying the core schema. This approach supports extensibility while preserving a consistent structure for policy metadata, provenance, lifecycle management, and distribution.
--- middle

# Introduction

The increasing complexity and automation of distributed systems, such as programmable networks, multi-cloud platforms, and intent-based infrastructures, require scalable and interoperable mechanisms for managing policies. In these environments, policies are no longer confined to static configuration files or manually managed. Instead, they are dynamic artifacts that must be created, validated, distributed, updated, and eliminated programmatically.

Policies increasingly govern not only access control but also operational behavior, compliance requirements, and governance constraints across multiple areas. These policies may be authored in one area, distributed through another, and enforced in many. As a result, consistent handling of policies throughout their lifecycle becomes critical.

Existing approaches often rely on system-specific policy formats and ad hoc distribution mechanisms, leading to several challenges:

* Fragmentation: Incompatible representations and semantics hinder interoperability and auditability.
* Limited lifecycle control: Many systems lack standardized mechanisms for validation, versioning, and decommissioning.
* Trust ambiguity: In multi-domain environments, it is often unclear whether a policy source is authorized or trustworthy.
* Governance gaps: Systems frequently lack mechanisms to verify whether a policy author is permitted to define policies for a specific domain.

To address these challenges, this document defines a structured and interoperable framework for representing and managing policies as governed artifacts. YANG is used as the canonical representation format for policy artifacts. A YANG-defined policy encapsulates its metadata together with embedded policy content, which is treated as opaque executable logic. This structured representation enables schema-based validation, provenance verification, and interoperable lifecycle management while remaining agnostic to the underlying evaluation engine.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 [RFC2119] [RFC8174] when, and only when, they appear in all capitals, as shown here.

* Policy: A rule or set of rules that define behavior, access, or operational constraints within a system.
* Authorization policy: A policy that governs access or permissions based on user/agent, resource, or environmental attributes.
* Policy lifecycle: The set of stages through which a policy passes, including creation, validation, distribution, update, rollback, and removal.
* Policy-as-Code (PaC): A paradigm in which policies are represented as declarative code artifacts, allowing automation, versioning, and testing.

# Requirements for Policy Management

Systems that manage or exchange policies across domains MUST satisfy the following requirements:

* Policy expressiveness: The framework SHOULD support the representation and distribution of policies with varying levels of expressiveness and granularity, depending on the capabilities of the policy language and evaluation environment.
* Lifecycle control: Policies SHOULD support creation, validation, update, rollback, distribution, and removal.
* Interoperability: Policy representations SHOULD be portable and interpretable across different administrative areas.
* Verifiability: The framework SHOULD provide mechanisms to verify policy integrity and provenance.

# Policy Languages

Policies can be expressed using a variety of declarative policy languages and policy frameworks, including Policy-as-Code (PaC) languages such as Rego, Cedar, and ALFA, as well as other policy languages such as XACML.

Although these technologies differ in syntax, evaluation models, execution environments, and expressive capabilities, they share common lifecycle, governance, provenance, and distribution requirements when used in distributed and multi-domain environments.

The objective of this framework is not to standardize or promote a particular policy language. Instead, it provides a common artifact format for representing, governing, and distributing policies independently of their internal semantics. As a result, the same lifecycle-management and distribution mechanisms can be applied to policies expressed in different languages.

This framework treats policy content as opaque executable logic embedded within a YANG-defined policy artifact. The YANG representation does not interpret, validate, or constrain the internal semantics of the policy language. Instead, it provides a structured and interoperable container for lifecycle governance, version control, provenance binding, and distribution. 

Policy languages are expected to evolve over time. For this reason, policy languages are represented using YANG identities and identity references, allowing new policy languages to be introduced without modifying the base schema.

Rego is used throughout this document for illustrative purposes only. The artifact model is intended to support any policy language represented through the policy-language identity hierarchy.

Example in Rego syntax:

~~~
package example
# Allow read access if the user has the "read" role
default allow = false
allow if {
    input.user.role == "read"
}
~~~

The policy logic above is treated as opaque content by the YANG representation. Its evaluation behavior is determined exclusively by the target execution environment, while lifecycle and governance properties such as the ownership is managed independently.

# Policy representation in YANG

The YANG structure defined in this document does not model a policy repository or datastore. Instead, it defines the representation of a single policy artifact exchanged between lifecycle-management components, such as a Policy Administration Point (PAP) and a Policy Decision Point (PDP), during policy creation, update, rollback, or removal operations.

Consequently, the YANG container represents one policy artifact in transit rather than a collection of policies. The organization of multiple policies within a datastore or repository is implementation-specific and outside the scope of this document.

YANG provides a structured and schema-driven mechanism for representing policies as managed and governed artifacts. In this framework, YANG serves as the canonical container format for policy definitions, encapsulating:

* Policy metadata, including owner, author, origin, area, and description.
* The language used to express the policy logic.
* The embedded policy content, treated as opaque data.
* Optional leaf for validation and provenance.


In addition, each policy instance MUST include an explicit owner attribute that identifies the authority responsible for the policy definition, ensuring accountability within and across domains. By associating a policy with a clearly identified authority, the framework enables governance controls and allows systems to determine whether the policy source is authorized within a given scope. The owner attribute MUST be expressed as a URI that uniquely identifies the authoritative entity responsible for the policy.

The author and origin fields complement the owner. The author field identifies the entity that created the policy. The author will be different from the owner of the policy when the owner of the policy outsources the authorship to other entity. The origin field identifies the object that precedes the policy. Such object can be other policy (more or less general) or extrinsic structure (e.g., the document gathering the administrative operational requirements or a network intent). The author field MUST be fulfilled whereas the origin field is only required when a policy derives from other object.

The area field MUST be filled. It identifies the operational environment of the policy. The area will be used by both the PDP and PEP to restrict the respective decision and enforcement of the policy. An area will define, for example, the outbound walls (boundaries) to which the PEP will limit its reach and focus, as well as the inbound walls (opposite) that will be particularly observed by the PEP to ensure no extrinsic behavior is present within the operation boundaries of the policy. Similarly, the PDP will make use of the area to determine the object, within which it conducts its deliberations, particularly concerning the involved parameters and rules.

The structure of the objects linked by the owner, author and origin fields will be subject to their own schemas. They will be specified separately in ulterior documents.

The policy language is represented using YANG identities and identity references rather than fixed enumerations. This design follows common extensibility practices in YANG models, enabling future languages to be introduced dynamically without requiring modifications to the base schema.

The specific URI structure is determined by the administrative environment. Ownership metadata MAY be cryptographically linked to the policy provenance, enabling verification against the provenance signature key. This mechanism ensures that the policy origin and integrity can be independently verified and trusted.

The stable elements are the YANG-based artifact structure, provenance bindings, and policy metadata. These elements are independent of the policy evaluation logic and remain unchanged regardless of the policy language or deployment environment.

Similarly, the artifact does not assume any specific enforcement architecture. Centralized, distributed, hierarchical, or domain-specific PDP/PEP deployments can consume the same policy artifact representation.

The model also does not mandate a particular trust framework. While ownership, authorship, origin, and provenance information can be conveyed within the artifact, the interpretation of that information and the resulting trust decisions remain deployment specific.

The YANG model below illustrates a simplified structure for representing authorization policies as managed artifacts:

~~~
module example-policy {
  namespace "urn:ietf:params:xml:ns:yang:example-policy";
  prefix policy;

  import ietf-inet-types {
    prefix inet;
  }

  import ietf-yang-provenance {
    prefix iyangprov;
  }

  organization
    "IETF NMOP Working Group";

  contact
    "WG Web:   <https://datatracker.ietf.org/wg/nmop/>

     WG List:  <mailto:nmop@ietf.org>

     Authors:
       Lucía Cabanillas
       <mailto:lucia.cabanillasrodriguez@telefonica.com>

       Diego López
       <mailto:diego.r.lopez@telefonica.com>

       Ana Méndez Pérez
       <mailto:ana.mendezperez@telefonica.com>;

       Pedro Martínez-Julia 
       <mailto:pedro@nict.go.jp>";

  description
    "Illustrative YANG model for representing distributed policies as managed artifacts";

  revision 2026-07-30 {
    description
      "Fifth revision";
    reference
      "RFC XXXX: Model for distributed
       policy sharing";
  }

  /*
   * Identities
   */

  identity policy-language {
    description
      "Abstract base identity for policy languages";
  }

  identity rego {
    base policy-language;
    description
      "Rego policy language";
  }

  identity cedar {
    base policy-language;
    description
      "Cedar policy language";
  }

  identity alfa {
    base policy-language;
    description
      "ALFA policy language";
  }

  identity xacml {
    base policy-language;
    description
      "XACML policy language";
  }

  /*
   * Typedefs
   */

  typedef policy-language-ref {
    type identityref {
      base policy-language;
    }
    description
      "Reference to a policy language identity";
  }

  /*
   * Policy model
   */

  container policy {
    description
      "Container representing a policy artifact";

    leaf area {
      type inet:uri;
      mandatory true;
      description
        "Administrative or operational area to which the policy belongs";
    }

    leaf description {
      type string;
      description
        "Optional human-readable description of the policy";
    }

    leaf language {
      type policy-language-ref;
      mandatory true;
      description
        "Specifies the policy language used to express the policy";
    }

    leaf policy-content {
      type string;
      mandatory true;
      description
        "Opaque policy content expressed in the language identified by the language leaf";
    }

    leaf owner {
      type inet:uri;
      mandatory true;
      description
        "URI identifying the authoritative entity responsible for this policy";
    }

    leaf author {
      type inet:uri;
      mandatory true;
      description
        "URI of the entity that authored the policy";
    }

    leaf origin {
      type inet:uri;
      description 
        "URI of the entity that originated this policy";
    }

    leaf policy-provenance {
      type iyangprov:provenance-signature;
      description
        "Cryptographic provenance signature associated with the policy";
    }
  }
}
~~~

This YANG snippet demonstrates how policy content can be represented as structured data while keeping the logic in a declarative format. By explicitly indicating the language, management systems can validate and process policies appropriately, enabling interoperability between tools and engines.

## Example JSON Encoding

The following example illustrates how a policy instance can be represented using JSON encoding for the YANG model:

~~~json
{
  "example-policy:policy": {
    "area": "urn:example:server-dmz-layer1",
    "description": "Policy example: Allow read access for 'reader' role and write access for 'admin'",
    "language": "policy:rego",
    "policy-content": "package policy\n\ndefault allow = false\n\nallow if{\n    input.user.role == \"reader\"\n}\n\nallow_write if{\n    input.user.role == \"admin\"\n}",
    "owner": "urn:example:user:lucia",
    "author": "urn:example:policy-creator",
    "origin": "urn:example:administrative-operational-requirements"
  }
}
~~~

In this example, the `language` leaf references the `rego` identity defined in the YANG module through an `identityref`. Additional Policy-as-Code languages may be introduced by defining new identities derived from the `policy-language` base identity.

It also extends the rego policy with metadata that describes it, specifies that it is owned by "urn:example:user:lucia", created by "urn:example:policy-creator" (so it was outsourced by the author), originated from the requirement documents identified as "urn:example:administrative-operational-requirements", and scoped to the boundaries determined by "urn:example:server-dmz-layer1".

# Architecture Overview

Policy management in distributed environments relies on a set of functional components that cooperate to define, govern, distribute, evaluate, and enforce policies across administrative areas {{RFC2904}}. This framework adopts that functional separation while introducing structured policy representation and lifecycle governance based on YANG.

## Functional Roles

Policy Author: The entity (human or automated system/agent) responsible for creating the policy definition. The author produces a YANG-encoded policy document that includes metadata (description, area, language, owner) and the actual declarative rule.

Policy Administration Point (PAP): The Policy Administration Point manages the lifecycle and governance of policy artifacts. Upon receiving a YANG-encoded policy, it performs schema validation and, where applicable, verifies provenance using cryptographic mechanisms such as those described in {{I-D.ietf-opsawg-yang-provenance}}.

The PAP MAY maintain historical policy revisions through external version-control systems in order to support rollback, auditing, and policy recovery operations.

Before accepting or distributing a policy artifact, the Policy Administration Point (PAP) MAY perform an authorization verification step. In this step, the PAP queries a Policy Decision Point to determine whether the declared owner or submitting entity is authorized to define or modify policies within the relevant area.

All lifecycle events, including creation, update, activation, rollback, and decommissioning, MAY be recorded in an append-only accounting ledger to ensure traceability and auditability.

Once validated and authorized, the PAP distributes the executable policy artifact to the appropriate decision components.

Policy Decision Point (PDP): The Policy Decision Point evaluates policy logic at runtime. It receives validated policy artifacts and processes requests based on contextual attributes, claims, or environmental information. The PDP produces decisions such as Permit or Deny, optionally including obligations or advice.

Policy Enforcement Point (PEP): The PEP enforces the decisions issued by the PDP. Enforcement may include granting or denying access, applying configurations, or triggering operational actions.

## Functional Interaction

The interaction among the architectural components reflects a strict separation between policy governance and runtime decision-making. Policy artifacts are first validated, governed, and recorded as managed entities before being distributed for evaluation.

The following diagram illustrates the logical interaction flow:

~~~
                     Policy Author
                           |
                           |  YANG-encoded policy artifact
                           v
        +-----------------------------------+     +-------------------------+
        | Policy Administration Point (PAP) |---->|    Accounting Ledger    |
        |-----------------------------------|     |-------------------------|
        | - Schema validation               |     | - create / update       |
        | - Provenance verification         |     | - rollback / remove     |
        | - Version enforcement             |     +-------------------------+
        | - Lifecycle state management      |
        |-----------------------------------|
        |  Governance Authorization Check   |
        |  (PAP -> PDP query)               |
        +------------------+----------------+
                           |
                           |  Authorized policy
                           v
              +-----------------------------------+
              | Policy Decision Point (PDP)       |
              |-----------------------------------|
              | - Runtime policy evaluation       |
              +------------------+----------------+
                                 ^
                                 | Request
                                 |
                                 v Decision
              +-----------------------------------+
              | Policy Enforcement Point (PEP)    |
              |-----------------------------------|
              | - Enforcement of decisions        |
              +-----------------------------------+

~~~

## Policy Distribution

This document defines the structure of the policy artifact and its lifecycle semantics but does not mandate a specific transport mechanism for policy distribution.

Policy artifacts MAY be distributed using different protocols and deployment architectures. The choice of distribution mechanism is deployment-specific.

Policy lifecycle operations are represented through management actions performed on policy artifacts:

* Create: introduction of a new policy artifact.
* Update: replacement of an existing policy artifact.
* Rollback: restoration of a previous policy revision.
* Remove: withdrawal of a policy artifact from the receiving domain.

# Relationship to Other Models

## AuthZEN Policy Store
The AuthZEN Policy Store specification defines a portable representation for organizing and storing authorization policy artifacts independently of a specific Policy Decision Point (PDP) implementation {{authzen}}.

The scope of that work is complementary to the model defined in this document. While a policy store addresses the organization and persistence of policy artifacts, this document focuses on the representation, governance and distribution of individual policy artifacts across administrative domains.

In particular, the YANG structure defined in this document represents a single policy artifact exchanged between policy-management components. It does not define the structure of a policy repository or datastore holding multiple policies. Such a repository may use a policy-store model, while the artifact defined in this document may be used as the unit of policy exchange and distribution.

The separation between policy storage and policy distribution allows the two models to evolve independently while remaining complementary.

## OpenConfig gNSI Authorization Model
Existing data models demonstrate that YANG can be effectively used to carry authorization-related information in operational environments.

The OpenConfig gNSI authorization model {{OCgNSI}} defines a YANG module that represents metadata associated with gRPC authorization policies installed on network devices. That model focuses on device-level state and observability, including policy metadata, operational status, and evaluation counters.

This document is complementary to that approach. While the OpenConfig model concentrates on operational visibility for a specific enforcement technology, the framework defined here focuses on the representation, lifecycle management, provenance, and distribution of policy artifacts across systems and administrative areas.

# Security Considerations

Ensuring the integrity, authenticity, and provenance of policy data is critical to prevent unauthorized modification. Policies SHOULD include cryptographic protection mechanisms that allow their origin and validity to be verified.

The mechanisms defined in {{I-D.ietf-opsawg-yang-provenance}} — Applying COSE Signatures for YANG Data Provenance — provide a suitable foundation for these protections. That document specifies how COSE signatures {{RFC9052}} are used to include digital signatures within the YANG data, enabling, in this case, verifiable provenance and ensuring the integrity of the YANG-based distributed policy sharing model proposed in this draft.

When such provenance mechanisms are applied to policy definitions, each policy instance can include a verifiable signature providing proof of origin and integrity of the provided policy. By treating policies as signed and governed artifacts, this framework reduces the risk associated with automated and cross-domain policy exchange, including the additional risks in multi-domain deployments such as determining whether a policy has been modified in transit and whether the policy issuer is authorized to define policies applicable to the receiving area.

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}

This document is based on work partially funded by the iTrust6G project (Grant agreement N 101097083) and the ROBUST-6G project (Grant agreement N 101139068).
