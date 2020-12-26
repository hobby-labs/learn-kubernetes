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
```

