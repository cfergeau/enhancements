---
title: Cluster Profiles
authors:
  - "@csrwng"
reviewers:
  - "@abhinavdahiya"
  - "@crawford"
  - "@sdodson"
  - "@derekwaynecarr"
  - "@smarterclayton"
approvers:
  - "@derekwaynecarr"
  - "@smarterclayton"
creation-date: 2020-02-04
last-updated: 2019-02-04
status: implementable
see-also:
replaces:
superseded-by:
---

# cluster-profiles

## Release Signoff Checklist

- [x] Enhancement is `implementable`
- [ ] Design details are appropriately documented from clear requirements
- [ ] Test plan is defined
- [ ] Graduation criteria for dev preview, tech preview, GA
- [ ] User-facing documentation is created in [openshift/docs]

## Summary

Cluster profiles are a way to support different deployment models for OpenShift clusters.
A profile is an identifier that the Cluster Version Operator uses to determine
which manifests to apply. Operators can be excluded completely or can have different
manifests for each supported profile.

## Motivation

To support different a deployment model in which not all operators rendered by
the CVO by default are needed. This includes IBM Public Cloud, in which a
hosted control plane is used. It's also useful for Code Ready Containers which
runs a single node cluster with resource constraints, and needs to scale down
some operands, or disable some unneeded operators.

### Goals

- Equip the CVO to include/exclude manifests based on a matching identifier (profile)

### Non-Goals

- Define which manifests should be excluded/included for each use case

## Proposal

### User Stories

#### Story 1
As a user, I can create a cluster in which manifests for control plane operators are
not applied by the CVO.

#### Story 2
As a user, I can create a cluster in which node selectors for certain operators target
worker nodes instead of master nodes.

### Story 3
As a user, I can create a cluster in which some operands are scaled down to
only one replica to lower resource usage in non-HA setups.

### Design

A cluster profile is specified to the CVO as an identifier in the
`openshift-config/cluster-profile` configmap. For a given cluster, only one CVO
profile may be in effect.

NOTE: The mechanism by which the configmap is created/set is
out of the scope of this design.

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: cluster-profile
  namespace: openshift-config
data:
  profile: crc
```
When the configmap does not exist, or when it does not contain a `profile` key, the `default`
cluster profile is in effect.

The following annotation may be used to include manifests for a given profile:

```
include.release.openshift.io/[identifier]=true
```

This would make the CVO render this manifest only when the
`openshift-config/cluster-profile` configmap has a `profile: crc` key.

Manifests may support inclusion in multiple profiles by including as many of these annotations
as needed.

For items such as node selectors that need to vary based on a profile, different manifests
will need to be created to support each variation in the node selector. This feature will
not support including/excluding sections of a manifest. In order to avoid drift and
maintenance burden, components may use a templating solution such as kustomize to generate
the required manifests while keeping a single master source.
