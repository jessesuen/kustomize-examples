# Renaming Resources

This example demonstrates a kustomize workaround to rename a resource using JSON 6902 patching

Base: `pod.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: old-name
spec:
  containers:
  - name: guestbook
    image: guestbook:latest
```

JSON 6902 patch `rename.yaml`:

```yaml
- op: replace
  path: /metadata/name
  value: new-name
```

Final output:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: new-name
spec:
  containers:
  - image: guestbook:latest
    name: guestbook
```