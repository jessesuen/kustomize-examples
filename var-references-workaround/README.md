# Kustomize Vars Poor Man's Templating

This example demonstrates a trick users have done to get a "poor-man's" templating capability in
kustomize. The idea is to define a dummy ConfigMap using a conigMapGenerator with literal values.
This ConfigMap will never be used by any pods, it is simply used for the var reference

```yaml
configMapGenerator:
- name: kustomize-vars
  literals:
  - foo=spam
  - bar=ham
  - baz=eggs
generatorOptions:
  disableNameSuffixHash: true
```

Each VAR should reference the keys of the ConfigMap that it will be made available as a var.

```yaml
vars:
- name: FOO
  objref:
    kind: ConfigMap
    name: kustomize-vars
    apiVersion: v1
  fieldref:
    fieldpath: data.foo
- name: BAR
  objref:
    kind: ConfigMap
    name: kustomize-vars
    apiVersion: v1
  fieldref:
    fieldpath: data.bar
- name: BAZ
  objref:
    kind: ConfigMap
    name: kustomize-vars
    apiVersion: v1
  fieldref:
    fieldpath: data.baz

```

A resource can reference the vars:

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: guestbook
spec:
  containers:
  - name: guestbook
    image: guestbook:v1
    command:
    - echo $(FOO) $(BAR) $(BAZ)

```

which would get expanded to:

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: guestbook
spec:
  containers:
  - name: guestbook
    image: guestbook:v1
    command:
    - echo spam ham eggs
```
