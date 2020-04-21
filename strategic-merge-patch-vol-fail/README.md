# Strategic Merge Patch (SMP) Volume Mount Failure

This example demonstrates strategic merge patch (SMP) gotcha/inability
to patch a volumeMount by name. This is because the Kubernetes type
definitions for 
[the VolumeMount array](https://github.com/kubernetes/api/blob/d04500c8c3dda9c980b668c57abc2ca61efcf5c4/core/v1/types.go#L2113-L2115) specifies mountPath as the merge key (as opposed to name). Since the patch contains a different mountPath, SMP will
append the object when patching instead of replace.

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
    volumeMounts:
    - name: logs
      mountPath: /usr/local/tomcat/logs/
  volumes:
  - name: logs
    emptyDir: {}
```

The patch: `modify-volume-mounts.yaml` attempts to change the volumeMount mountPath location:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: guestbook
spec:
  containers:
  - name: guestbook
    volumeMounts:
    - name: logs
      mountPath: /app/logs/
```

The `kustomize build` results in a volumeMount array where the patch object is prepended in the array
(as opposed to replace) and results in two `logs` entries in the volumeMounts array:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: guestbook
spec:
  containers:
  - image: guestbook:latest
    name: guestbook
    volumeMounts:
    - mountPath: /app/logs/
      name: logs
    - mountPath: /usr/local/tomcat/logs/
      name: logs
  volumes:
  - emptyDir: {}
    name: logs
```