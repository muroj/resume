---
title: "Kubernetes Upgrade"
---

# Kubernetes Upgrade

Our kubernetes clusters were running [version 1.14](https://kubernetes.io/blog/2019/03/25/kubernetes-1-14-release-announcement/), which at the time was at three years old and no longer eligible for support. The clusters were provisioned years prior using a tool called [kops](https://kops.sigs.k8s.io/) by an engineer who was no longer with the company. The kops version used to provision the clusters was also years old and running a deprecated version. No one was familiar with kops and no one dared upgrade the clusters. However, our hand was forced when our production cluster could no longer scale out due to worker nodes failing to join the cluster due to a deprecated dependency in the kops script. The inability to scale out put our customers at risk, as our login service was often running at capacity, and we often needed to increase the deployment size to prevent failed login attempts.

## Task

* Create a plan to upgrade our clusters to the oldest support version. I targeted the oldest version that was still eligible for support rather than the most recent version to minimize application changes. I wanted to limit the risk of breaking anything at the application layer. I was familiar with our helm charts, so I could work with my direct team to PR any required manifest changes.
* Minimize downtime. The SRE team did not have it's own cluster. We shared the dev cluster wit the app developers. I could not break this cluster because it would block the entire development team. In other words, I had to get it right the first time.

## Actions

* I started with the development cluster. The first step was to familiarize myself with kops by reading the documentation. Kops uses YAML files to define the desired cluster state. I located our YAML files, which had become woefully unmaintained, and reconciled the YAML config with the live cluster state. After doing so, I could then proceed with the upgrade process.
* I performed a multi-stage upgrade in our dev cluster. I could not upgrade directly to version 1.23, I first had to upgrade to 1.17. To do that, I had to upgrade kops to a version that supported the desired k8s version.
* I meticulously documented the entire process to allow.
* I ran every command with the dry-run option to allow me to review the diff in the cluster state.
* I performed the upgrade during off-hours to minimize downtime for the developers.
* I verified that all deployments and services running in the cluster were valid.

## Result

The upgrade was successful. The end result was the cluster was running a supported version. This removed deprecated APIs and ensured our customer workload was running on a supported version. It also fixed the worker node issue preventing the cluster from scaling out.
