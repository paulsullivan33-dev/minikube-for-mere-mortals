# The Moving Parts

**Outcome: given any minikube behaviour — a file that appeared, a setting that stuck, an error that named a component — you can say which part produced it.**

Minikube looks like one program. It is better
understood as six parts with distinct jobs; most
confusion about it is two parts being mistaken
for each other.

## The command

`minikube` itself is a single self-contained
program you run from a terminal. Its command list
(taken from `minikube --help` on the version this
guide's lab verified, **v1.39.0**) groups itself
honestly:

- **Basic:** `start`, `status`, `stop`, `delete`,
  `dashboard`, `pause`, `unpause`
- **Images:** `docker-env`, `podman-env`, `cache`,
  `image` — ways to get container images into the
  cluster without a registry round-trip
- **Configuration and management:** `addons`,
  `config`, `profile`, `update-context`
- **Networking:** `service`, `tunnel`
- **Advanced:** `mount`, `ssh`, `kubectl`, `node`,
  `cp` — including driving extra nodes in a
  multi-node cluster
- **Troubleshooting:** `ip`, `logs`,
  `update-check`, `version`, `options`, and the
  ssh-key helpers

The full help surface is in the lab's transcript
`t01`; every flag this guide teaches appears in
it.

## The driver

The **driver** is the machinery that holds the
cluster: a container runtime (Docker or Podman),
a hypervisor that runs a small virtual machine
(KVM, QEMU, VirtualBox, VMware), an SSH
connection to a machine elsewhere, or — the
special case — **none**, meaning Kubernetes runs
directly on the host with no holder at all. The
driver is chosen at `start`, recorded in the
profile, and responsible for most start failures:
it is the part that needs a daemon running, a
kernel device present, or a group membership you
may not have. Chapter 3 is devoted to it.

## The guest

What the driver holds is the **guest**: the
environment Kubernetes is installed into. With a
VM driver the guest is a small Linux virtual
machine booted from an ISO image minikube
downloads (this version's is
`minikube-v1.39.0-amd64.iso`). With the Docker
driver it is a purpose-built container image
(the help text names it: `kicbase`, Kubernetes
in a container). With the none driver there is
no guest at all — the host is the guest, which
is why that driver has the sharpest caveats.

## The bootstrapper

Inside the guest, something has to install
Kubernetes itself: lay down the certificates,
start the control plane, join the node. Minikube
uses **kubeadm**, Kubernetes' own cluster
assembly tool, for that work. You rarely meet
kubeadm directly in minikube — it appears in the
start output's later stages and in the version
requirements it enforces (it is kubeadm's world
that demands tools like `crictl` on a none-driver
host; chapter 3 shows the refusal).

## kubectl and the kubeconfig

Your conversation with the finished cluster goes
through **kubectl**, a separate program, and
kubectl finds clusters by reading a **kubeconfig**
file — by default `~/.kube/config` — that holds,
for each cluster, three linked entries: where the
cluster is, who you are to it, and a **context**
that pairs the two. When `minikube start`
succeeds, minikube writes those entries for you,
named after the profile, and sets the context
current. Chapter 8 takes the kubeconfig apart
with kubectl's own commands, run for real.

## The state on disk

Everything minikube knows lives in one directory
tree: its **home**, `~/.minikube` by default —
profiles (one folder per cluster, holding that
cluster's own configuration), downloaded images
and ISO caches, certificates, logs, and the
global `config/config.json` that chapter 6
edits. The `MINIKUBE_HOME` environment variable
moves the whole tree, which is how this guide's
lab kept its experiments separate from a home
directory another project was already using —
a trick worth stealing on any shared machine.
Two details from the lab, both verified: with
`MINIKUBE_HOME` set, minikube nests its tree in a
`.minikube` folder *inside* the directory you
name; and a failed start still leaves a profile
folder behind (the lab's contained only an
`events.json`), which is why chapter 7's delete
commands have something to clean up even when
nothing ever ran.
