---
description: Complete Disaster Recovery
---

# Outline

* [Create the Cluster](../architecture-and-execution/create-the-cluster.md) - Declaratively
  * Easy Disaster Recovery for Stateless Clusters
  * Bootstrapping Infrastructure - Public vs. Private
* [Importing Secrets](../architecture-and-execution/importing-secrets.md) - Internal vs. External storage
  * External Secrets Operator
  * Vault OSS
  * External KMS vs Kubernetes Secrets
* [Persistent Volumes](../architecture-and-execution/persistent-volumes/) - Local vs Non-Local
  * Easy CSI from scratch (using Kind and localized storage)
  * CSI at scale (cloud-controller-manager and non-local disk)
* [Storage Classes](../architecture-and-execution/persistent-volumes/storage-classes.md) and Snapshot Classes
* [Persistent Data on GitOps with Helm](../architecture-and-execution/persistent-volumes/persistent-data-with-helm.md)
  * Introducing the Retain strategy for PV reclaim policy
  * PVC rebinding and the existingClaim convention
* [Persistent Data on GitOps without Helm (the Hard Way)](../architecture-and-execution/persistent-volumes/persistent-data-the-hard-way.md)
  * Creating a PV with a StorageClass and Reclaim Policy
  * Binding PV to PVC in a Pod, with a Deployment or StatefulSet
* [Creating Snapshots](../architecture-and-execution/persistent-volumes/creating-snapshots.md) of Local and Non-Local Disks
  * Easy Backups from Local Path Provisioner storage
  * Managing volume snapshots in the same region
  * Moving snapshots off-site (for account-level compromise)
* [Restoring Snapshots](../architecture-and-execution/persistent-volumes/restoring-snapshots.md) (service-level outages)
  * When an original remains intact (for debug/testing purposes)
  * When the original has been compromised or destroyed
* ****[**Going To Production**](../architecture-and-execution/going-to-production.md)****
  * Summary of decisions we must have made
  * On the non-uniformity of environments and Conway's Law
  * Our straw-man "production" environment in the home-lab
    * Prometheus Monitoring
  * Another target for deployment in the cloud
* [Restoring Everything](../../disaster-recovery-plans-run-book/restoring-everything/) (when catastrophe strikes)
  * [From the retained PV](../../disaster-recovery-plans-run-book/restoring-everything/from-retained-pvs.md)
    * (if resource group is fully intact)
  * [From the latest snapshot](../../disaster-recovery-plans-run-book/restoring-everything/from-a-snapshot.md)
    * (if resource group intact, but the PVs were not retained)
  * [From the offsite backup](../../disaster-recovery-plans-run-book/restoring-everything/from-offsite-backups.md)
    * (if resource group is not intact, and the PVs are lost)
