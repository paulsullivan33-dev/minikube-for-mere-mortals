# Talking to the Cluster

**Outcome: you can explain how kubectl finds and authenticates to a cluster, build that wiring by hand once so it is never magic again, and read the "connection refused" family of errors correctly.**

Minikube builds the cluster; **kubectl** is how
you talk to it. The two are separate programs
with a file between them, and understanding that
file — the **kubeconfig** — is the difference
between using kubectl and being used by it.
Everything kubectl-side in this chapter was run
for real in the lab (transcript `t06`), against
an empty kubeconfig the lab built by hand: the
same anatomy minikube writes for you, minus a
cluster that answers.

## The kubeconfig's three lists

kubectl reads `~/.kube/config` by default (the
`KUBECONFIG` environment variable can point it
elsewhere — the lab used that to keep its
experiment in a scratch file). Inside are three
lists and a pointer:

- **clusters** — where a cluster's API server
  is: a name and a server address (and how to
  trust its certificate).
- **users** — who you are to a cluster: a token,
  or a client certificate and key.
- **contexts** — a pairing: this cluster, as
  this user. Plus **current-context**, the
  pairing kubectl uses when you do not say
  otherwise.

An empty kubeconfig, which `kubectl config view`
renders even when the file does not exist yet,
is all nulls:

```yaml
apiVersion: v1
clusters: null
contexts: null
current-context: ""
kind: Config
users: null
```

(`config view` never creates the file — the lab
checked; the file appears only when something
*writes* an entry.)

## Building one by hand

The commands below are exactly what a tool like
minikube does on your behalf. The address is the
one a minikube API server uses; the token is a
deliberate fake, for a cluster that does not
exist:

```bash
kubectl config set-cluster practice \
  --server=https://127.0.0.1:8443 \
  --insecure-skip-tls-verify
kubectl config set-credentials practice-user \
  --token=example-token-not-real
kubectl config set-context practice \
  --cluster=practice --user=practice-user
kubectl config use-context practice
```

Each step confirms itself (`Cluster "practice"
set.`, `Switched to context "practice".`), and
the file that results is short enough to read
whole:

```yaml
apiVersion: v1
clusters:
- cluster:
    insecure-skip-tls-verify: true
    server: https://127.0.0.1:8443
  name: practice
contexts:
- context:
    cluster: practice
    user: practice-user
  name: practice
current-context: practice
kind: Config
users:
- name: practice-user
  user:
    token: example-token-not-real
```

Two facts in that small file deserve emphasis:

**The credential is in plain text.** The token
sits in the file exactly as typed. A kubeconfig
*is* a credential — guard it like a password
file, because anyone holding it holds your
access to every cluster it names.

**`kubectl config view` hides what the file
shows.** In the lab's run, the same file viewed
through `kubectl config view` printed the token
as `<redacted>` — verified by writing the view
to a file and counting: the token appears zero
times in the view, once in the file. So `config
view` output is safe to paste into a bug report;
the file is not.

When `minikube start` succeeds, it writes the
same three entries with the profile's name for
all of them — cluster `minikube`, user
`minikube`, context `minikube` for the default
profile — using a client certificate instead of
a token, and points current-context at it.
`minikube update-context` repairs those entries
if the cluster's address changes, and
`minikube delete` removes them.

## When nobody answers

The lab then asked its practice cluster a
question:

```
$ kubectl cluster-info
E… memcache.go:381] "Couldn't get current server
  API group list" err="Get
  \"https://127.0.0.1:8443/api?timeout=32s\":
  dial tcp 127.0.0.1:8443: connect:
  connection refused"
… (the same line, five attempts)
To further debug and diagnose cluster problems,
use 'kubectl cluster-info dump'.
The connection to the server 127.0.0.1:8443 was
refused - did you specify the right host or port?

exit: 1
```

Learn to love this error's precision. **Connection
refused** means the address is real but nothing
is listening — which, translated, is the most
common kubectl situation in minikube life: *the
cluster is stopped* (or was never started, or
was deleted). The remedies in order: `minikube
status` to see the cluster's state, `minikube
start` to bring it up, and `kubectl config
current-context` to confirm you are asking the
cluster you think you are asking — the other
classic cause is a current-context pointing at
yesterday's cluster, or at some other tool's.
Different words, different problem: a *timeout*
suggests a network or a tunnel in between; a
*certificate* error suggests the cluster was
recreated and your stored certificate authority
is stale — `minikube update-context`, or delete
and start again.

## minikube's own kubectl

Finally, the wrapper from chapter 2:
`minikube kubectl -- <kubectl args>` runs a
kubectl that minikube fetched, at the version
matching the cluster. The lab ran it with no
cluster at all and still got an answer:

```
$ minikube kubectl -- version --client
Client Version: v1.37.0
Kustomize Version: v5.8.1
```

v1.37.0 — not the v1.37.2 the lab installed
standalone in chapter 4. Neither is wrong:
kubectl works against clusters one minor version
either side of its own (chapter 9 tells the
skew rule properly). The wrapper's virtue is
that it can never be the *wrong* version for the
cluster it ships with, which makes it the safe
choice in scripts and in help given to others:
"try `minikube kubectl -- get pods`" works on
any machine that has minikube, kubectl installed
or not.
