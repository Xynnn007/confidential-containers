---
name: New project release
about: Checklist for the coming project's release
title: "[Release] Check list for v<TARGET_RELEASE>"
labels: ''
assignees: ''
---

# v<TARGET_RELEASE>

## Overview

The release process mainly follows from this dependency graph.

```mermaid
flowchart LR
    Trustee --> Versions.yaml
    Guest-Components --> Versions.yaml
    Kata --> kustomization.yaml
    Guest-Components .-> Client-tool
    Guest-Components --> enclave-agent
    enclave-cc --> kustomization.yaml
    Guest-Components --> versions.yaml
    Trustee --> versions.yaml
    Kata --> versions.yaml

    subgraph Kata
        Versions.yaml
    end
    subgraph Guest-Components
    end
    subgraph Trustee
        Client-tool
    end
    subgraph enclave-cc
        enclave-agent
    end
    subgraph Operator
        kustomization.yaml
        reqs-deploy
    end
    subgraph cloud-api-adaptor
        versions.yaml
    end
```

Starting with v0.9.0 the release process no longer involves centralized dependency management.
In other words, when doing a CoCo release, we don't push the most recent versions of the subprojects
into Kata and enclave-cc. Instead, dependencies should be updated during the normal process of development.
Releases of most subprojects are now decoupled from releases of the CoCo project.

## The Steps

### Determine release builds

Identify/create the bundles that we will release for Kata and enclave-cc.

- [ ] 1. :eyes: **Create enclave-cc release**

    Enclave-cc does not have regular releases apart from CoCo, so we need to make one.
    Make sure that the CI [is green](https://github.com/confidential-containers/operator/actions/workflows/enclave-cc-cicd.yaml) and then use the Github release tool to create a tag and release.
    This should create a bundle [here](https://quay.io/repository/confidential-containers/runtime-payload?tab=tags).

- [ ] 2. :eyes: **Find Kata release version**

    The release will be based on an existing Kata containers bundle.
    You should use a release of Kata containers.
    Release bundles can be found [here](https://quay.io/repository/kata-containers/kata-deploy?tab=tags).
    There is also a bundle built for [each commit](https://quay.io/repository/kata-containers/kata-deploy-ci?tab=tags).
    If you absolutely cannot use a Kata release,
    you can consider releasing one of these bundles.


### Test Release with Operator

- [ ] 3. :eyes: **Check operator pre-installation**
    
    The operator uses a pre-install container to setup the node.
    Check that the container matches the dependencies used in Kata
    and that the operator pulls the most recent version of the container.

    * Check that the version of the `nydus-snapshotter` used by Kata matches the one used by the operator
        * Compare `nydus-snapshotter` version in Kata [versions.yaml](https://github.com/kata-containers/kata-containers/blob/main/versions.yaml#L291) with the [Makefile](https://github.com/confidential-containers/operator/blob/main/install/pre-install-payload/Makefile#L4) for the operator pre-intall container.
            * If they do not match, update the operator. This can be part of the PR described in the next step.
    
    * Make sure that the operator pulls the most recent version of the pre-install container
        * Find the last commit in the [pre-install directory](https://github.com/confidential-containers/operator/tree/main/install/pre-install-payload)
            * Make sure that the commit matches the preInstall / postUninstall image specified for [enclave-cc CRD](https://github.com/confidential-containers/operator/blob/main/config/samples/enclave-cc/base/ccruntime-enclave-cc.yaml) and [Kata CRD](https://github.com/confidential-containers/operator/blob/main/config/samples/ccruntime/default/kustomization.yaml)
                * If these do not match (for instance if you changed the snapshotter above), update the operator so that they do match. This can be part of the PR described in the next step.
    

- [ ] 4. :wrench: **Open a PR to the operator to update the release artifacts**

    Update the operator to use the payloads identified in steps 1 and 2.

    There are a number of places where the payloads are referenced. Make sure to update all of the following to the tag matching the latest commit hash from steps 7 and 8:
    * Enclave CC:
      * [SIM](https://github.com/confidential-containers/operator/blob/main/config/samples/enclave-cc/sim/kustomization.yaml)
      * [HW](https://github.com/confidential-containers/operator/blob/main/config/samples/enclave-cc/base/ccruntime-enclave-cc.yaml)
    * Kata Containers:
      * [default](https://github.com/confidential-containers/operator/blob/main/config/samples/ccruntime/default/kustomization.yaml)
      * [s390x](https://github.com/confidential-containers/operator/blob/main/config/samples/ccruntime/s390x/kustomization.yaml)
      * [peer-pods](https://github.com/confidential-containers/operator/blob/main/config/samples/ccruntime/peer-pods/kustomization.yaml)
          Note that we need the quay.io/confidential-containers/runtime-payload-ci registry and kata-containers-latest tag

    **Also, update the [operator version](https://github.com/confidential-containers/operator/blob/main/config/release/kustomization.yaml#L7)**
   
### Final Touches

- [ ] 5. :trophy: **Cut an operator release using the GitHub release tool**

- [ ] 5. :green_book: **Make sure to update the [release notes](https://github.com/confidential-containers/confidential-containers/tree/main/releases) and tag/release the confidential-containers repo using the GitHub release tool.**

- [ ] 7. :hammer: **Poke Wainer Moschetta (@wainersm) to update the release to the OperatorHub. Find the documented flow [here](https://github.com/confidential-containers/operator/blob/main/docs/OPERATOR_HUB.md).**

