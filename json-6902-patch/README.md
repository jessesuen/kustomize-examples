# JSON 6902 Patching

This example demonstrates the use of [JSON 6902](https://tools.ietf.org/html/rfc6902) patching to
append an item to an array of primitives. This example appends a `--debug` argument to the existing
command line args. 

NOTE: When append to `spec.containers.args`, which is a list of primitives (strings),
JSON 6902 patching is required, because the
[args array in the PodTemplateSpec](https://github.com/kubernetes/api/blob/d155b85a4fda56ad0dc1ce03269c6276a4805960/core/v1/types.go#L2137)
does not specify the SMP
[merge directive](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-api-machinery/strategic-merge-patch.md#merge-directive)
for that list.

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