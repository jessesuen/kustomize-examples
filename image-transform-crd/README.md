# Image Transformation

This example demonstrates how kustomize is able to automatically detect places where
images are used in a spec, and replace them with the image transformer.

Base `foo-pod.yaml`:

```yaml
apiVersion: v1 
kind: FooPod   # use a kind unknown to kustomize
metadata:
  name: my-foo-pod
spec:
  random:
    path:
      which:
        has:
          containers:
          - name: guestbook
            image: guestbook:REPLACEME
          containerz: # containerz with a 'z'
          - name: guestbook
            image: guestbook:REPLACEME
```

Final output:

```yaml
apiVersion: v1
kind: FooPod
metadata:
  name: my-foo-pod
spec:
  random:
    path:
      which:
        has:
          containers:
            # the image field below was updated by image transformation, because the
            # structure is similar to k8s types
          - image: guestbook:v2
            name: guestbook
          containerz:
            # the image field below was NOT be updated by image transformation since the structure
            # does not match any known signature (since containerz was spelled with a 'z')
          - image: guestbook:REPLACEME
            name: guestbook
```