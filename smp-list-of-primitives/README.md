# Strategic Merge Patching Lists of Primitives

This example illustrates how patching an array of primitives behaves differently depending on
the merge definition of the field. In this example, we attempt to strategic merge patch two separate
lists of primitives:
* `metadata.finalizers`
* `spec.loadBalancerSourceRanges`


Base: `service.yaml`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: guestbook-svc
  finalizers:
  - foo
spec:
  selector:
    app: guestbook
  ports:
  - port: 80
  loadBalancerSourceRanges:
  - 1.1.1.1
```

Patch `modify-lists.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: guestbook-svc
  finalizers:
  - bar
spec:
  loadBalancerSourceRanges:
  - 2.2.2.2
```

In the rendered output, we see that the `metadata.finalizers` list was merged, but the 
`spec.loadBalancerSourceRanges` was replaced. This is because the
[type definition of `metadata.finalizers`](https://github.com/kubernetes/apimachinery/blob/a32c0f5f25641f7248fc102f1cb3f17c2cd0b787/pkg/apis/meta/v1/types.go#L263-L264)
uses a
[merge directive](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-api-machinery/strategic-merge-patch.md#merge-directive),
whereas the
[type definition of `spec.loadBalancerSourceRanges`](https://github.com/kubernetes/api/blob/d155b85a4fda56ad0dc1ce03269c6276a4805960/core/v1/types.go#L3913)
does not.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: guestbook-svc
  # the finalizers list was merged, because the metadata.finalizers uses the SMP `merge` directive
  finalizers:
  - bar
  - foo
spec:
  ports:
  - port: 80
  selector:
    app: guestbook
  # the loadBalancerSourceRanges was replaced, because its definition does not define a merge directive
  loadBalancerSourceRanges:
  - 2.2.2.2
```