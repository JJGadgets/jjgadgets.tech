---
title: "Flux Repo Structure"
description: "Some explanations as to the FluxCD repo structure used at https://github.com/JJGadgets/Biohazard/ repo."
date: "2023-05-03"
slug: "2024/flux-repo-structure"
categories:
  - "GitOps"
  - "Kubernetes"
  - "Homelab"
tags:
  - "FluxCD"
---
# Introduction
In this post, I go over the repository structure that I use for my Kubernetes homelab, powered with GitOps by FluxCD.

Remember that *there is no "right way" or "one size fits all" to repo structuring*, define and be clear on your goals before you structure your repo.

**NOTE:** From here on out,  all *files* will be in bold and is prefixed by **./** while all *folders* will be in bold and suffixed by **/** e.g. **folder/** and **./file.yaml**

# Goals

  + Define desired configuration of apps and services deployed in Kubernetes cluster.

  + Version control all configuration changes with Git.

  + Automate all the things! (as much as possible)

  + Establish high security baseline of both Kubernetes cluster's components, and GitOps components.

  + Allow for basic multi-cluster environments (production and non-production) while using monorepo

    + Cluster-specific configurations using variable substitution in deploy manifests, and cluster-specific secrets.

  + Allow for opting-in which apps/services/components to deploy per cluster, to optimize resource allocation (CPU, memory, storage, network etc).

    + I *don't really need* a Minecraft Java server on both prod and staging if I don't change any of its configuration, do I?
  
  + Minimize human errors or forgetfulness by keeping duplicated code to an absolute minimum.

# Repo Structure (**kube/**)

  + ## **bootstrap/**

    + **flux/**

      + **./kustomization.yaml**

        + Installs Flux from the kustomization.yaml located in ./manifests/install of fluxcd/flux2 remote GitHub repo.

  + ## **clusters/**

    + **cluster-name/**

      + Cluster specific folder

      + Any name is fine, e.g. **prod/** and **dev/**

      + **distro/**

        + Cluster distribution's configuration

        + e.g. Talos via talhelper (**talos/**)

      + **config/**

        + All Flux manifests for cluster-specific configuration state goes here

        + **./flux-install.yaml**

          + Flux SourceRepository pointed to Flux's manifests OCI repo, scoped to version branch

          + Fluxtomization to deploy and control Flux's components (takes over control of Flux components from Flux bootstrap)

        + **./flux-repo.yaml**

          + Flux SourceRepository pointed to user's repo (e.g. JJGadgets/Biohazard, onedr0p/home-ops, 0dragosh/homelab etc), use SSH key to clone (and optionally push if configured)

          + "Master" Fluxtomization to deploy and control cluster-specific configuration (path points to cluster folder) and all other deployments (via cluster folder's ./kustomization.yaml)

            + Includes configuration and patches for variable substitution ( `${VARIABLE}` ) and SOPS secret decryption of all other deployments

            + Includes patches for DRYing configs

              + For the truly lazy, patching a Fluxtomization with patches for other resources controlled by said Fluxtomization (e.g. HelmRelease) is possible

        + **./kustomization.yaml**

          + Native Kubernetes Kustomization to control what resources are deployed, either by Fluxtomization or `kubectl apply -k`

          + Used here for opt-in deployment of all cluster-specific configuration manifests in cluster folder.

          + Used here for opt-in deployment of which apps folders to deploy to cluster (from **kube/deploy/**).

            + References the apps folders to deploy like so: `../../../deploy/apps/jellyfin`

            + Each app folder will then have its own **kustomization.yaml**, which opts-in deployment of the namespaces needed as well as the app's Fluxtomization (**ks.yaml**).

              + Indirectly the "master" Fluxtomization *also controls the namespaces deployed* (since there's no Fluxtomizations in between the chain of **kustomization.yaml**s).

              + This allows the app's Fluxtomization to add "master" Fluxtomization as dependency (via dependsOn).

                + Ensures that namespaces are always deployed before the resources are deployed within the namespaces.

              + *NOTE:* (about namespacing)

                + I choose to group apps in their own namespace, unless an app requires either multiple containers (e.g. server and web client, or microservices architecture), or 2 different apps must be in the same namespace to share Kubernetes resources or other reasons.

                + My structure is a janky ghetto way to implement "multi-cluster" with a very specific purpose of having production and non-production (dev/test/staging/UAT, whichever is appropriate) clusters, where the differences in the clusters mostly come down to cluster-specific variables/secrets and control of which apps are deployed, allowing the non-prod cluster(s) to be scaled smaller than the prod cluster (e.g. I *don't really need* Jellyfin on both prod and staging if I don't change any of its configuration)

                + Other folks like onedr0p, bjw-s, 0dragosh etc will have the "master" Fluxtomization deploy a separate "apps" Fluxtomization that points to **kubernetes/apps** which has **kubernetes/apps/namespace/kustomization.yaml** control both namespace and app Fluxtomizations (**kubernetes/apps/namespace/app/ks.yaml**) to deploy, *which isn't a wrong way to structure for their needs*, however this means that there is no easy way to control which cluster should deploy which apps which I desire

        + **./secrets.yaml**

          + Cluster-specific native Kubernetes secrets, encrypted-at-rest by SOPS (Flux supports decrypting SOPS-encrypted files).

          + Can be used via variable substitution at deploy-time.

          + external-secrets is preferred over storing secrets in here to easily sync and/or consume namespace-specific secrets.

          + One hacky way to have cluster-specific yet namespace-specific native Kubernetes secrets (you can't reference a secret stored in `flux-system` namespace from e.g. `minecraft` namespace) is to variable substitute the secret data into a `kind: Secret` manifest in **kube/deploy/**.

        + **./vars.yaml**

          + Cluster-specific variables.

          + Can be used via variable substitution at deploy-time.

  + ## **deploy/**

    + Apps, services and components to be deployed to clusters.

    + Contains baseline common manifests that are written to work across all clusters.

    + Cluster-specific configuration is achieved via variable substitution and optionally patches.

    + **core/**

      + Backend(-related) components that are "core" to Kubernetes clusters for them to operate, such as CNI, storage solution, Ingress Controller, and more.

    + **apps/**

      + User-facing apps and services.

      + **app-name/**

        + Replace **app-name/** with app name, such as **minecraft/**, **authentik/**, or **immich/**.

        + **./ns.yaml**

          + Namespace definition for app, or group of apps.

        + **./ks.yaml**

          + All app-specific Fluxtomizations that deploy and control the app's resources.

          + Points to either **app/** and/or **component-name/** folders which are explained below.

          + Use dependsOn to ensure that dependencies (such as storage solution and Ingress Controller) are deployed and configured before app's resources are deployed and configured.

        + **./repo.yaml**

          + Only needed if deployment method relies on Flux SourceRepository, such as HelmRepository or OCIRepository.

        + **./kustomization.yaml**

          + Native Kubernetes Kustomization to control what resources are deployed, either by Fluxtomization or `kubectl apply -k`

          + Used here for opting-in all YAML files but not folders, since folders are controlled by app-specific Fluxtomization which itself is controlled by "master" Fluxtomization via this and cluster's **kustomization.yaml**.

        + **app/** OR **component-name/**

          + All manifests used to deploy Kubernetes resources needed for app are placed here.

          + Usually if both **app/** and another folder such as **config/** or **certs/** are present, **app/** is used to deploy components which are a dependency for the other folders' resources to be deployed.

          + **./netpol.yaml**

            + CiliumNetworkPolicy or Kubernetes native Network Policy

            + "*Principle of Least Privilege*" and "*Default Deny*" practices are followed, only allow necessary connections

              + Commonly allowed include Ingress Controller, intra-namespace traffic, communication with other apps' Kubernetes services *as needed*.

              + Not all Kubernetes components need to be network accessible by every application, such as APIServer, storage solution pods/services, or Flux. Assign only if such traffic is necessary for app to function.

          + **./hr.yaml**

            + HelmRelease deployment method.
