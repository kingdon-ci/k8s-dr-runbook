---
description: >-
  There are some important scenarios which are common but they are considered
  outside of the scope of this talk. The known list of scenarios that we
  considered, but decided were non-goals.
---

# Non-Goals

#### Databases and transactional systems

When you do scheduled backups you have to consider a few things in order to build and understand your SLO, and what type of guarantees you can offer with respect to catastrophic losses of the system and its component services.

If your system is transactional in nature, then you may have different requirements. If it's a database then you typically want to keep multiple replicated copies in-flight at once, constantly synchronizing from a lead, or with some consensus algorithm to support the transactional behaviors that your system requires from the database. (We should be able to say with a high degree of certainty that we did not lose a transaction after it was committed, for example.)

This is not a run-book (or a talk) about databases. The kind of backups we will consider here and we are working on today are CSI snapshots. If you have databases now don't get up and start walking away! There will still be plenty for you here in this run-book.

You can take snapshots of your databases with database tools, even when they are hosted in Kubernetes, I'm sure. The difficult trick, historically, was always to acquire a read lock in a non-blocking way first. There are declarative database APIs which now permit users to manage snapshots and store them on CSI volumes. Or perhaps your workloads themselves are stateful and like databases, so they have to be stopped in order to prevent state updates if you want to guarantee the coherency of the state in the system that would be restored from the backups.

Hopefully not. Think happy thoughts, but we don't aim to cover that strategy here for now. It's assumed that stateful workloads can be snapshotted safely at a point-in-time without any such surprises or coherency concerns.

Deployments with databases or database-like behaviors are recommended to keep the resources externally, in the spirit of S3 or [Twelve Factor](https://12factor.net/), which in many cases remains as relevant today as when it was published. In cases where simply retaining periodic snapshots of CSI volumes will not be pervasive enough to provide the service level expectations of a transactional system, look for a different approach, such as high-availability or distributed replication.

#### Multi-cloud deployments

While the approach provided here can be used to enable a migration from cloud to cloud, there is no treatment of multi-cloud in the approach given here. An expectation baked into the approach presented is that costs should remain low, in spite of our availability guarantees.

An SLO sets the expectation that while Bob in Accounting can have backups restored of a file that was deleted from the server some time between four to six weeks ago, he cannot always be so guaranteed that the backups of a file saved to the server and subsequently deleted will keep all data, if the edit and delete events fall too close together between a scheduled snapshot cycle.

Accounting can understand the SLO for backups when it is explained this way, and they can have a fuller understanding of the remedies that snapshotting makes available when they need to test a guarantee. When your guarantee includes replication across regions or across cloud providers, that's when things tend to get more expensive.

We can assume that multi-region deployments within a single cloud provider will still incur mostly the same costs as multi-cloud deployments (bandwidth, data storage, replication costs), if their data graphs are sufficiently connected and replicated to survive a region-wide outage, or for a provider-size outage that affects everything in a given cloud, to fail over to the last man standing.

These costs are not insignificant, and the cost of data replication can be mitigated through two easy levers: reduce the amount of data that needs to be replicated across regions, and reducing the frequency of that replication. A third possibility remains: decreasing the cost of the copy through optimizing and efficient transfer of deltas.

While this remains an interesting area for potential expansion, and hot-failover to another cloud is an attractive and laudable goal, we don't approach this at all.

What _remains in-scope_ is migrating of persistent data from a bare-metal cluster to cloud, or cloud to bare-metal, in whichever order (and it follows that the techniques could be used to migrate from cloud to cloud, with downtime.)

Such migration is possible and remains in scope, as we can show with examples for total account compromise to restore from offsite backups, it is functionally the same as a cloud-to-cloud state migration scenario.

#### Encryption of persistent data volumes

This run-book briefly covers secrets management through some encryption providers, and persistent data volumes with snapshots through a common snapshot interface. There is enough information provided to exfiltrate potentially secret persistent data, where access is permitted.

There might be parts of this guide which need to change in order to support these workflows when clusters are using encrypted persistent volumes. There is no consideration given to the encryption of data in persistent volumes, or the encryption of snapshots or their exports.

If you are handling sensitive or encrypted persistent information and in the course of your work you find that you need to export the data from an encrypted data source, you may find that some adaptations of this guide are needed. Your mileage may vary, proceed at your own risk! 🐲

While having encrypted persistent volumes in the mix is considered a non-goal for the scope of this guide and presentation, feedback and enhancements are always welcomed via PR, and a discussion about the scope of this guide would also be entertained in the spirit of exploring what future presentations might cover, or further iterations of this run-book for any audience at all!
