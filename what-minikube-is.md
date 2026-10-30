# What Minikube Is

**Outcome: you can say what minikube builds, what it is for, and what it is not — before touching a command.**

Minikube builds a complete Kubernetes cluster on
a single ordinary computer: your laptop, your
desktop, a spare machine under the desk. One
command creates it, one command deletes it, and
in between it is a genuine cluster — the real
Kubernetes software, arranged the way Kubernetes
is arranged, answering the same commands with the
same behaviour. People use it to learn Kubernetes
without renting anything, and to try their
manifests and settings against a real cluster
before those go anywhere that matters.

Two paragraphs of Kubernetes, so the rest of the
guide has somewhere to stand. Kubernetes is a
system for running containerised applications
across a group of machines: you describe what
should be running — in files called **manifests**
— and Kubernetes works to make reality match your
description, restarting what crashes and spreading
work across the machines it has. The machines are
**nodes**; the smallest unit of work is a **pod**
(one or more containers that live and die
together); a program called the **control plane**
keeps the records and gives the orders, and you
talk to it with a command-line tool, **kubectl**.
That is enough to follow this guide. For the full
picture — pods, deployments, services, storage,
and honest advice about when you do not need any
of it — this series has
[Kubernetes for Mere Mortals](https://paulsullivan33-dev.github.io/kubernetes-for-mere-mortals/).

What minikube adds to that picture is a
*where*. A real cluster's nodes are separate
machines. Minikube's cluster lives inside
something your one machine can hold: usually a
container or a small virtual machine that plays
the part of a node (or several, if you ask).
Minikube downloads the Kubernetes components,
assembles them inside that holder, wires up
kubectl so your commands reach it, and reports
back. The machinery that plays the holder is
called the **driver**, and choosing one is the
decision chapter 3 is about.

Three properties define the tool, and this guide
will keep returning to them:

- **It is disposable.** A minikube cluster is
  meant to be deleted. Testing something risky,
  breaking something educational, or trying a
  different Kubernetes version costs one
  `minikube delete` and a fresh start. Nothing
  about it asks to be preserved.
- **It is representative, not identical.** The
  Kubernetes inside is the real thing, at a real
  version you can choose (chapter 9). What is
  *not* real is the setting: one machine, no
  network worth the name, storage that is a file
  on your disk. Chapter 14 (Limits) is precise
  about which conclusions transfer and which do
  not.
- **It is yours.** No account, no cloud, no
  control panel elsewhere. The cluster runs on
  hardware you can unplug, and when it is deleted
  there is nothing left to bill you.

One more thing, because this guide's lab makes
it unavoidable: minikube can only build its
cluster out of machinery your machine actually
has. A machine with no container daemon and no
hypervisor gives minikube nothing to build with,
and minikube says so — clearly, in the end, but
one requirement at a time. This guide's lab
machine is exactly such a machine, and chapters
3 and 5 quote its refusals in full. On an
ordinary machine — Docker installed, or a working
hypervisor — the same commands succeed, and the
chapters mark honestly which steps the lab ran
and which it read about.
