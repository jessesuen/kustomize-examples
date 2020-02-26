# Reusing a Strategic Merge Patch

This example demonstrates a workaround to re-use strategic merge patch
by using YAML anchors and aliases, and the Kubernetes List kind.

The base `pods.yaml` contains three independently defined pods:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-a
spec:
  containers:
  - name: guestbook
    image: guestbook:v1
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-b
spec:
  containers:
  - name: guestbook
    image: guestbook:v2
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-c
spec:
  containers:
  - name: guestbook
    image: guestbook:v3
```

The SMP patch re-use is accomplished by using a Kubernetes List as a patch, and leveraging
YAML anchors and aliases to avoid repeating the hostAlias object multiple times.

Patch `host-aliases.yaml`:
```yaml
# The Kubernetes List kind is used, because YAML anchors and aliases can only reference
# objects in the same document
apiVersion: v1
kind: List
items:
- apiVersion: v1
  kind: Pod
  metadata:
    name: pod-a
  spec: &hostAliases  # Set an YAML anchor for .spec, called `hostAliases`
    hostAliases:
    - ip: 1.1.1.1
      hostnames:
      - a.example.com
    - ip: 2.2.2.2
      hostnames:
      - b.example.com
- apiVersion: v1
  kind: Pod
  metadata:
    name: pod-b
  spec: *hostAliases  # Reference & inject the `hostAliases` anchor with an YAML alias
- apiVersion: v1
  kind: Pod
  metadata:
    name: pod-c
  spec: *hostAliases
```

The final output contains the three original pods, each with the new `hostAliases` added
to each one.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-a
spec:
  containers:
  - image: guestbook:v1
    name: guestbook
  hostAliases:
  - hostnames:
    - a.example.com
    ip: 1.1.1.1
  - hostnames:
    - b.example.com
    ip: 2.2.2.2
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-b
spec:
  containers:
  - image: guestbook:v2
    name: guestbook
  hostAliases:
  - hostnames:
    - a.example.com
    ip: 1.1.1.1
  - hostnames:
    - b.example.com
    ip: 2.2.2.2
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-c
spec:
  containers:
  - image: guestbook:v3
    name: guestbook
  hostAliases:
  - hostnames:
    - a.example.com
    ip: 1.1.1.1
  - hostnames:
    - b.example.com
    ip: 2.2.2.2
```