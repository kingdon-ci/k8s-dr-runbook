---
description: GitOpsCon / KubeCon / Cloud Native Con 2022 - North America - Detroit, MI
---

# Introduction

This is a beginner's guide to disaster readiness for Kubernetes and GitOps practitioners. These topics are, broadly speaking, not beginning topics; but we present them here for beginners!

The content assumes a bit of familiarity with Kubernetes and Cloud Native topics. The goal of this text is to build context around the utilities that CSI provides for persistence in Kubernetes natively first, then present one successful implementation with pragmatic decisions for an implementation that you can follow at home as a "straw-man Production."

#### From Straw-Man to Production

This pragmatism comes with drawbacks like any good straw-man, owing to limitations in a Home Lab setting. Taking inventory of these boundaries, we used our Home Lab to validate the ideas in a Staging setting, and then wrap back to revisit those limitations with [Going To Production](architecture-reference/architecture-and-execution/going-to-production.md).

Two orthogonal approaches (Home Lab vs. Cloud) both are presented as opposing ideas, but the CSI model is the same in both environments. Snapshots are available depending on your CSI driver or cloud provider for backup and restore; this guide presents a complete approach to disaster recovery for Home Lab and Cloud, with approaches that can be compatible with either.

The decisions are presented with our choice (Home Lab) vs our preference or recommendation for production, with both options presented together as a run-book, more frequent backups that dial up the overhead costs in order to match our elevated Service Level expectations.

The expectation of this guide is that we can take our Home Lab cluster, with persistent data and all, then migrate it to the cloud and back. We will learn about several different storage classes and the current state-of-the-art in CSI facilities for managing snapshots, as well as different strategy and use cases for either local or non-local data volumes in the K8s CSI model.

Presented with full examples and a model cluster for Home Lab and for Cloud, this run-book aims to assuage the whole gamut of disasters that can befall your K8s cluster, and provides a pathway to recovery with scenarios. Topics responsive to a range of issues from malware and ransomware encryption scams, or untimely accidental deletion of stateful sets, including volumes and secrets and persistent volume data.

After the [Architecture and Execution](architecture-reference/architecture-and-execution/) run-books, we have checklists and disaster recovery plans with a run-book for [Restoring Everything](disaster-recovery-plans-run-book/restoring-everything/). These are structured and intended to be appropriate for reference or copy, to make ready use in live action incident recovery plans and planning.

### Elevator Pitch

Whether it's hackers who turned your servers into a cryptographically secured paperweight, or it's Bob in Accounting that has unfortunately deleted some important files from the accounting server nearly eight weeks ago, and he needs them now for a critically important filing deadline; we can provide service-level guarantees for availability that are resilient to these and other instance types of failure.

The run-book aims to show how to protect the business from harm, by treating scary disaster type failure scenarios as likely probabilities which can only be limited but never fully mitigated, and engineering for successful outcomes through clearly documented recovery protocols for when the unlikely failure scenario happens in production, or at the worst possible time.

## Abstract

Stateful workloads present a heavy challenge for platform operators to secure, backup, and provide for their recovery in the event of "black swan" or cluster-ending catastrophes.

Developing a strategy for CSI volumes and managing snapshots should be a priority for cluster operators who otherwise may leave their important data vulnerable to ransomware attacks or other threats; what mitigations can be applied in the event of a full compromise?

With GitOps, managing recovery of STATELESS workloads is made easy, almost effortless. It hasn't been shown before how that level of convenience is replicated for STATEFUL workloads and DR on persistent volumes.

Cluster operators may already know enough about CSI to understand that PVCs are backed by PVs and have a ReclaimPolicy that may be set to Retain.

How does one manage this persistence with GitOps? How to manage snapshots? What exactly does snapshot restore executed via GitOps approach look like?

Let’s endeavour now to offer clear prescriptive guidance about effective GitOps strategies for managing disaster recovery.

What are some strategic approaches to apply to bring our risks back within tolerance? We will find out more in this guide, as we explore the important lessons in the form of a runbook for Kubernetes CSI disaster recovery.

### Background

A stateful cluster is defined here as a cluster with persistent volumes and non-ephemeral data.

You can make your production cluster stateful or stateless. The primary approach recommended here by which prod workloads are made stateless, is by externalizing the stateful components.

Twelve Factor said, "treat backing services as attached resources" and made stateful components into an abstraction behind a resource URI and some authentication secret.

So it follows that you can avoid CSI/PV attachment sometimes, by taking on an external dependency and providing for the backing service. This guide therefore approaches both: persistence through CSI and secret attachment, since your cluster might likely need either or both (secrets and CSI), and persistent data volumes might also contain secrets, we can aim to broadly cover both persistence and secret management in a single consistent approach.&#x20;

If the production environment is designed to be stateless, then once you have finished reading [the next page](architecture-reference/architecture-and-execution/create-the-cluster.md) and actuating it, disaster recovery planning for the scope of this guide is over! When no persistent data volumes are stored on the cluster, there is nothing to back up. It should not be understated that this simplification results in a drastic reduction in complexity, that you should take advantage of a stateless production cluster if you can avoid stateful workloads.

But we cannot always externalize such complexity and it would not make for a very interesting run-book if the story ended there, with "backing services are external" as the whole punchline.

Let's consider a balanced approach to hosting secrets or persistence in a cluster, internally or externally, and what it means to offer backup availability guarantees with point-in-time recovery in a few different models of persistent data storage that are commonly used with Kubernetes.

