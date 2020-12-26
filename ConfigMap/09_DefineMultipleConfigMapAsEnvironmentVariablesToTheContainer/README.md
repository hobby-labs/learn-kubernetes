# Define multiple config map as environment variables to the container

* configmap/configmaps.yaml
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: special-config
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

# Reference
https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/

