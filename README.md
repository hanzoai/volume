# Volume

Node-local persistent volumes for a Hanzo cluster.

A PersistentVolumeClaim on one of these storage classes gets a directory on the
node its pod lands on. That is the whole behaviour, and it is the right one for
a disk: fast, no network in the path, nothing to operate.

```
provisioner   hanzo.ai/volume
data          /var/lib/hanzo/volume
config        the hanzo-volume-config ConfigMap
```

## What this is not

**It is not durability.** A volume here lives on exactly one node, and a node
that goes takes its volumes with it. Durability is a separate concern and has
its own answer: `hanzoai/replicate` streams a tenant's SQLite to object storage
and restores it on a miss, so a service can come back on another node with its
data. Disk and durability are different questions; keeping them apart is what
lets the disk stay simple.

A storage class named for network-attached block storage but provisioned by this
is a lie an operator will eventually act on. Name a class for what it is.

## Why it is ours

It is a fork of Rancher's local-path-provisioner (see NOTICE), and the fork is
about identity rather than behaviour:

  - **`hanzo.ai/volume`**, so a stock k3s `rancher.io/local-path` on the same
    cluster cannot answer for our claims. Two provisioners sharing one name race
    for the same PVCs, and which one wins is not something a cluster should
    decide by chance.
  - **`/var/lib/hanzo`**, because an operator reading a Hanzo cluster should not
    have to learn another vendor's directory to find their data.

## Build

```
make            # binary
go test ./...   # tests
```
