# Image Transformation

This example demonstrates how kustomize 
[automatically detects](https://github.com/kubernetes-sigs/kustomize/blob/4a7ade6421620b1ef4cccf9162374af35b6f6db9/plugin/builtin/imagetagtransformer/ImageTagTransformer.go#L84) 
portions of the spec where container images might be used, and replaces them using image transformation.

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

The `kustomization.yaml` uses image transformation:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- foo-pod.yaml

images:
- name: guestbook
  newTag: v2
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
            # the image field below was updated by image transformation, because the containers
            # structure is something which appears similar to native k8s types
          - image: guestbook:v2
            name: guestbook
          containerz:
            # the image field below was NOT be updated by image transformation since the structure
            # does not match any known signature (since containerz was spelled with a 'z')
          - image: guestbook:REPLACEME
            name: guestbook
```