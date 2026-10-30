# The Neighbours

**Outcome: you can say, for each nearby tool, what shape of cluster it makes and which job it is better at than minikube — so the choice is deliberate, not habitual.**

Minikube is the eldest of the local-Kubernetes
tools, but it is not the only one, and its
neighbours are not copies: each makes a different
trade. The characterisations below are
documented-only — from the projects' own
descriptions, listed in `minikube-lab/sources.txt`
— except K3s, which this series has run for real
in its own guide.

## kind — Kubernetes IN Docker

**kind** (a Kubernetes SIG project) runs cluster
nodes as Docker containers — no VM option, no
dashboard-and-addons layer, configuration as a
YAML file describing the cluster's shape. Its
home ground is *testing Kubernetes itself* and
CI: clusters that spin up fast, in numbers, with
multi-node and multi-control-plane layouts that
are awkward in minikube, on machines that are
themselves often containers. If your work is
"run my test suite against three fresh clusters
before breakfast," kind is shaped for it. If
your work is "learn Kubernetes on my laptop,"
kind will feel bare next to minikube's drivers,
addons and dashboard — it assumes you brought
your own everything.

## k3d — k3s in Docker

**k3d** wraps Rancher's lightweight k3s
distribution in Docker containers: a cluster as a
set of containers, created and deleted by one
small tool. It inherits k3s's leanness (next
section) and kind's container-only shape, and it
is popular for local development loops that want
k3s's exact behaviour. Choose it when the target
is k3s and the machine has Docker; it has nothing
to say to a machine without a daemon — the same
wall this guide's lab hit in chapter 3.

## Docker Desktop's Kubernetes

If Docker Desktop is already your container
runtime on macOS or Windows, it contains a
single-node Kubernetes you switch on in its
settings — no second tool to install, kubectl
context (`docker-desktop`) included. It is the
lowest-effort cluster on those platforms and the
least configurable: one node, the version Docker
chose, no driver matrix, no profiles in
minikube's sense (one cluster per Desktop). For
"my laptop just needs a cluster sometimes," that
trade is often right. Minikube earns its place
when you need the choices — versions, sizing,
several clusters, addons — or when there is no
Docker Desktop in the picture, as on Linux.

## k3s — not local-only at all

**k3s** is a complete, production-capable
Kubernetes distribution compressed into one small
binary, from Rancher. It runs happily on a single
small machine — which makes it a minikube
alternative for a spare box or a Raspberry Pi —
but it does not stop being a real cluster when
you close the laptop lid: it is meant to keep
running, on real or virtual hardware, possibly
several machines, possibly for good. The question
that separates them is *intent*: minikube builds
a cluster you will throw away on the machine in
front of you; k3s installs a cluster you might
keep, on a machine that stays on. This series'
[K3s for Mere Mortals](https://paulsullivan33-dev.github.io/k3s-for-mere-mortals/)
guide covers installing and growing it, verified
live.

## A real cluster, briefly

Managed Kubernetes (the clouds' offerings) and
self-built multi-machine clusters are not
neighbours so much as the destination: what the
local tools rehearse for. Everything minikube
teaches about kubectl, contexts, versions and
manifests transfers unchanged — chapter 8's
kubeconfig is exactly how you will talk to them,
with different entries in it. What does not
transfer is listed honestly in the next chapter:
the forgiving physics of one machine.

## The short version

- **Learning, rehearsing, addons, choice of
  versions and drivers, several throwaway
  clusters:** minikube.
- **Fast disposable clusters for tests and CI,
  Docker present:** kind.
- **Rehearsing for k3s, Docker present:** k3d.
- **Already living in Docker Desktop, one casual
  cluster:** turn on its Kubernetes.
- **A small cluster that stays:** k3s.
