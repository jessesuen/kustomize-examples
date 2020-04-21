# Kustomize Vars (Variable References)

Kustomize Vars are Kustomize's weird way of templating (while still claiming they don’t do templating).
Vars are a reflection mechanism where values can _*only*_ come from a referenced field inside a
referenced object. Furthermore, Vars can only be substituted at specific locations in resources.
Mostly this means: container command, args, and env value fields. Vars are currently
under consideration for replacement ([Issue #2052](https://github.com/kubernetes-sigs/kustomize/issues/2052)).

Vars are defined in a `vars` section in the kustomization.yaml:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- service.yaml
- deployment.yaml

namePrefix: jesses-

vars:
- name: SERVICE_NAME
  objref:
    kind: Service
    name: guestbook-svc
    apiVersion: v1
  fieldref:     # optional -- if omitted, defaults to metadata.name
    fieldpath: metadata.name
```

Vars are referenced using bash variable like syntax (e.g. `$(VAR_NAME)`):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
 name: guestbook
spec:
  template:
    spec:
      containers:
      - name: guestbook
        image: guestbook:v1
        command:
        - ping 
        - $(SERVICE_NAME)
```

which would get expanded to:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jesses-guestbook
spec:
  template:
    spec:
      containers:
      - command:
        - ping
        - jesses-guestbook-svc
        image: guestbook:v1
        name: guestbook
```

Note that the VAR replacement also took into account the `namePrefix` in the kustomization.yaml.