# Versions and Sizing

**Outcome: you can start a cluster at the Kubernetes version you actually need — typically the one production runs — with kubectl at a compatible version, and give the cluster resources that match the job.**

Two version numbers matter in minikube life, and
they are not the same number: the version of
**Kubernetes** the cluster runs, and the version
of **kubectl** you talk to it with. Then there
is a third set of numbers — how much machine the
cluster gets. All of the facts in this chapter
were read off the lab's verified binaries
(transcripts `t01`, `t01b`, `t07`).

## The Kubernetes version

`minikube start --help` states the default in
its own words:

```
--kubernetes-version='':
    The Kubernetes version that the minikube VM
    will use (ex: v1.2.3, 'stable' for v1.37.0,
    'latest' for v1.37.0). Defaults to 'stable'.
```

So in this build, both `stable` and `latest`
mean v1.37.0. Pin an exact version when you mean
one:

```bash
minikube start --kubernetes-version=v1.36.4
```

or make it a habit in the config file
(`minikube config set kubernetes-version v1.36.4`,
chapter 6). Which versions can this build
install at all? Ask it — `minikube config
defaults kubernetes-version` listed, for the
lab's build, every release from v1.37.0 back to
v1.28.0 (plus their release candidates and
betas). That list is a property of the *minikube*
release, not of the internet: an older minikube
cannot start a Kubernetes newer than the newest
it knows, which is the usual reason to keep
minikube itself current (chapter 4).

Why pin at all? Because the point of a local
cluster is usually to rehearse something bound
for a real one, and Kubernetes behaviour —
especially which API versions exist — changes
between minor versions. A manifest that works on
1.37 and fails on 1.34 is exactly the kind of
surprise you want on your laptop, not in
production. Match the minor version of the
cluster you are rehearsing for.

## The kubectl version: the skew rule

Kubernetes' rule for kubectl is simple: it
supports a cluster **one minor version either
side** of its own. kubectl v1.37 talks properly
to clusters at 1.36, 1.37 and 1.38; further
apart, commands start failing in ways that look
like cluster bugs and are not.

The lab installed the current stable kubectl,
v1.37.2, while minikube's default cluster is
v1.37.0 and its built-in wrapper kubectl
(`minikube kubectl --`, chapter 8) is v1.37.0.
All comfortably inside the rule. Where the rule
bites: keeping an ancient kubectl around and
pointing it at a new local cluster, or the
reverse. If versions must match exactly — a
script, a tutorial, a test matrix — the wrapper
is the version that always matches the cluster,
by construction.

## Sizing: CPUs, memory, disk

From the same help text (transcript `t01b`):

```
--cpus='2':
    Number of CPUs allocated to Kubernetes.
    Use "max" to use the maximum number of CPUs.
    Use "no-limit" to not specify a limit
    (Docker/Podman only)

--memory='':
    Amount of RAM to allocate to Kubernetes
    (format: <number>[<unit>], where unit = b, k,
    m or g). Use "max" to use the maximum amount
    of memory. Use "no-limit" to not specify a
    limit (Docker/Podman only)

--disk-size='20000mb':
    Disk size allocated to the minikube VM
    (format: <number>[<unit>], where unit = b, k,
    m or g).

--nodes=1:
    The total number of nodes to spin up.
    Defaults to 1.
```

Reading those carefully: CPUs default to 2 and
disk to about 20 GB; memory prints no built-in
number — minikube decides it at start time for
your driver unless you pin it, which is a good
reason to pin it for any cluster you will
describe to someone else. With VM drivers these
are the virtual machine's size; with container
drivers, limits on the cluster's container
(`no-limit` exists only there, because a VM
cannot be told "however much there is").

Practical guidance, stated as guidance:

- **Learning and manifest testing:** the
  defaults are fine. Kubernetes' own machinery
  fits comfortably; what you add is your own
  workload.
- **Running a real application's stack locally:**
  give memory first (most local pain is memory),
  then CPUs. `--memory=6144 --cpus=4` is a
  typical step up. Remember the cluster shares
  the machine with you — sizing to `max` makes
  the *laptop* the thing that struggles.
- **Disk is the quiet one:** images accumulate
  inside the cluster. 20 GB fills faster than
  expected if you build images into it (the
  `docker-env` workflow); size up, or prune with
  the image commands from chapter 2.
- **More nodes:** `--nodes=3` gives a genuinely
  multi-node cluster on the one machine —
  valuable for rehearsing how work spreads and
  drains, at the obvious cost of dividing the
  same hardware three ways. Multi-*control-plane*
  (the `--ha` flag) exists too; for learning it
  is mostly ceremony, and the flag's own text
  says it needs at least three control-plane
  nodes that also work.

Sizes are chosen at start and recorded in the
profile. Changing them for an existing cluster
is the config file's warning made flesh: delete,
then start with the new numbers.
