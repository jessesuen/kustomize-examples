# Strategic Merge Patch (SMP) with Custom Resources

This example is identical to the [strategic-merge-patch](../strategic-merge-patch) example,
except for the fact that it uses a custom resource (`foo/v1 Pod`) instead of a native Kuberentes
kind (`v1 Pod`). This example highlights kustomize's inability to strategic merge patch an array
inside unknown resource kinds (i.e. custom resources), and demonstrates the fact that it falls back
to standard json merge patching.

Base: `pod.yaml`:
```yaml
apiVersion: foo/v1
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
apiVersion: foo/v1
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

The `kustomize build` results in a container array where the entire `spec.containers` array was replaced with the array in the patch:
```yaml
apiVersion: foo/v1
kind: Pod
metadata:
  name: guestbook
spec:
  # the entire containers array has been replaced (not merged) from the patch
  containers:
  - name: guestbook
    # `image: guestbook:latest` is now missing due to the replacement of the container array
    resources:
      limits:
        memory: 1Gi
    # json merge patching does not understand SMP directives (e.g. $patch)
    # so the directive is preserved literally
  - $patch: delete
    name: haproxy
  - name: nginx
    image: nginx:latest
```