# The Config File

**Outcome: you can set defaults that every future start obeys, read them back, undo them — and open the actual file they live in.**

Command-line flags speak once. The **config
file** speaks every time: values stored there
become the defaults for every `minikube start`
until you change them. The tool for it is
`minikube config`, with five subcommands — `set`,
`get`, `view`, `unset`, and `defaults` — and
everything in this chapter was run for real in
the lab, against a scratch `MINIKUBE_HOME` so
the machine's real settings were never touched
(transcript `t02`).

The configurable fields are listed by
`minikube config --help` itself: `driver`,
`cpus`, `memory`, `disk-size`,
`kubernetes-version`, `container-runtime`,
`profile`, `bootstrapper`, `insecure-registry`,
and a tail of notifications and driver-specific
settings. If a name is not on that list, `set`
refuses it — the lab tried:

```
$ minikube config set banana 3
X Exiting due to MK_CONFIG_SET: Set failed: find
  settings for "banana" value of "3": property
  name "banana" not found

exit: 10
```

Values are validated too, not just names:

```
$ minikube config set driver banana
X Exiting due to MK_CONFIG_SET: Set failed: run
  validations for "driver" with value of
  "banana": [driver "banana" is not supported]

exit: 10
```

## Setting, getting, viewing

```bash
minikube config set driver docker
minikube config set cpus 2
minikube config set memory 3072
minikube config set kubernetes-version v1.37.0
```

Some of those answer with a warning worth
understanding:

```
! These changes will take effect upon a minikube
  delete and then a minikube start
```

Config writes the *defaults for the next
cluster*, not instructions to a running one.
Changing the driver or the version of a cluster
that exists means deleting and recreating it —
minikube's clusters are disposable by design, so
that is a small threat, but it is the rule.

Reading back, one value or all of them:

```bash
$ minikube config get driver
docker

$ minikube config view
- cpus: 2
- driver: docker
- kubernetes-version: v1.37.0
- memory: 3072
```

And undoing is symmetric:

```bash
minikube config unset memory
```

after which `config view` listed the remaining
three. On a fresh machine, before anything is
set, `config view` prints nothing at all and
exits successfully — an empty answer, not an
error. That emptiness is itself meaningful: every
setting minikube uses has a built-in default
until you replace it.

## The file itself

There is no mystery format. The lab's four sets
and one unset left this file — at
`$MINIKUBE_HOME/.minikube/config/config.json`
(chapter 2 explains the nested `.minikube`
folder), on a default install `~/.minikube/config/config.json`:

```json
{
    "cpus": "2",
    "driver": "docker",
    "kubernetes-version": "v1.37.0"
}
```

Plain JSON, values stored as strings. You *can*
edit it by hand — it is yours — but `config set`
is the better habit: it validates the name and
the value, as the two refusals above show, and a
hand-edited typo would only surface later, at a
less convenient moment.

## defaults: what the binary will accept

`minikube config defaults <field>` lists the
valid values, or the known range, for a field:

```bash
$ minikube config defaults driver
* virtualbox
* kvm2
* qemu2
* qemu
* vmware
* none
* docker
* podman
* ssh
```

For `kubernetes-version` the same command lists
every Kubernetes version this minikube build
will install — in the lab's build, from v1.37.0
down through v1.28.0, including the release
candidates and betas (transcript `t07`,
appendix). Two uses follow: it tells you whether
the version you need for a test is even
available, and it quietly dates the tool — a
minikube that has never heard of your target
version needs updating first (chapter 4). Not
every field has defaults to list; asking for one
that does not is a usage error, which the lab
also met and which names the remedy (`minikube
config defaults -h`).

## Flags, config, and profiles

Three layers decide what a start does, in this
order: a **flag** on this command beats the
**config file**, which beats the **built-in
default**. And per-cluster choices — the driver
a profile was created with, its version, its
size — are recorded in the profile itself, under
`~/.minikube/profiles/<name>/`. The config file
is where your *habits* live; profiles, next
chapter, are where individual clusters live.
