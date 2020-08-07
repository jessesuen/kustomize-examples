# Unexpected Null Value

This example demonstrates a quirk/bug with kustomize where the rendered output has unexpected null values, when patching is used in conjunction with VARs. 
In this example, we have a standard resource and patch. The patch is setting the command list to an empty array:

```
kind: Deployment
apiVersion: apps/v1
metadata:
  name: deployment
spec:
  template:
    spec:
      containers:
      - name: app
        command: []
```

However, if the following VAR present:


```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- deployment.yaml

patchesStrategicMerge:
- deployment-patch.yaml

# The presence of the following VAR, will cause `command: null` to appear in final output
vars:
- name: SELECTOR
  objref:
    kind: Deployment
    name: deployment
    apiVersion: apps/v1
  fieldref:
    fieldpath: spec.selector.matchLabels.app
```

then the final output will contain `command: null`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment
spec:
  selector:
    matchLabels:
      app: service
  template:
    metadata:
      labels:
        app: service
    spec:
      containers:
      - command: null
        image: foo
        name: app
```
