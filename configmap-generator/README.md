# ConfigMap Generator

This example demonstrates the ConfigMap generator feature of Kustomize, which allows you to generate
a ConfigMap from files. The generated ConfigMap has the default behavior of injecting a suffix
to the ConfigMap name. The suffix is a hash of the ConfigMap contents. The purpose of the hash is to
indirectly trigger a rolling update by also update the Pod spec of any resources which are
referencing the ConfigMap to use the new name with new hash. The change in the pod template would
be treated as a normal rolling update, and effectively cause the pods to restart.

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- deployment.yaml

configMapGenerator:
- name: animals-say
  files:
  - whale.txt
  - cow.txt=rename-me.txt

generatorOptions:
  annotations:
    argocd.argoproj.io/compare-options: IgnoreExtraneous

```

This results in the following ConfigMap being generated. Notice the `-b68c9cd42b` hash appended to
the name.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: animals-say-b68c9cd42b
  annotations:
    argocd.argoproj.io/compare-options: IgnoreExtraneous
data:
  cow.txt: |2-
     _______
    < moooo >
     -------
            \   ^__^
             \  (oo)\_______
                (__)\       )\/\
                    ||----w |
                    ||     ||
  whale.txt: |2-
     _______
    < woooo >
     -------
        \
         \
          \
                        ##        .
                  ## ## ##       ==
               ## ## ## ##      ===
           /""""""""""""""""___/ ===
      ~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~
           \______ o          __/
            \    \        __/
              \____\______/
```

The original Deployment spec

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: animal-sounds
spec:
  template:
    metadata:
      labels:
        app: animal-sounds
    spec:
      containers:
      - env:
        - name: COWSAY
          valueFrom:
            configMapKeyRef:
              key: cow.txt
              name: animals-say-b68c9cd42b  # renamed from `animals-say`
        image: animal-sounds:v1
        name: animal-sounds
        volumeMounts:
        - mountPath: /etc/animals-say
          name: animals-say-vol
      volumes:
      - configMap:
          name: animals-say-b68c9cd42b      # renamed from `animals-say`
        name: animals-say-vol
```