# Choosing a Driver

**Outcome: you can pick a driver for a machine you know, state what it will demand from that machine, and recognise each driver's characteristic failure when you meet it.**

The driver is the machinery that holds the
cluster up, and it is the only minikube decision
that is mostly made *for* you — by your operating
system, by what is already installed, and by what
your machine permits. The version this guide
verified lists its drivers in the `start` help
itself:

```
Driver is one of: virtualbox, kvm2, qemu2, qemu,
vmware, none, docker, podman, ssh
(defaults to auto-detect)
```

The drivers fall into three families.

## Containers: docker, podman

The cluster runs inside a container managed by
Docker or Podman. This is the route most people
should take on a machine that has either: no
virtual machine to boot (starts are fast), no
separate kernel (the guest shares yours), and
sizes that follow container rules. Its demand is
simple and absolute — **the daemon must be
there**. The Docker command-line tool being
installed is not the daemon; they are separate
things, and confusing them is the classic first
failure.

This guide's lab machine has exactly that shape:
a Docker client and no daemon. The lab's first
start attempt (transcript `t04`) got one line of
welcome and then:

```
* [lab] minikube v1.39.0 on Ubuntu 24.04 (amd64)
  - MINIKUBE_HOME=/home/hatch/workspace/minikube-labwork/mh
* Using the docker driver based on user configuration

X Exiting due to PROVIDER_DOCKER_VERSION_EXIT_1:
  "docker version --format <no value>-<no value>:<no value>"
  exit status 1: failed to connect to the docker API at
  unix:///var/run/docker.sock; check if the path is correct
  and if the daemon is running: dial unix
  /var/run/docker.sock: connect: no such file or directory
* Documentation: https://minikube.sigs.k8s.io/docs/drivers/docker/

exit: 69
```

Read it once and you can read any minikube
failure: the error class (`PROVIDER_DOCKER_…`),
the command that failed, the plain-language
cause (no socket file — the daemon is not
running), a documentation link, and a numeric
exit code. On your machine, the fix for this one
is to start Docker — on a desktop, launching
Docker Desktop; on a Linux server, starting the
`docker` service.

## Virtual machines: kvm2, qemu2, qemu, virtualbox, vmware

The cluster runs in a small virtual machine
booted from minikube's ISO. This is the route
when you want the cluster isolated behind a real
kernel boundary, or when containers are not
available but a hypervisor is. The demands are
the hypervisor's own: hardware virtualisation, a
kernel device (`/dev/kvm` on Linux), and usually
a permission — on Linux, membership of the
`libvirt` group for the kvm2 driver.

The lab machine fails this family too, and at an
earlier rung than the missing `/dev/kvm` (the
lab verified there is none). The kvm2 attempt
(transcript `t04`, appendix) stopped at:

```
X Exiting due to PR_KVM_USER_PERMISSION:
  libvirt group membership check failed:
  user is not a member of the appropriate libvirt group
* Suggestion: Ensure that you are a member of the
  appropriate libvirt group (remember to relogin
  for group changes to take effect!)
* Documentation: https://minikube.sigs.k8s.io/docs/reference/drivers/kvm2/

exit: 67
```

Note the shape: minikube checks the cheap
prerequisites first and names each one as it
fails. A start that stops is not minikube being
difficult; it is a list of what the machine does
not yet offer, delivered one item at a time.

## none: no holder at all

The **none** driver installs Kubernetes directly
on the host operating system — no container, no
VM. It exists for jobs like automated testing on
a disposable machine, and it has the sharpest
terms in the tool:

- It needs **root**, and root's environment: the
  cluster's tools must be on root's PATH.
- It takes over the host: Kubernetes' services
  run as the machine's own services. There is no
  boundary to delete afterwards; the official
  documentation treats the machine itself as the
  disposable part.
- It supports **only the default profile** — no
  named profiles, which the lab proved by asking
  for one (transcript `t05`):

```
X Exiting due to DRV_UNSUPPORTED_PROFILE:
  The 'none driver does not support multiple
  profiles: https://minikube.sigs.k8s.io/docs/reference/drivers/none/

exit: 56
```

Retry without a profile name and the next rung
appears — the host prerequisites. The lab's
machine has `conntrack`, `socat` and `iptables`
installed but not `crictl`, the command-line tool
for the container runtime interface, and the
attempt ended (transcript `t05`):

```
! Using the 'containerd' runtime with the 'none'
  driver is an untested configuration!

X Exiting due to GUEST_MISSING_CONNTRACK:
  Sorry, Kubernetes 1.37.0 requires crictl to be
  installed in root's path

exit: 86
```

Two honest wrinkles worth learning from. First,
the error *code* says `CONNTRACK` while the
*message* says `crictl` — read messages, not just
codes; the codes are historical, the messages are
maintained. Second, the warning above it:
minikube paired the none driver with the
containerd runtime and flagged its own choice as
untested. The none driver is a specialist tool,
and the program says so at every opportunity.
(While writing this chapter the lab tried to
fetch the none driver's documentation page for a
final check and the fetch failed; its URL is the
one the binary itself printed above, and the
terms listed here rest on the binary's own
behaviour and messages.)

## ssh: a machine elsewhere

The ssh driver builds the cluster on a remote
machine reached over SSH — for when the power (or
the permission) is somewhere else. It inherits
most of the none driver's character, one network
hop away. This guide does not use it further;
the official driver documentation is its home.

## So which one?

- **Docker (or Podman) already running on your
  machine?** Use it. This is the default for a
  reason.
- **No containers, but a hypervisor you already
  use?** kvm2/qemu2 on Linux, or your platform's
  VM driver. Expect the group-membership and
  virtualisation checks.
- **A disposable bare-metal box for testing, and
  you understand it becomes the cluster?** none,
  on the default profile, as root.
- **Letting minikube auto-detect** is respectable:
  it picks from what it finds. This guide pins
  the driver explicitly in its examples (`--driver=`
  on `start`, or `minikube config set driver`)
  because an explicit choice fails more
  informatively — as the transcripts above show.
