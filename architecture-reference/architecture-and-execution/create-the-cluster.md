---
description: >-
  Notes about cluster creation and any decisions made for disaster recovery and
  preparedness.
---

# Create the Cluster

## Terraform Plan and Apply

The cluster creation will start with a Terraform plan.

We've used Azure AKS for our example, but you could use any cloud host cluster (and while this documentation will consider both Cloud provider clusters and local+ephemeral clusters like Kind clusters, the expectation is that you will have some production system which is not ephemeral.)

The terraform plan allows us to encapsulate the provisions and provisioning for our cluster.

We will start with the Production cluster as a target: for this guide, we have chosen Microsoft Azure and the Azure Kubernetes Service (AKS) as it met our requirements for production. We have chosen Terraform as the configuration management layer to provision this cluster, and we've decided to kick off the terraform plan and apply manually, to reconcile when changes are made.

There are a few **Terraform resources to be aware of**, and a few directives to know that may have an outsized impact later, so let's have a look at the [example (straw-man) Production environment](https://github.com/kingdonb/learn-terraform-provision-aks-cluster)!

\[TODO: Video tour of Production, with instance workloads running on it – three clusters in three regions with resource groups and persistent volumes provisioned from AKS, across each region.]

In the terraform plan linked above, there are (`aks-cluster-1.tf`, `aks-cluster-2.tf`, `aks-cluster-3.tf`) AKS clusters in three regions, and (`azure-rg.tf`) resource groups that host a cluster in each region. We use a [SystemAssigned Identity](https://github.com/kingdonb/learn-terraform-provision-aks-cluster/blob/8e11ac597a91296a5ed5c1943c4787c5be02601d/aks-cluster-1.tf#L22-L24) and have [enabled auto-scaling](https://github.com/kingdonb/learn-terraform-provision-aks-cluster/blob/8e11ac597a91296a5ed5c1943c4787c5be02601d/aks-cluster-1.tf#L12-L14) on our clusters, and there is almost nothing else notable about this Terraform AKS cluster configuration.

It is mostly derived from the [Provision an AKS Cluster guide](https://learn.hashicorp.com/tutorials/terraform/aks) provided by Hashicorp Learn.

**The \_resource group**\_\*\* is configured to\*\* [**prevent destroy**](https://github.com/kingdonb/learn-terraform-provision-aks-cluster/blob/8e11ac597a91296a5ed5c1943c4787c5be02601d/azure-rg.tf#L7-L9)**,** which could be important later on. (_Hint:_ since our persistent volumes are in a resource group, destroying the resource group could wipe the persistent volume data and even snapshot data, making DR even just a bit more difficult.)

This document will not go in-depth about the cluster creation workflow itself, since it is covered elsewhere, other than to present it as a brief video supercut which will soon be posted here:

\[TODO: Add Video for "Create the Cluster" with Terraform and AKS]

### Terraform Destroy

Because the `prevent_destroy` directive is used on the resource group, `terraform destroy` will balk and quit unless the `-target` parameter is given, making the destroy command more specific. If the complete "disaster" recovery scenario is being engaged, then the directive can be set to `false`, or removed entirely, allowing the destroy to proceed with wiping out everything.

_Be quite sure you understand the implications of this setting_ **before** _disengaging this safeguard!_

\[TODO: Add Video for "Destroy the Cluster" and "Destroy Everything"]

## Home Lab Alternatives

In our home lab, we use physical nodes which are homogenous Linux machines, often running whatever flavor or distribution of the month (Debian, Red Hat, Ubuntu, ...) and since we are _lean_ we can _lean into this decision_ by taking direct advantage of the persistence of the operating system's built-in filesystem. We do not have fancy block stores, or network attached volumes.

Oh no... we've got `/opt/local-path-provisioner` and a GUID generator, somebody stop us!

### Local Path Provisioner

We can use kubectl, kustomize, or helm to install the [Local Path Provisioner](https://github.com/rancher/local-path-provisioner). The amount of time we saved making this choice has easily paid for the decision, as it is the choice for our staging cluster, where an outage or fault is implicitly without cost. It does not make backup and restore any harder, in fact this actually makes things much easier, as we can simply [take a periodic copy](https://medium.com/swlh/using-rclone-on-linux-to-automate-backups-to-google-drive-d599b49c42e8) of the directories in `/opt/local-path-provisioner` and following that, we are done with disaster recovery prep work. Almost...

The local-path StorageClass is enabled on our cluster, and it uses the `WaitForFirstConsumer` volume binding mode.

This means that persistent volumes will not be filled by the provisioner before a pod is scheduled, so a claim can be located on the same node as the pod. This means that pods with persistent volume attachments are bound to a given node "forever" – or as long as the PV is persisted there.

They cannot be quickly migrated from one node to another, as their dynamic assignments have been statically bound. This is a limitation we are prepared to accept in our Home Lab. It is not a good solution for production, (but we get there when we get there!)

\[TODO: BREAKOUT - A note about a convention used in this runbook. Contrary choices made for Production environments are always presented later, at the bottom of each page. We should be in Staging before we are in Production, so that is the order we talk about them in this book.]

This means that when we backup and restore our staging cluster, we do need to take care that volumes are restored to the same node where they were backed up, or else these static bindings are updated so that pods can be rescheduled and newly bound to wherever the restored data has been recovered.

But if we are backing up and restoring our staging cluster, we are either tilting at windmills or have made some poor decisions, because backup and restore operations are for production. In staging, we will instead prefer to simply Retain the volumes, and prevent them from being deleted. If they host important persistent data, then there should always be a plan to reproduce it from scratch because this is Staging.

### Kubeadm

Our homogenous Linux machines all have a supported installation path on [Kubeadm](https://kubernetes.io/docs/reference/setup-tools/kubeadm/), the reference implementation of a Kubernetes distribution. While kubeadm administration for your home lab is currently beyond the scope of this guide, you can use [Kind (Quick Start)](https://kind.sigs.k8s.io/docs/user/quick-start/) for an equally good experience that is both completely based on Kubeadm and also ephemeral.

Kubeadm clusters are easy to manage at first, but there are several drawbacks to doing this manually, one of which is that we must manually join each physical node to the cluster after the cluster is provisioned. If we have many nodes and many clusters, this can become tedious.

When cluster creation and deletion is used by robotic processes, like a pull-request builder that runs test code in isolation, something slightly more streamlined is also needed.

### VCluster + K3s

[VCluster](https://www.vcluster.com/) is a virtual Kubernetes cluster that runs inside a regular namespace, and we decided this was a good choice for our staging "pet" environments as well as for ephemeral clusters. The goal was to keep important parts hermetic, like the Harbor instance and localized S3-compatible storage buckets; they should be easy to back up and restore, or create from scratch. There is no retention requirement for our Harbor instance, (we'd just prefer to keep it rather than wiping and reconfiguring every time that Staging is recreated.)

VCluster has a fast lifecycle workflow for iterating quickly on the design of this disaster recovery guide. If your cluster takes longer to provision and destroy, then you will certainly do it with a lesser frequency. We want to encourage our dev team to destroy and recreate staging frequently, so it will be less painful whenever we need to repeat this process in production.

We needed to know that we could delete and recreate clusters smoothly in order to produce this demo, and that meant we'd be destroying a lot of test environments! VCluster was more practical for testing; we went from a weekly drill with our staging cluster, to this example disaster recovery trial that works from end-to-end in less than 5 minutes.

{% embed url="https://www.youtube.com/watch?v=1ehRYlsKhYM" %}
Supercut - Disaster Recovery in Staging with FluxCD
{% endembed %}

We've got this supercut that shows the whole thing in under 45 seconds, but it should be clear from watching that this is a non-trivial cluster with more than just a handful of demo-app services and production-targeted services across a wide variety of Cloud Native vendors and projects.

VCluster was absolutely perfect for our Home Lab's "Production" and "Staging" instances.

We can allow our child clusters [pass-through access to the storage class](https://www.vcluster.com/docs/architecture/storage#sync-persistent-volumes) in our "Production" kubeadm cluster, syncing persistent volumes they create to the parent cluster and permitting privileged access to the physical nodes, so pet clusters can create persistent volumes with a `hostPath` (or we can prohibit all of that, but for our purposes it's exactly what we need.)

### All Together

We had enough physical nodes for one cluster, but we needed that cluster to host persistent services like database and authentication, services that provide service to other clusters with varying service levels of their own, making frequent tear-down and recreation of the entire cluster in our Home Lab an impractical target.

But now we can simulate a close approximation.

The VCluster "Staging" pet has persistent storage that's linked to the parent "Production" cluster, and when our "Staging" cluster goes down, any volume definitions are expected to be retained on the parent cluster unless their reclaim policy was set to `Delete` (but there will be more on retention and the reclaim policy when we get to [Persistent Data with Helm](persistent-volumes/persistent-data-with-helm.md), later on in the guide!)

We still needed a complete solution for disaster recovery that we could use to tear down and recreate the "straw-man" environment, even if it wasn't going to production, because we might need to use it for services that should have as close as possible to a production-grade SLO.

Examples are Harbor, Chartmuseum, Hephy Workflow, Ingress controllers and HTTP routers, Keycloak, an OIDC Proxy for Kubernetes authenticating with Keycloak or something else, MetalLB, Minio for storage, a VPN service providing remote access to the Home Lab internal network, ... these are all services which we do not want to lose access to during our Disaster Recovery trials. They may be required to complete the disaster recovery itself, or to gain access in an emergency.

\[TODO: Video tour of our Kubeadm + VCluster + Local Path Provisioner storage in the Home Lab]

We can enable all of these services on a VCluster, and support them with external databases hosted on the parent cluster provided by KubeDB. (Thanks to AppsCode for their generous donation of a KubeDB Enterprise license for Team Hephy!)

These services on all levels are backed with the same local-path provisioner, that we mentioned before can be backed up [in the traditional way, with cron and rclone](https://medium.com/swlh/using-rclone-on-linux-to-automate-backups-to-google-drive-d599b49c42e8), or another similar approach.

\[TODO: Video demo of backup and restore with cron and rclone]

These are snapshots too, but they are not CSI volume snapshots. We will spend time in [Persistent Volumes](persistent-volumes/) exploring the mechanisms that CSI provides that supersedes node-local storage when cloud providers are engaged. There are many options, and while node-local storage as an option does not strictly go away, the requirement to use a `hostPath` volume in a `privileged` context makes this choice a bit more unlikely to be accepted in a multi-tenant or production setting.

You can still use local storage, but remotely attached block storage can provide a more refined interface, an abstraction over more efficient snapshotting backend that can be linked to policy, and generally graduating to use modern CSI drivers is the point of this disaster recovery guide!

## Flux Bootstrap

Now assuming that you've made your choices above between either the Home Lab or Production approach, and you have your cluster ready, we'll use Flux Bootstrap to replace the stateless, declarative resources on the cluster. Stateful volumes have a stateless definition, so whether your cluster has stateful volumes or not, this part of the process is exactly the same.

We will prepare a deploy key on the cluster and apply it through the GitHub API for Deploy Keys as a built-in part of the bootstrap command.

We can run `flux bootstrap github` manually, or we can use the Terraform provider that implements Flux bootstrap. There are many demos that run `flux bootstrap` and it is well documented how to use the `flux` CLI for this task, so we will prefer instead to let Terraform perform the bootstrap of Flux on the cluster, getting this task out of the way without adding any manual steps to the disaster recovery process.

\[TODO: Video of Flux Bootstrap with Terraform, single cluster]

### Flux Bootstrap using Terraform

While the [terraform provider](https://github.com/fluxcd/terraform-provider-flux) is developed in a separate repository than the `flux` CLI, it implements basically the same workflow and either works well.

If you aren't using terraform, just use `flux bootstrap` instead, as the result won't be any different. The advantage using Terraform is that we can just as easily provision three clusters using Flux provider to bootstrap each, all in the same Git repository or in separate repositories.

We will quickly bootstrap three clusters in different regions with the Flux terraform provider.

This example uses multiple clusters, so the terraform provider is a big advantage here, but there's no architectural reason why we need to use multiple clusters. Later on, it seems we can also use [Terraform to provision Vclusters](https://loft.sh/blog/loft-terraform-provider/)! That is out of scope for this guide, as we won't be using VCluster in [Going To Production](going-to-production.md). But we are using Terraform in production, so here's the same example expanded to work on those three different clusters.

\[TODO: Implement the Multi-Cluster example]
