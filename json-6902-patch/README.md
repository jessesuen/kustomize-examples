# JSON 6902 Patching

This example demonstrates the use of [JSON 6902](https://tools.ietf.org/html/rfc6902) patching to
append an item to an array. This example appends a `--debug` argument to the existing command line
args. When appending to an array of primitives, JSON 6902 patching is required, because there is no
"merge key" available to perform strategic merge patching.

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
    command:
    - server
    args:
    - --redis
    - redis:6379
```

JSON 6902 patch `add-cli-arg.yaml`:

```yaml
- op: add
  path: /spec/containers/0/args/-
  value: --debug
```

Final output:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: guestbook
spec:
  containers:
  - args:
    - --redis
    - redis:6379
    - --debug
    command:
    - server
    image: guestbook:latest
    name: guestbook
```