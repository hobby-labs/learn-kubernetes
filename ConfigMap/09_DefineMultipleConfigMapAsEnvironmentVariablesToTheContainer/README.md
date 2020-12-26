# Define multiple config map as environment variables to the container

* configmap/configmaps.yaml
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: special-config-09
  namespace: default
data:
  special.how: very
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: env-config
  namespace: default
data:
  log_level: INFO
```

```
$ kubectl create -f configmap/configmaps.yaml
```

* pod-multiple-configmap-env-variable.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod-09
spec:
  containers:
  - name: test-container
    image: k8s.gcr.io/busybox
    command: ["/bin/sh", "-c", "env"]
    env:
      - name: SPECIAL_LEVEL_KEY
        valueFrom:
          configMapKeyRef:
            name: special-config-09
            key: special.how
      - name: LOG_LEVEL
        valueFrom:
          configMapKeyRef:
            name: env-config
            key: log_level
  restartPolicy: Never
```

```
$ kubectl create -f pod-multiple-configmap-env-variable.yaml
```

# Reference
https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/

