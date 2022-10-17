---
description: >-
  When the account and resource group are fully intact, the persistent volume
  has been retained, and the data has not been compromised, the only problem is
  that PVs are no longer attached.
---

# From retained PVs

This scenario covers what to do when a Persistent Volume is in detached state, and the association with the workload has been compromised.

We assume that you used the [Retain strategy](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#retain) and that either the Persistent Volume Claim (PVC) or Persistent Volume (PV) resource (or both) has been deleted from the cluster.

When the `Retain` reclaim policy is in use, the PV specification dictates that the backing service should not automatically reclaim or delete infrastructure once it has been used for persistence.

You can either define a new PV in the cluster that reuses an existing externally attached disk, or you can modify an existing PV definition in the cluster so that it can be bound again and reused.

* [ ] Verify the `Retain` policy is set
*
* [ ] (Optional) Delete the PVC and PV from the cluster
* [ ] Re-binding from an existing PV
