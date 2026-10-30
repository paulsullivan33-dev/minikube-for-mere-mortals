# Your First Start

**Outcome: you can run `minikube start`, say what it is doing at each stage, and — just as valuable — read a start that stops and name the rung it stopped on.**

The basic shape could not be smaller:

```bash
minikube start
```

On a machine with a working driver, this one
command does all of the following, and reports
as it goes:

1. **Settles the shape of the cluster** — driver
   (auto-detected or as configured), how many
   CPUs and how much memory and disk, which
   Kubernetes version, which addons to enable.
   Chapter 9 covers the choices; the defaults
   are sane for learning.
2. **Fetches what it needs** — the VM image or
   container base image, and the Kubernetes
   components for the chosen version, into its
   cache inside `~/.minikube`. First starts
   download a lot; later starts reuse it.
3. **Builds the guest** — starts the container or
   boots the VM, and prepares it.
4. **Installs Kubernetes** — the bootstrapper
   (kubeadm) lays down certificates, starts the
   control plane (you will see the API server's
   port, 8443, in the configuration), and brings
   the node up.
5. **Waits for the parts it promised to wait
   for** — by default the API server and the
   system pods, per the `--wait` flag's own help:
   *"comma separated list of Kubernetes components
   to verify and wait for after starting a
   cluster… defaults to `apiserver,system_pods`"*.
6. **Wires up kubectl** — writes the cluster,
   user and context entries into your kubeconfig
   (named after the profile) and makes the new
   context current, so `kubectl get nodes` talks
   to your cluster immediately.
7. **Enables the starting addons** — the storage
   provisioner and default storage class that
   make persistent volumes work out of the box.

Steps 6 onward — everything after the cluster
exists — are **documented-only in this guide**:
the lab's machine could not build a cluster, so
no successful start was observed in it. The
sequence above is the official documentation's
account of a start, cross-checked against the
stages this version's help and error messages
describe. The checks that confirm a good start
on your machine are two commands:

```bash
minikube status
kubectl get nodes
```

`minikube status` reports the host, kubelet and
API server states for the profile; `kubectl get
nodes` shows the node (or nodes) Ready. Those
are chapter 8's tools; run them whenever a start
finishes — or seems to.

## The starts this lab actually ran

The lab ran three starts, and each failed at a
*different* stage — which is precisely why they
are instructive. All three are quoted in full in
chapter 3; the pattern to carry away is:

- **docker driver** — failed at stage 3's door:
  no daemon to build the guest with
  (`PROVIDER_DOCKER_VERSION_EXIT_1`, exit 69).
- **kvm2 driver** — failed before stage 3 even
  began: a permission prerequisite
  (`PR_KVM_USER_PERMISSION`, exit 67).
- **none driver** — failed at stage 4's door:
  the guest is the host, and the host lacked a
  tool the bootstrapper requires
  (`GUEST_MISSING_CONNTRACK` in code, `crictl` in
  the message, exit 86).

Three lessons generalise from them:

**A failed start is a diagnosis, not a dead
end.** Minikube's errors follow one grammar —
an error class in capitals, the failing step in
plain words, often a suggestion and a
documentation link, and an exit code. Chapter 15
turns that grammar into a field guide.

**Each retry goes one rung further.** Fix what
the message names and run again; the next
message names the next requirement. The none
driver's two-step progress in the lab — profile
restriction first, missing `crictl` second — is
the normal experience of bringing up a new kind
of machine.

**Failed starts leave almost nothing behind.** A
profile folder with an events log, and cache
entries. `minikube delete` sweeps even that
(chapter 7 runs it on all three of the lab's
failed profiles). You can experiment freely; the
cost of a failed start is a message, not a mess.

## Starting with intent

Bare `minikube start` is the right first command.
The second time, start saying what you mean:

```bash
minikube start --driver=docker --profile=learning
minikube start --kubernetes-version=v1.36.4 --profile=work-test
minikube start --cpus=4 --memory=6144 --profile=roomy
```

Flags beat the persistent config for one-off
experiments; the config file (next chapter) is
for the choices you make every time. Either way,
what you chose is recorded in the profile, and
`minikube status -p <name>` will tell it back to
you.
