# Strategic Merge Patch (SMP)

This example demonstrates strategic merge patching by merging the container list.
It leverages the `$delete` SMP directive, to delete an element from a list.

Note that it is not possible to SMP custom resources.
See [#1510](https://github.com/kubernetes-sigs/kustomize/issues/1510),
[#1846](https://github.com/kubernetes-sigs/kustomize/issues/1846),
[1715](https://github.com/kubernetes-sigs/kustomize/issues/1715).
Also See the
[CRD version of this example](../strategic-merge-patch-crd-fail)
to illustrate this point: 

Base: `pod.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: guestbook
spec:
  containers:
  - name: guestbook
    image: guestbook:latest
  - name: haproxy
    image: haproxy:latest
```

Patch: `modify-containers.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: guestbook
spec:
  containers:
  # 1. update memory limit of the guestbook container
  - name: guestbook
    resources:
      limits:
        memory: 1Gi
  # 2. delete the haproxy sidecar
  - $patch: delete
    name: haproxy
  # 3. add a new nginx sidecar
  - name: nginx
    image: nginx:latest
```

The `kustomize build` results in a merged container list containing the changes
defined in the patch:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: guestbook
spec:
  containers:
  - image: guestbook:latest
    name: guestbook
    resources:
      limits:
        memory: 1Gi
  - image: nginx:latest
    name: nginx
```