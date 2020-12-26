# Create a ConfigMap from generator(kustomization.yaml) and literals

* kustomization.yaml
```
configMapGenerator:
- name: special-config-2
  literals:
  - special.how=very
  - special.type=charm
```

```
$ kubectl apply -k .
$ kubectl get configmap
NAME                          DATA   AGE
...
special-config-2-c92b5mmcf2   2      31s
...

$ kubectl get configmap special-config-2-c92b5mmcf2 -o yaml
apiVersion: v1
data:
  special.how: very
  special.type: charm
kind: ConfigMap
......
```

# Reference
https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/

