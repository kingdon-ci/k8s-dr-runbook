# Table of contents

## Architecture Reference

* [Introduction](README.md)
  * [Outline](architecture-reference/introduction/outline.md)
  * [Non-Goals](architecture-reference/introduction/non-goals.md)
* [Architecture and Execution](architecture-reference/architecture-and-execution/README.md)
  * [Create the Cluster](architecture-reference/architecture-and-execution/create-the-cluster.md)
  * [Importing Secrets](architecture-reference/architecture-and-execution/importing-secrets.md)
  * [Persistent Volumes](architecture-reference/architecture-and-execution/persistent-volumes/README.md)
    * [Storage Classes](architecture-reference/architecture-and-execution/persistent-volumes/storage-classes.md)
    * [Persistent Data with Helm](architecture-reference/architecture-and-execution/persistent-volumes/persistent-data-with-helm.md)
    * [Persistent Data the Hard Way](architecture-reference/architecture-and-execution/persistent-volumes/persistent-data-the-hard-way.md)
    * [Creating Snapshots](architecture-reference/architecture-and-execution/persistent-volumes/creating-snapshots.md)
    * [Restoring Snapshots](architecture-reference/architecture-and-execution/persistent-volumes/restoring-snapshots.md)
  * [Going To Production](architecture-reference/architecture-and-execution/going-to-production.md)

## Disaster Recovery Plans (Run-book)

* [Restoring Everything](disaster-recovery-plans-run-book/restoring-everything/README.md)
  * [From retained PVs](disaster-recovery-plans-run-book/restoring-everything/from-retained-pvs.md)
  * [From a snapshot](disaster-recovery-plans-run-book/restoring-everything/from-a-snapshot.md)
  * [From offsite backups](disaster-recovery-plans-run-book/restoring-everything/from-offsite-backups.md)
