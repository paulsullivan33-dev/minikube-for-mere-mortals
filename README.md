# Minikube for Mere Mortals

Kubernetes is usually something that runs in a
data centre, on machines you rent. **Minikube**
shrinks it to one machine you own: it builds a
small, complete, real Kubernetes cluster on your
laptop or desktop, so you can learn how the thing
behaves and test your work against it before any
of it goes near production. The cluster is not a
simulation or a screenshot tour — the same
`kubectl` commands, the same manifests, the same
kinds of failure. When you delete it, it is gone,
and nothing anywhere else noticed.

This guide explains what minikube does, the one
decision that shapes everything (the **driver** —
the machinery that holds the cluster up), how to
install it, and how to run your first cluster and
talk to it. Then it covers the parts that make
minikube pleasant for its real job: the persistent
config file, **profiles** (several independent
clusters on one machine), **addons** (batteries
such as a dashboard and an ingress controller,
one command away), and the three ways to reach an
app running inside.

The lab that wrote this guide deserves a word up
front, because it shapes the whole book. The lab
machine is a shared sandbox with **no Docker
daemon, no virtual-machine support, and no
permission to run Kubernetes on the bare metal** —
so no cluster ever came up in it. Rather than
hide that, the guide makes it the first lesson:
every way of holding a cluster up was tried, each
one refused at a different rung, and the refusals
are quoted word for word. Everything on the
*client* side — downloading and checksum-verifying
the real binaries, the entire help and config
surface, profiles, the addons catalogue, and
kubectl's half of the conversation — was run for
real, and the transcripts are in `minikube-lab/`.
Everything past a running cluster is taught from
minikube's official documentation and labelled
**documented-only** wherever it appears.
`criteria.md` keeps the full ledger.

## What this guide covers

- [**What Minikube Is**](what-minikube-is.md) —
  the idea in plain terms: a whole cluster, one
  machine, built to be thrown away
- [**The Moving Parts**](the-moving-parts.md) —
  the command, the driver, the guest it builds,
  the bootstrapper, and where all the state lives
- [**Choosing a Driver**](choosing-a-driver.md) —
  containers, virtual machines, or bare metal:
  what each choice costs and what each demands,
  with this lab's three refusals as worked examples
- [**Installing Minikube**](installing-minikube.md) —
  the official download, verifying its checksum
  (including the trap in how that checksum is
  published), and the version check that proves
  it runs
- [**Your First Start**](your-first-start.md) —
  what `minikube start` actually does, step by
  step, and how to read a start that stops
- [**The Config File**](the-config-file.md) —
  settings that survive between runs: `config
  set`, `get`, `view` and `unset`, all run for
  real, and the plain JSON file they edit
- [**Profiles — More Than One Cluster**](profiles.md) —
  named clusters side by side, their status and
  their clean removal — including removing ones
  that never finished starting
- [**Talking to the Cluster**](talking-to-the-cluster.md) —
  kubectl and the kubeconfig file: clusters,
  users and contexts, what minikube writes for
  you, and how to read "connection refused"
- [**Versions and Sizing**](versions-and-sizing.md) —
  choosing a Kubernetes version, the version-skew
  rule for kubectl, and how much CPU, memory and
  disk a cluster gets
- [**Addons**](addons.md) — the catalogue of 41
  addons this version ships, who maintains each,
  and the handful worth meeting first
- [**Reaching Your App**](reaching-your-app.md) —
  the dashboard, `minikube service`, and
  `minikube tunnel`: the three doors into a
  cluster (documented-only)
- [**Recipes**](recipes.md) — a learning cluster,
  a cluster pinned to a production version, a
  throwaway cluster for testing manifests, and
  deleting everything properly
- [**The Neighbours**](the-neighbours.md) —
  kind, k3d, Docker Desktop's Kubernetes, k3s,
  and a real cluster: honest shapes, and when
  each is the better tool
- [**Limits**](limits.md) — one machine pretending,
  a cluster that sleeps when the laptop does, and
  the rest of what minikube is not
- [**Troubleshooting**](troubleshooting.md) —
  reading minikube's error codes, the daemon that
  is not there, the driver that wants a group
  membership, and the profile that does not exist
- [**Glossary**](glossary.md) — the terms,
  collected in one place

## What it doesn't cover

Kubernetes itself — what pods, deployments and
services *mean* — beyond the two paragraphs
chapter 1 spends on it. That is a guide of its
own in this series, and this one points to it
rather than re-teaching it badly. Production
clusters, cloud Kubernetes, and multi-machine
clusters are likewise out: minikube's job is the
machine in front of you. Nothing here needs an
account, a cloud, or a credit card.

## How to use this guide

Read chapter 1 for the model and chapter 3
before installing anything: the driver decision
explains nearly every success and every failure
that follows. Chapter 5 walks a first start with
this lab's failed attempts beside it, so a start
that stops on your machine is a named problem,
not a mystery. If your machine is an ordinary
laptop or desktop with Docker or a hypervisor,
your start will simply succeed where the lab's
could not — the chapters say plainly which parts
the lab proved and which parts it only read
about. The lab's transcripts live in
`minikube-lab/transcripts/`, and
`minikube-lab/sources.txt` lists the official
documentation the documented-only chapters rest
on.

---

## Part of the series

This guide is one of the [For Mere Mortals](https://paulsullivan33-dev.github.io/mere-mortals/) guides: plain-language guides to the tools programmers and makers use.

Related guides:

- [Kubernetes for Mere Mortals](https://paulsullivan33-dev.github.io/kubernetes-for-mere-mortals/)
- [Docker for Mere Mortals](https://paulsullivan33-dev.github.io/docker-for-mere-mortals/)
- [Docker Compose for Mere Mortals](https://paulsullivan33-dev.github.io/docker-compose-for-mere-mortals/)
- [K3s for Mere Mortals](https://paulsullivan33-dev.github.io/k3s-for-mere-mortals/)
- [Helm for Mere Mortals](https://paulsullivan33-dev.github.io/helm-for-mere-mortals/)

## License

The writing in this guide is licensed under [CC BY 4.0](https://github.com/paulsullivan33-dev/minikube-for-mere-mortals/blob/main/LICENSE). Code examples and lab scripts are licensed under the [MIT License](https://github.com/paulsullivan33-dev/minikube-for-mere-mortals/blob/main/LICENSE-CODE).
