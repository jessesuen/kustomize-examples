# One-to-Many

This example demonstrates how to achieve a one-to-many config pattern, where multiple clones of
an resource are generated from a single definition (with different names). YAML anchors and aliases
are used to avoid repetition. Also demonstrates how variants/customizations can be applied to the
clones.

Base `one-to-many.yaml`:

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
  spec: &podSpec
    containers:
    - name: guestbook
      image: guestbook:v1
# the pods defined below are clones of the pod spec above (with different names)
- apiVersion: v1
  kind: Pod
  metadata:
    name: pod-b
  spec: *podSpec
- apiVersion: v1
  kind: Pod
  metadata:
    name: pod-c
  spec: *podSpec
```

The patch `variants.yaml`, is a set of standard strategic merge patches.
This patch is configuring each pod to use different service accounts:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-a
spec:
  serviceAccountName: sa-1
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-b
spec:
  serviceAccountName: sa-2
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-c
spec:
  serviceAccountName: sa-3
```

Final output:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-a
spec:
  containers:
  - image: guestbook:v1
    name: guestbook
  serviceAccountName: sa-1
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-b
spec:
  containers:
  - image: guestbook:v1
    name: guestbook
  serviceAccountName: sa-2
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-c
spec:
  containers:
  - image: guestbook:v1
    name: guestbook
  serviceAccountName: sa-3
```
