# JSON Merge Patch

This example demonstrates basic JSON merge patching by modifying the annotations map with a JSON merge patch. It uses `null` to remove an
existing map field.

Base: `pod.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: guestbook
  annotations:
    foo: spam
    bar: ham
spec:
  containers:
  - name: guestbook
    image: guestbook:latest
```

Patch: `modify-annotations.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: guestbook
  annotations:
    foo: null    # remove the foo annotation
    baz: eggs    # add the baz annotation
```

The `kustomize build` results in a merged annotation map removing the foo key, and adding baz:

```yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    bar: ham
    baz: eggs
  name: guestbook
spec:
  containers:
  - image: guestbook:latest
    name: guestbook
```