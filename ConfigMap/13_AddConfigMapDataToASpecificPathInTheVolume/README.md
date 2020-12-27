# Add ConfigMap data to a specific path in the Volume

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: special-config-13
  namespace: default
data:
  SPECIAL_LEVEL: very
  SPECIAL_TYPE: charm
```

```
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: k8s.gcr.io/busybox
      command: ["/bin/sh", "-c", "cat /etc/config/keys"]
      volumeMounts:
      - name: config-volume-13
```

# Reference
https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/
