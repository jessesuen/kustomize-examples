# Kustomize Vars with a Custom Configuration

This example shows how you can use a custom var reference configuration to allow VARs to be
substituted in places other than the default locations (container command, args, env). This example
allows VARS to be substituted in Deployment and Service label selectors. Parameterizing label
selectors is useful if you needed to deploy a variant of a Service/Deployment in the same namespace,
but using different selector labels (so that they receive distinct traffic from one another). 

`kustomization.yaml.yaml`:
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

configurations:
- var-reference-in-selectors.yaml

resources:
- deployment.yaml
- service.yaml

vars:
- name: DEPLOYMENT_NAME
  objref:
    kind: Deployment
    name: guestbook
    apiVersion: apps/v1
```

The `var-reference-in-selectors.yaml` teaches kustomize that vars should also be replaced in the
`spec.selector.app` fields (and related fields): 
```yaml
varReference:
- path: spec/selector/app
  kind: Service
  apiVersion: v1
- path: spec/selector/matchLabels/app
  kind: Deployment
  apiVersion: apps/v1
- path: spec/template/metadata/labels/app
  kind: Deployment
  apiVersion: apps/v1

```

The "base" references the `DEPLOYMENT_NAME` var as part of the selectors. 

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
 name: guestbook
spec:
  selector:
    matchLabels:
      app: $(DEPLOYMENT_NAME)
  template:
    metadata:
      labels:
        app: $(DEPLOYMENT_NAME)
    spec:
      containers:
      - name: guestbook
        image: guestbook:v1
```

When running `kustomize build` on the pr01 or pr02 directories, we see that the label selectors 
are incorporating the name of the deployment in the selectors


```yaml
apiVersion: v1
kind: Service
metadata:
  name: guestbook-svc-pr01
spec:
  ports:
  - port: 80
  selector:
    app: guestbook-pr01
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: guestbook-pr01
spec:
  selector:
    matchLabels:
      app: guestbook-pr01
  template:
    metadata:
      labels:
        app: guestbook-pr01
    spec:
      containers:
      - image: guestbook:v1
        name: guestbook
```