# Limits

**Outcome: you can name what a minikube cluster cannot teach you, cannot survive, and should never be asked to do — before one of those surprises you.**

**One machine, pretending.** Every node a
minikube cluster has — even a three-node or
high-availability one — is the same physical
machine wearing different hats. It cannot teach
you how a cluster behaves when machines fail
separately, when a network partitions, or when
nodes disagree about the time. Scheduling,
spreading and draining behave correctly in
software; the physics underneath them is a
polite fiction. Rehearse procedures here;
validate resilience somewhere real.

**The cluster sleeps when the laptop does.**
Close the lid, reboot for an update, let the
host run out of battery — the cluster stops with
the machine, mid-whatever. `minikube start`
revives it with its state intact, which is
genuinely convenient, but nothing about minikube
is *always on*. It is not a home for services
other people or other machines depend on; that
job belongs to a machine that stays awake
(chapter 13's k3s, or a real server).

**Performance numbers lie, in both directions.**
Your workload shares one machine's CPUs, memory
and disk with the host, the driver, and
Kubernetes itself. Things that fly locally may
crawl in production on smaller nodes; things
that crawl locally may be starving next to your
browser. Use a local cluster for *does it work*
and *does it behave*, never for *how fast*.

**The network is a diorama.** One node, one
host-only path in, ports forwarded by tunnels
and host tools (chapter 11). Load balancers,
ingress, DNS and service discovery all
*demonstrably work* — and none of it exercises
the real internet's routing, firewalls, or
failure modes. Conclusions about connectivity
transfer as concepts, not as proofs.

**Storage is a file.** Persistent volumes are
served from inside the guest, which is inside
your disk. The storage classes, binding and
claims are real Kubernetes; the performance and
the failure modes (a disk is either there or
the machine is off) are not a storage system's.

**It needs a machine that can hold it.** The
driver matrix (chapter 3) is a list of
prerequisites — a daemon, a hypervisor, or
root on a disposable host. Restricted machines
(corporate laptops with locked-down
virtualisation, shared sandboxes like this
guide's lab) may offer none of them, in which
case minikube's answer is a clear refusal, not a
workaround. Nested environments — inside many
CI systems and cloud shells — are the same
story: sometimes containers work there, often
nothing does. This guide's lab is the worked
example; the refusals were complete and honest,
and there was no fourth option hiding behind
them.

**It is one Kubernetes at a time, chosen from a
list.** A minikube build installs the Kubernetes
versions it knows (the lab's: v1.28.0–v1.37.0).
Yesterday's exotic version, or tomorrow's
unreleased one, waits for a minikube that knows
it. Addons, too, are the versions the project
pins — you get the maintained thing, not the
customisable thing.

**And the small-print limits.** Clusters are
per-user (your `~/.minikube`, your kubeconfig);
the none driver is single-profile and
host-invasive by design; and a minikube cluster
deleted is deleted — the tool's disposability is
a feature right up to the moment someone kept
something only inside it. Nothing in the cluster
is backed up unless you built the backup.
