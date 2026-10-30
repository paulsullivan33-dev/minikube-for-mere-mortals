# Writing criteria for this guide

Every "for Mere Mortals" guide is written to the same nine-point
standard, so a reader who trusts one can trust the rest. Here it
is, with the verification notes for this particular guide below.

1. **Fix it first.** Reader outcomes lead, not action items: each
   chapter opens by naming the outcome it delivers, shows a check
   the reader can run to confirm they got it, and a reasonable
   reader finishes able to state what was done, what it means,
   and what to try next.
2. **Plain language.** Short sentences, everyday vocabulary,
   technical terms defined on first use. No slang, and no
   idioms that only make sense in one country — the reader may
   be translating as they go.
3. **No dead ends.** A guide never tells the reader to do
   something it cannot show them how to verify. Every claim a
   reader relies on — an install, a command, an output — is
   something this guide demonstrates or clearly labels
   documented-only below.
4. **Live-verified where it matters.** Procedures are run for
   real in the guide's lab wherever the build machine allows it,
   and the transcripts kept in the repository are the evidence.
   What could not be run is listed honestly below, never
   presented as if it had been.
5. **General audiences.** Nothing in this guide — examples,
   fixtures, names, addresses — is modelled on any particular
   reader or author's real life, network, or organisations.
   The lab's profiles are named `learning`, `lab` and the like,
   and its one credential is a token labelled not-real.
6. **Standalone.** A reader needs this guide, its lab, and
   nothing else. Where another guide would genuinely help, it
   is pointed to once, by name, and the chapter still completes
   without it.
7. **Honest about limits.** Chapter 14 names what the tool
   cannot do: one machine pretending to be several, a cluster
   that sleeps with its host, performance and network results
   that do not transfer, storage that is a file, prerequisites
   some machines cannot meet, and clusters that exist only in
   the versions a given minikube build knows.
8. **The ethics rule is in the text, early.** Minikube raises no
   interception question, so this guide's early lines are its
   honesty disclosures, stated in the README and chapter 1
   before any procedure depends on them: a minikube cluster is
   disposable and not a production tool, the kubeconfig file is
   a credential in plain text (chapter 8, where it is handled),
   and this lab never ran a cluster — no chapter borrows the
   authority of a run that did not happen.
9. **Builds clean.** The site builds with MkDocs in strict mode
   with no warnings, and a fresh reader following the chapters
   in order reaches the stated outcome of each.

## Verification notes for this guide

The build machine is a 2-CPU, ~7.7 GB Ubuntu 24.04 sandbox,
**shared with other work while the lab ran** — four other guide
builds were in progress on it — so the lab kept its state in a
scratch directory, used only its assigned loopback port range
(it needed none), and touched nothing outside its own folders.
The machine's pre-existing `~/.minikube` (dated 2026-10-17,
belonging to other work, with a v1.31.0 cache) was never read
beyond confirming it existed, and never modified.

The decisive fact about this lab: **the machine cannot hold a
cluster.** It has a Docker client but no Docker daemon
(`docker version` answers for the client and reports the
socket missing), no `/dev/kvm`, and lacks the `crictl` tool
the none driver requires. So no `minikube start` ever
succeeded here, and this guide does not pretend otherwise.
What that leaves is still substantial, and it splits cleanly:

### Run for real, with transcripts as evidence

All transcripts live in `minikube-lab/transcripts/`. Versions:
**minikube v1.39.0** (commit 7a9f6a84…), **kubectl v1.37.2**
client (Kustomize v5.8.1), downloaded 2026-10-30.

- Downloads and checksums (t00): minikube from the official
  release bucket, its published SHA-256 (`b738496d…`) matching
  the computed hash — including the recorded fact that the
  published `.sha256` file contains a bare hash with no
  filename, so `sha256sum -c` rejects it; kubectl from
  dl.k8s.io at the version in `stable.txt`, checksum OK;
  `minikube version` and `kubectl version --client` outputs.
- The full client surface (t01, t01b): `minikube --help` (the
  whole command taxonomy of chapter 2), `minikube config
  --help` (the configurable-fields list of chapter 6), and
  `minikube start --help` complete — 96 flags in its options
  list, from which every flag this guide teaches is taken
  (drivers list,
  `--cpus='2'`, `--disk-size='20000mb'`, `--memory` with no
  printed default, `--nodes=1`, `--wait` defaulting to
  `apiserver,system_pods`, `--kubernetes-version` defaulting
  to 'stable' = v1.37.0 in this build). The first capture of
  the start help was cut short by a display pipe and was
  recaptured in full as t01b rather than patched.
- The config file, end to end (t02): `set` × 4, `get`, `view`,
  `unset`, the resulting `config.json` read off disk, the
  "take effect upon a minikube delete and then a minikube
  start" warning, `config defaults driver`, and two verbatim
  refusals (unknown property, invalid driver — both exit 10).
  Also established by doing it: with `MINIKUBE_HOME` set,
  state nests in a `.minikube` folder inside it.
- The addons catalogue (t03): `minikube addons list` with no
  cluster in existence — 41 addons with maintainers, exit 0
  (and its one stderr complaint about the missing profiles
  directory, kept in the transcript).
- Three failed starts, each verbatim (t04, t05): docker
  driver, `PROVIDER_DOCKER_VERSION_EXIT_1`, exit 69 (no
  daemon); kvm2 driver, `PR_KVM_USER_PERMISSION`, exit 67
  (libvirt group; `/dev/kvm` independently shown absent);
  none driver, first `DRV_UNSUPPORTED_PROFILE`, exit 56 (a
  named profile), then `GUEST_MISSING_CONNTRACK` in code /
  `crictl` in the message, exit 86. The prerequisite audit
  behind the last one is in t04: conntrack, socat and
  iptables present, crictl absent. The none-driver attempts
  ran with stdin closed, a time cap, and all state confined
  to the scratch `MINIKUBE_HOME`; they wrote only profile
  skeletons and stopped at their own prerequisite checks —
  no packages were installed and nothing on the host was
  modified.
- kubectl's client half (t06): an empty kubeconfig rendered
  (`config view` creates no file — checked), a cluster/user/
  context built by hand with `set-cluster`,
  `set-credentials`, `set-context`, `use-context`, the file
  read back whole, and `kubectl cluster-info` against the
  empty address producing the verbatim connection-refused
  sequence (exit 1). The token used is the literal string
  `example-token-not-real`; the addendum proves the
  `<redacted>` in `config view` is kubectl's own doing
  (0 occurrences of the token in view output written to a
  file; 1 in the kubeconfig itself).
- Profiles after failure (t07): `profile list` with none
  existing (`MK_USAGE_NO_PROFILE`, exit 14), `status` for a
  missing profile (exit 85), `delete -p` of a failed
  profile, `delete --all` sweeping all four attempted
  profiles, and the list empty again afterwards.
- The wrapper and the version range (t07, appendix):
  `minikube kubectl -- version --client` answering v1.37.0
  with no cluster present, and `minikube config defaults
  kubernetes-version` listing v1.37.0 down to v1.28.0.
- Delete/stop help (t08): `--all` and `--purge` quoted from
  `minikube delete --help` before the recipes chapter relied
  on them.

### Documented only — described from the project's documentation, not run

Sources read on 2026-10-30 and listed in
`minikube-lab/sources.txt`:

- Everything from a successful `minikube start` onward:
  the start sequence's later stages, `minikube status`
  of a running cluster, `kubectl get nodes`, stopping and
  restarting with state intact, multi-node and `--ha`
  clusters, and the image workflows (`docker-env`,
  `image load`, `cache`).
- Addons *enabling* (chapter 10 covers the catalogue from
  live evidence; the enable/disable operations and the
  per-addon behaviour notes are the official addons
  documentation's).
- All of chapter 11: `minikube dashboard` and `--url`,
  `minikube service --url` and the NodePort walkthrough,
  `minikube tunnel` for LoadBalancer services, the
  30000–32767 range — quoted from the official dashboard
  and accessing-apps handbook pages.
- The cluster-side steps inside chapter 12's recipes
  (applying manifests, `kubectl top`), built from the
  same documentation around live-verified client steps.
- The none driver's general terms in chapter 3 beyond
  what the binary itself demonstrated (root's PATH,
  default profile, untested-runtime warning — those three
  are live). One attempt to fetch its documentation page
  for a final check failed and was not retried; the
  chapter says so.
- The neighbours of chapter 13 (kind, k3d, Docker
  Desktop's Kubernetes), characterised from their own
  project descriptions. K3s is the exception: this series
  ran it for real in its own guide.

Fixtures are this guide's own and deliberately generic:
profile names are `lab`, `learning`, `rehearsal`,
`throwaway`; the practice cluster is at
`https://127.0.0.1:8443`; the only credential anywhere is
the string `example-token-not-real`. No real cluster,
account, or organisation appears anywhere in the guide or
the lab.
