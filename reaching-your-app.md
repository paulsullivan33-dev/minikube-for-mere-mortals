# Reaching Your App

**Outcome: given something running in your cluster, you know the three doors between it and you — the dashboard, `minikube service`, and `minikube tunnel` — and which door fits which kind of service.**

> Everything in this chapter needs a running
> cluster, so all of it is **documented-only**:
> described from minikube's official handbook
> pages (listed in `minikube-lab/sources.txt`),
> not run in this guide's lab. The commands and
> behaviours below are quoted from those pages.

A pod in a cluster lives on the cluster's own
network. Your browser and terminal live on the
host's. Minikube provides three bridges, and the
skill is matching the bridge to the situation.

## The dashboard: seeing everything

```bash
minikube dashboard
```

This enables the dashboard addon and opens the
Kubernetes web dashboard in your browser: an
overview of everything running, a way to look at
workloads, read logs, and scale or edit
resources by clicking. Two facts from the
official page worth planning around: the command
runs a proxy in the foreground (Ctrl+C stops the
proxy; the dashboard itself keeps running in the
cluster), and browsers behave badly as root — if
you work as root, get just the address instead:

```bash
minikube dashboard --url
```

The dashboard shows the cluster kubectl sees:
which cluster you are looking at is decided by
the current context (chapter 8), and by the
`-p` profile you name.

## NodePort services: `minikube service`

The basic way to expose an app out of a cluster
is a **NodePort** service: Kubernetes opens a
port (by default in the range 30000–32767) on
the node, and traffic to it reaches your app.
The handbook's worked example, condensed, is the
canonical first run:

```bash
kubectl create deployment hello-minikube1 \
  --image=kicbase/echo-server:1.0
kubectl expose deployment hello-minikube1 \
  --type=NodePort --port=8080
minikube service hello-minikube1 --url
```

The last command answers with a URL —
`http://127.0.0.1:<some port>` on Docker-driver
setups — and, on several platforms, it *is* a
tunnel: the docs note it must keep running in its
terminal to keep the path open, because with the
Docker driver on macOS, Windows and WSL the
node's own IP is not directly reachable from the
host. On Linux with the Docker driver, no tunnel
is created and the address is the node's. Either
way, the same information exists in kubectl
terms — the node's IP is `minikube ip`, and the
assigned NodePort is in `kubectl get svc` — the
`service` command exists to spare you assembling
them.

## LoadBalancer services: `minikube tunnel`

A **LoadBalancer** service is the standard way
to expose an app on a real cluster, where the
cloud provides an external IP. A laptop has no
cloud, so without help the service's EXTERNAL-IP
sits at `<pending>` forever. The help is:

```bash
minikube tunnel
```

Run it in its own terminal and leave it running:
it creates a route from your host into the
cluster's service network (it will ask for your
password — adding routes is a privileged act),
and LoadBalancer services then get real,
reachable external IPs. Ctrl+C tears the routes
down; if a tunnel ever dies uncleanly and leaves
stale routes behind, the docs' remedy is
`minikube tunnel --cleanup`.

Choose this door when the thing you are
rehearsing is itself about LoadBalancer
behaviour — ingress controllers and production
manifests usually assume it — or skip it and use
NodePort when all you need is to see your app in
a browser. `metallb` (chapter 10) is the third
option, for rehearsing load balancing as a
cluster feature rather than a host tunnel.

## Which door, summarised

- **Look at the cluster itself** → `minikube
  dashboard` (add `--url` when a browser cannot
  be opened for you).
- **Reach one app, quickly** → a NodePort service
  and `minikube service <name> --url`.
- **Rehearse the production shape** → a
  LoadBalancer service and `minikube tunnel` in
  its own terminal, or the `metallb` addon.

And the standing warning for all three: these
bridges exist for a local cluster on your own
machine. They are conveniences for development,
not ways to publish services to the world.
