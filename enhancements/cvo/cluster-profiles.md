---
title: Cluster Profiles
authors:
  - "@csrwng"
reviewers:
  - TBD
approvers:
  - TBD
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

In order to support different deployment models in which not all operators rendered by
the CVO by default are needed, a mechanism is needed by which we tell the CVO which 
manifests to include or exclude. One such deployment is the hosted control plane case
in which the control plane components and corresponding operators should not be rendered
by the CVO.


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

### Design

A cluster profile is specified to the CVO as an identifier in an environment
variable. For a given cluster, only one CVO profile may be in effect.

```
CLUSTER_PROFILE=[identifier]
```
This environment variable would have to be specified in the CVO deployment. When
no `CLUSTER_PROFILE=[identifier]` variable is specified, the default cluster profile
is in effect.

The following annotations may be used to include/exclude manifests for a given profile:

```
exclude.release.openshift.io/[identifier]=true
```
This would exclude the manifest from rendering when `CLUSTER_PROFILE=[identifier]`
has been specified. For the default cluster profile, manifests that contain only
exclude annotations are always included.

```
include.release.openshift.io/[identifier]=false
```
This would make the CVO render this manifest only when `CLUSTER_PROFILE=[identifier]`
has been specified. For the default cluster profile (no `CLUSTER_PROFILE` variable), 
manifests that have any include annotation will always be excluded.

Manifests may support inclusion/exclusion of multiple profiles by including as many of
these annotations as needed.

For items such as node selectors that need to vary based on a profile, different manifests
will need to be created to support each variation in the node selector. This feature will
not support including/excluding sections of a manifest. In order to avoid drift and 
maintenance burden, components may use a templating solution such as kustomize to generate
the required manifests while keeping a single master source.
