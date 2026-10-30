# Profiles — More Than One Cluster

**Outcome: you can hold several independent clusters on one machine — say, one for learning and one pinned to a work version — address each by name, ask after each, and remove each cleanly.**

A **profile** is a named cluster. Everything
about one cluster — its driver, version, size,
its folder under `~/.minikube/profiles/<name>/`,
and its entries in your kubeconfig — hangs off
the name. The global flag is on every command:

```
-p, --profile='minikube': The name of the
minikube VM being used. This can be set to allow
having multiple instances of minikube
independently.
```

(quoted from `minikube options`). Left alone,
the profile is named `minikube` — which is why
the default cluster's kubeconfig context is
called `minikube` too. Give `-p` a name at start
and that cluster is its own world:

```bash
minikube start -p learning
minikube start -p pinned --kubernetes-version=v1.36.4
minikube status -p learning
minikube stop -p pinned
```

Clusters in different profiles share the machine
and the image cache, and nothing else. Each can
run a different Kubernetes version, a different
size, even (on machines that support both) a
different driver. Stopping one leaves the other
untouched. This is the practical answer to "I
want to learn the new version without losing the
setup that mirrors work": two profiles, both
disposable, neither precious.

One restriction, proven in chapter 3: the
**none driver supports only the default
profile**. A named profile with `--driver=none`
is refused before anything starts
(`DRV_UNSUPPORTED_PROFILE`, exit 56).

## Asking after profiles

Everything in this section was run in the lab —
against profiles whose starts had failed, which
turn out to be the most instructive kind
(transcript `t07`). When no profile exists at
all, the list command says so in minikube's
error grammar:

```
$ minikube profile list
* Exiting due to MK_USAGE_NO_PROFILE:
  No minikube profile was found.
* Suggestion:
    You can create one using 'minikube start'.

exit: 14
```

Status for a name that was never created is
equally direct, and helpfully prescriptive:

```
$ minikube status -p lab
* Profile "lab" not found. Run "minikube profile
  list" to view all profiles.
  To start a cluster, run: "minikube start -p lab"

exit: 85
```

When profiles *do* exist and are running, the
same commands give you the table of clusters and
each one's host/kubelet/API-server state. The
shapes above are the empty case of that same
machinery — learn them, and the populated case
reads itself.

## Deleting: the full stop

Deleting is a profile operation, and it is
thorough:

```bash
minikube delete -p learning
```

removes the guest (container or VM), the
profile's folder, and the profile's kubeconfig
entries — the cluster's certificates and context
go with it. (Handy to know: `minikube delete`
also *releases* the name. A half-created profile
can block a fresh start with the same name;
delete is the broom.)

The lab's deletes are worth quoting because
they include the failed starts. A profile whose
start died at the driver check still had traces:

```
$ minikube delete -p lab
! "lab" profile does not exist, trying anyways.
* Removed all traces of the "lab" cluster.
```

and the sweep of everything at once:

```
$ minikube delete --all
* Removed all traces of the "labkvm" cluster.
* Removed all traces of the "labnone" cluster.
* Removed all traces of the "minikube" cluster.
* Successfully deleted all profiles
```

Note what `--all` removed: every profile the lab
had *attempted*, including the default-named one
from the none-driver attempt. Failed starts are
not invisible to the tool; they are profiles in
a sorry state, and delete treats them the same
as healthy ones.

What delete deliberately leaves behind: the
downloaded image and ISO caches (shared between
profiles, and expensive to fetch again), your
global config file, and minikube itself. Chapter
12's last recipe covers removing *everything*,
caches and all, when that is what you actually
want.
