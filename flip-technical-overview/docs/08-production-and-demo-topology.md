# 08 — Product and Synthetic Environment Boundary

## Why a separate technical environment exists

Flip’s architecture should be inspectable without exposing real community data, production credentials, or administrative authority. A synthetic technical environment exercises the same product contracts with versioned non-sensitive relationships.

<img src="../diagrams/deployment-topology.svg" alt="Product and synthetic deployment boundary" width="900" />

This separation supports public review; it is not the product’s defining architecture.

## Shared behavior, independent authority

Product, synthetic, CI, and local environments can share source version, migrations, domain contracts, job definitions, provider adapters, and scenario definitions.

They do not share the things that confer authority:

- databases or user uploads;
- credentials, encryption/signing material, or sessions;
- queues, PubSub namespaces, or administrative state;
- object/media storage;
- provider keys unless independently provisioned.

An environment identifier is infrastructure configuration. It is not a model-controlled switch and the synthetic environment cannot become a route into product data.

## Synthetic data must preserve relationships

A useful technical environment does more than populate tables. It needs enough relational fidelity to expose the architecture:

- members with different visibility;
- chat with reply structure;
- a forum artifact derived from chat;
- explicit AI-authored content;
- safe citations and artifacts in several lifecycle states;
- a correction or recuration path;
- client synchronization across durable and ephemeral state.

These relationships let a reviewer inspect authorship, permission, provenance, failure, and recovery rather than only a staged happy-path screen.

## The architecture remains active

A public environment should not bypass the contracts it demonstrates. Authentication, authorization, job uniqueness, tool admission, URL and rendering controls, provider-key secrecy, rate/abuse controls, and product persistence remain in force.

The environment may use cheaper routes, reduced concurrency, deterministic provider fakes for selected cases, or restricted media capability. Those differences cannot change the data model or authorship/provenance rules.

## Reproducibility without operational disclosure

Synthetic scenarios should be reconstructible from versioned fixtures and migrations without depending on product snapshots. The public documentation describes the scenario and expected product transitions.

Reset credentials, host commands, private secrets, and deployment choreography are operational material and do not belong in the architecture portfolio.

## What the environment should make visible

A representative review should be able to answer:

- Does internal retrieval obey the invoking actor’s community access?
- Does external research produce inspectable evidence?
- Does curation preserve participant authorship and source navigation?
- Do artifacts show pending, completed, and failed state honestly?
- Do web/native clients converge on committed state?
- Does provider or tool failure degrade explicitly rather than corrupting the product?

The scenario guide is in [demo/README.md](../demo/README.md).

## Endpoint independence

The documentation and diagrams remain the durable public explanation. Temporary endpoint availability or provider configuration should not determine whether the architecture can be understood.