# Addons

**Outcome: you know what addons are, can list the ones your minikube ships and who maintains each, and know which few are worth enabling first — and why enabling is a cluster operation, not an install.**

An **addon** is a maintained extra that minikube
can install into your cluster for you: a
dashboard, an ingress controller, a metrics
server, a local registry. They are the batteries
of the tool — things nearly every local cluster
ends up wanting, kept working by the minikube
project or by the upstream projects themselves,
one command from being on:

```bash
minikube addons enable ingress
minikube addons list
```

Enabling applies real manifests to a running
cluster and changes what runs in it, so it is
**documented-only in this guide** — the lab had
no cluster to enable anything on. What the lab
*could* do, and did, is interrogate the catalogue
of the verified build itself (transcript `t03`).
Note, as a small honesty check, that `addons
list` works without any cluster: it is a property
of the minikube binary, not of a cluster. (It
printed one complaint to its error log about the
missing profiles directory while doing so, and
still answered, exit 0.)

## The catalogue, as shipped

**41 addons** in minikube v1.39.0, each with a
named maintainer. The full table is transcript
`t03`; the shape of it matters more than the
roll call:

- **Maintained by Kubernetes or the minikube
  project** — the core set: `dashboard`,
  `default-storageclass`, `storage-provisioner`,
  `ingress`, `metrics-server`, `volumesnapshots`,
  `auto-pause`, `gvisor`, `headlamp`'s neighbours
  aside, `registry`, `ingress-dns`. These are the
  addons whose behaviour this guide describes
  with most confidence.
- **Maintained by their own projects** (marked
  "3rd party"): `istio`, `kong`, `ambassador`,
  `traefik`, `metallb`, `portainer`, `olm`,
  `kubevirt`, `volcano`, and more. Same install
  mechanism; support and documentation live with
  the project named.

Two addons are special: `default-storageclass`
and `storage-provisioner` are enabled by a fresh
start itself (chapter 5), which is why a new
minikube cluster can serve a PersistentVolume
claim without your asking for anything.

## The handful worth meeting first

From the official addons documentation
(`minikube-lab/sources.txt` lists the URLs):

- **`dashboard`** — the Kubernetes web UI.
  Enabled on demand by `minikube dashboard`
  (chapter 11), which is usually the better way
  to meet it than enabling it permanently.
- **`ingress`** — an ingress controller (NGINX-
  based), so Ingress resources — the standard
  way to route web traffic into a cluster —
  actually route somewhere. Local clusters
  without it can only watch Ingress objects sit
  there, correct and useless.
- **`metrics-server`** — the piece that makes
  `kubectl top pods` and `kubectl top nodes`
  answer, and that autoscaling rehearses
  against. A plain cluster without it reports
  "Metrics API not available", which startles
  everyone exactly once.
- **`registry`** — a container registry inside
  the cluster (port 5000), the tidy route for
  testing images you build locally without
  pushing them anywhere public. The alternative
  worth knowing is `minikube image load`, which
  puts an image straight into the cluster with
  no registry at all.
- **`metallb`** — a load-balancer implementation
  for clusters without a cloud, for rehearsing
  LoadBalancer services without `minikube
  tunnel` (chapter 11). Needs a small address
  range configured before it hands out IPs.

## Working habits

Enable addons per cluster (they are recorded in
the profile), and prefer enabling at start when
you know you will want one — `minikube start
--addons=ingress,metrics-server` (the `--addons`
flag is in the start help, transcript `t01b`).
Disable symmetrically: `minikube addons disable
<name>`. And when an addon misbehaves, remember
what it is: ordinary Kubernetes workloads in
your cluster, inspectable with the ordinary
tools — `kubectl get pods -A` will show you its
pods like any others. The magic is only in the
delivery.
