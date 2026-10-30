# Glossary

**Addon** — a maintained extra (dashboard,
ingress controller, metrics server…) that
minikube installs into a cluster with
`minikube addons enable <name>`. The catalogue is
a property of the minikube binary: `minikube
addons list`.

**Bootstrapper** — the component that installs
Kubernetes inside the guest. Minikube uses
kubeadm, Kubernetes' own assembly tool.

**Cluster** — a whole Kubernetes installation:
control plane plus one or more nodes. In
minikube, one profile is one cluster.

**Context** — a kubeconfig entry pairing a
cluster with a user: "talk to *this* cluster as
*this* identity." `current-context` is the
pairing kubectl uses by default.

**Control plane** — the part of Kubernetes that
keeps the records and gives the orders (its API
server is the part kubectl talks to).

**Daemon** — a background service process. The
Docker driver needs Docker's daemon, which is a
different thing from the Docker command-line
client; a client with no daemon is chapter 3's
first refusal.

**Driver** — the machinery that holds a minikube
cluster: a container runtime (docker, podman), a
hypervisor (kvm2, qemu2, virtualbox, vmware), an
SSH target, or none (the host itself).

**Guest** — the environment Kubernetes is
installed into: a container, a virtual machine,
or (none driver) the host.

**kubeadm** — Kubernetes' own cluster-assembly
tool; minikube's bootstrapper. Its requirements
(for example, `crictl` on a none-driver host)
surface as minikube errors.

**kubeconfig** — the file (default
`~/.kube/config`) where kubectl finds clusters,
users and contexts. It holds credentials in
plain text and should be guarded like a password
file.

**kubectl** — the command-line tool for talking
to a Kubernetes cluster. Separate from minikube,
which builds clusters; kubectl uses them.

**LoadBalancer** — a service type that asks for
a real external IP. On minikube, `minikube
tunnel` plays the part of the provider that
grants one.

**Manifest** — a file (usually YAML) describing
what should run in a cluster; Kubernetes works
to make reality match it.

**MINIKUBE_HOME** — the environment variable
that moves minikube's whole state tree away from
the default `~/.minikube` (it nests a `.minikube`
folder inside the directory you name).

**Node** — one machine (real or played by a
container/VM) in a cluster, on which pods run.
`minikube start --nodes` sets how many.

**NodePort** — a service type that opens a port
(30000–32767 by default) on the node; the basic
way in from the host, with `minikube service`.

**none driver** — the driver that installs
Kubernetes directly on the host: root required,
default profile only, no isolation, and the host
itself becomes disposable.

**Pod** — the smallest unit of Kubernetes work:
one or more containers that live and die
together.

**Profile** — a named minikube cluster, with its
own driver, version, size and folder. The
default profile is named `minikube`; select one
with `-p <name>`.

**Version skew** — how far apart kubectl and its
cluster may be: one minor version either side,
by Kubernetes' rule.
