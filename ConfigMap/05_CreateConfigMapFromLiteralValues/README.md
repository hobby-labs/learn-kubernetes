# Create ConfigMaps from literal values

```
$ kubectl create configmap special-config --from-literal=special.how=very --from-literal=special.type=charm
$ kubectl get configmaps special-config -o yaml
apiVersion: v1
data:
  special.how: very
  special.type: charm
kind: ConfigMap
......
```

# Reference
https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/

