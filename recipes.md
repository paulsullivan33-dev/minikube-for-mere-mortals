# Recipes

**Outcome: four complete setups — a learning cluster, a cluster pinned to a chosen version, a throwaway test cluster, and a complete tidy-up — assembled from the pieces the earlier chapters proved.**

Steps that run against a live cluster follow the
official documentation (the lab had no cluster;
see `criteria.md`). Every client-side step —
config, profiles, deletes — is one this guide
ran for real.

## A learning cluster

Goal: a forgiving place to work through the
Kubernetes guide, with the dashboard a command
away.

```bash
minikube config set driver docker
minikube start -p learning
minikube dashboard -p learning
```

The config line is once, ever; the start is once
per cluster; `minikube stop -p learning` parks it
with its state intact for tomorrow, and the same
`start` brings it back.

## A cluster that matches production's version

Goal: rehearse against the Kubernetes version a
real cluster runs — say v1.34:

```bash
minikube start -p rehearsal --kubernetes-version=v1.34.11
minikube kubectl -p rehearsal -- version
```

`minikube kubectl` answers with the client *and*
server versions: server should read v1.34.11 —
your proof the pin took — and the client is
minikube's matching kubectl, so the skew rule
(chapter 9) is satisfied by construction. Check
your target version exists in this build first:
`minikube config defaults kubernetes-version`
(the lab's build offers v1.37.0 down to v1.28.0).
Apply the manifests you mean to ship, and treat
every surprise here as a production incident
caught early and cheap.

## A throwaway cluster for testing manifests

Goal: prove a set of manifests applies cleanly
to a fresh cluster, then throw the cluster away
— the local rehearsal for a CI job.

```bash
minikube start -p throwaway --interactive=false
kubectl --context=throwaway apply -f ./manifests/
kubectl --context=throwaway get all
minikube delete -p throwaway
```

`--interactive=false` keeps scripts from waiting
on questions (it is in the start help; the lab
used it on every scripted attempt). Naming the
context to kubectl keeps the test honest even if
your current-context points elsewhere. And
delete makes the next run start from nothing —
which is the point: a test that passes only on a
cluster with history is not a test.

## Trying an addon without committing to it

```bash
minikube start -p addon-tour --addons=ingress,metrics-server
kubectl top nodes
minikube delete -p addon-tour
```

Addons chosen at start (chapter 10) live and die
with the profile. `kubectl top nodes` answering
— instead of "Metrics API not available" — is
your proof metrics-server is really in there.

## Deleting everything, completely

Goal: minikube leaves the machine, or a machine
changes hands.

```bash
minikube delete --all --purge
```

Delete itself — all profiles, with their
kubeconfig entries — was run in the lab
(transcript `t07`): it removed even the profiles
whose starts had failed. The `--purge` flag (in
the delete help) goes the last step and removes
the `~/.minikube` directory itself: caches,
config file, certificates, all of it. What
remains after that is only the two binaries,
which you delete like any files, and any
`MINIKUBE_HOME` directory you created on purpose.
Verify with `minikube profile list`: the answer
should be chapter 7's empty-house message,
`MK_USAGE_NO_PROFILE`, exit 14.
