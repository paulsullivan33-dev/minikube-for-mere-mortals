# Troubleshooting

**Outcome: a minikube problem lands, and you can sort it into its family — driver, permission, prerequisite, profile, or kubectl-side — from the words of the error itself.**

## First, read the error properly

Minikube's failures follow one grammar, and
learning it is most of troubleshooting:

```
X Exiting due to <ERROR_CLASS>: <what failed, in
  plain words>
* Suggestion: <often, a concrete fix>
* Documentation: https://minikube.sigs.k8s.io/…
exit: <a number>
```

The **class** tells you the family (the prefixes
below), the **message** tells you the specific
thing, and the exit code is a stable number you
can search or script against. Quoted throughout
this chapter are the lab's real failures
(transcripts `t04`, `t05`, `t07`) — every entry
here was met, not imagined.

## "…failed to connect to the docker API… no such file or directory"

```
PROVIDER_DOCKER_VERSION_EXIT_1 … dial unix
/var/run/docker.sock: connect: no such file or
directory        exit: 69
```

The Docker **client** is installed; the
**daemon** is not running. (`PROVIDER_` classes
are the driver reporting that its machinery
answered wrongly or not at all.) Start Docker —
Docker Desktop on a desktop machine, the docker
service on a Linux host — then start again. The
same shape with podman means its service/socket
is not up.

## "libvirt group membership check failed"

```
PR_KVM_USER_PERMISSION: user is not a member of
the appropriate libvirt group        exit: 67
```

`PR_` classes are **prerequisites**: cheap checks
minikube runs before building anything. The fix
is in the message — join the group, and *log out
and back in*, because group membership is read at
login. The family generalises: missing
virtualisation, missing permissions, unsupported
hosts all arrive as `PR_` messages naming the
one thing to fix. Fix exactly that thing; the
next start will check the next one.

## "Sorry, Kubernetes … requires crictl to be installed in root's path"

```
GUEST_MISSING_CONNTRACK (the message, not the
code, says crictl)        exit: 86
```

None driver only. `GUEST_` classes concern the
environment Kubernetes is being installed into —
here, the host itself, which lacks a tool the
bootstrapper needs, on **root's** PATH
specifically. Two morals: read the message over
the code (the codes are historical — this one
says CONNTRACK), and remember the none driver
wants host-level tools installed by you, as a
deliberate act, on a machine you mean it for.

## "The 'none driver does not support multiple profiles"

```
DRV_UNSUPPORTED_PROFILE        exit: 56
```

`DRV_` classes are the driver refusing the
*request* (as opposed to the machine). Here:
drop the `-p` name and use the default profile,
or choose another driver. A driver that cannot
do what you asked says so before touching
anything — take the hint literally.

## "No minikube profile was found" / "Profile … not found"

```
MK_USAGE_NO_PROFILE        exit: 14
Profile "lab" not found …    exit: 85
```

`MK_` classes are minikube usage problems: you
asked about a cluster that does not exist.
`minikube profile list` shows what does; `start`
creates what does not. And its cousin:

## "connection refused" from kubectl

```
The connection to the server 127.0.0.1:8443 was
refused - did you specify the right host or port?
```

Not a network fault: the address is right and
nothing is listening. The cluster is stopped,
unstarted, or deleted — or kubectl's
current-context names a different (stopped)
cluster. In order: `kubectl config
current-context`, `minikube status -p <that
name>`, `minikube start -p <that name>`. If the
words are instead *certificate* errors, the
cluster was recreated under you: `minikube
update-context`, or delete and start again.
Chapter 8 takes this error apart line by line.

## A start that stops somewhere new each time

That is the prerequisite ladder working as
designed (chapter 5). Fix the named rung, run
again, meet the next. If you suspect the profile
itself is wedged — half-created, inconsistent —
the broom is `minikube delete -p <name>` (it
removes even failed profiles; transcript `t07`)
and a fresh start.

## "Metrics API not available" from kubectl top

Not an error in your cluster; the metrics-server
addon is not enabled. `minikube addons enable
metrics-server` (chapter 10).

## Collecting evidence like the lab did

- `minikube logs` gathers the cluster-side logs;
  minikube's own error footer suggests attaching
  them (and its session log file) to a GitHub
  issue.
- `minikube options` lists the global flags,
  including `-p/--profile` — when a command seems
  to ignore your cluster, check which profile you
  are actually addressing.
- Exit codes are stable per failure: the numbers
  quoted in this chapter (69, 67, 86, 56, 14, 85,
  and `config set`'s 10) came off the lab's runs
  and will match yours for the same failures.
