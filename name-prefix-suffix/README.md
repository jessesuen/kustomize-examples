# Kustomize Name Prefix and Suffix Transformers

Kustomize has a feature to append or prepend a string to all resource names through the use of `nameSuffix` and `namePrefix` transformers.

To use the feature, add `namePrefix:` and/or `nameSuffix:` to the kustomization.yaml:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- deployment.yaml

namePrefix: jesses-
nameSuffix: -prod
```

The original resource object would be modified to contain the prefix and/or suffix added to the name.

Original resource:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: guestbook
spec:
  template:
    spec:
      containers:
      - image: guestbook:v1
        name: guestbook
```

Final result:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jesses-guestbook-prod
spec:
  template:
    spec:
      containers:
      - image: guestbook:v1
        name: guestbook
```
