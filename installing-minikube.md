# Installing Minikube

**Outcome: minikube and kubectl are on your machine, each verified as the genuine article by its published checksum, and `minikube version` proves the first one runs.**

Minikube is one self-contained program file.
Installing it means: download the file for your
platform from the project's official release
point, check it against the checksum the project
publishes, put it somewhere on your PATH.
kubectl is a separate download of the same shape.
Package managers (Homebrew, the Windows
installers, distribution packages) wrap the same
steps; this chapter does them by hand once,
because the hand route shows you exactly what you
have and works identically on a machine with no
package manager — which is how this guide's lab
did it (transcript `t00`).

## Download and verify minikube

The project's release bucket is
`https://storage.googleapis.com/minikube/releases/`.
For the current version on 64-bit Linux:

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64.sha256
```

Now the verification — and a trap worth meeting
in a guide rather than alone. For most projects
you would run `sha256sum -c` on the checksum
file. Minikube's `.sha256` file contains **only
the hash, with no filename**, so that command
reports "no properly formatted checksum lines
found" even though nothing is wrong. Compare the
two values directly instead:

```bash
cat minikube-linux-amd64.sha256
sha256sum minikube-linux-amd64
```

The lab's result, both lines:

```
b738496da01be06bbaf80c688f57ce25acd3849fbb518155f3a88e03ef555aa4
b738496da01be06bbaf80c688f57ce25acd3849fbb518155f3a88e03ef555aa4  minikube-linux-amd64
```

Same value: the 143 MB file that arrived is the
file the project published. (The file's size is
itself information — if your download is a few
kilobytes, you have an error page, not a binary.)
Other platforms follow the same pattern:
`minikube-darwin-amd64` / `minikube-darwin-arm64`
for macOS, `minikube-windows-amd64.exe` for
Windows, and the project's releases page on
GitHub mirrors the same files.

Make it runnable and prove it:

```bash
chmod +x minikube-linux-amd64
./minikube-linux-amd64 version
```

```
minikube version: v1.39.0
commit: 7a9f6a841470a207de8cf4bafcccee0969d8ba10
```

Then move it onto your PATH (for example as
`minikube` in `/usr/local/bin`, or in a personal
`bin` directory) so later chapters can call it
plainly. Everything else in this chapter works
the same whether the binary sits in a system
directory or a scratch folder — this guide's lab
kept it in the lab folder throughout.

A note on staying current: `minikube update-check`
reports the latest version against yours, and a
newer minikube is installed by repeating this
chapter — the program holds no state of its own
outside its home directory (chapter 2), so
replacing the file replaces the tool. Existing
clusters are a different matter from the tool's
version; chapter 9 covers Kubernetes versions.

## Download and verify kubectl

kubectl comes from the Kubernetes project's own
download point. Ask it for the current stable
version number first:

```bash
curl -sSL https://dl.k8s.io/release/stable.txt
```

The lab's answer was `v1.37.2`. Then:

```bash
curl -LO "https://dl.k8s.io/release/v1.37.2/bin/linux/amd64/kubectl"
curl -LO "https://dl.k8s.io/release/v1.37.2/bin/linux/amd64/kubectl.sha256"
echo "$(cat kubectl.sha256)  kubectl" | sha256sum -c -
```

Here the checksum file is a bare hash too, but
the conventional one-liner above supplies the
filename yourself, and the lab's answer was the
satisfying `kubectl: OK`. Prove the client runs:

```bash
chmod +x kubectl
./kubectl version --client
```

```
Client Version: v1.37.2
Kustomize Version: v5.8.1
```

`--client` matters: without it kubectl also asks
a cluster for *its* version, and with no cluster
yet that half fails. Chapter 8 puts kubectl to
work properly. One preview of a common question:
minikube can also fetch a kubectl for you —
`minikube kubectl -- version --client` answered
`v1.37.0` in the lab, the kubectl matching
minikube's default Kubernetes version, rather
than the newest stable. Chapter 9 explains why
the two numbers differ and why either is fine.
