# Add ConfigMap data to a Volume

* configmap/configmap-multikeys.yaml
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: special-config-12
  namespace: default
data:
  SPECIAL_LEVEL: very
  SPECIAL_TYPE: charm
```

```
$ kubectl create -f configmap/configmap-multikeys.yaml
```

# Reference
https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/
