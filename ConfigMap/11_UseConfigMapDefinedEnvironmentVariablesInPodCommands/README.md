# Use ConfigMap-defined environment variables in Pod commands

You can use ConfigMap-defined environment variables in the `command` and `args` of a container using the `$(VAR_NAME)` Kubernetes substitution syntax.

* configmap/configmap-multikeys.yaml
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: special-config-11
  namespace: default
data:
  SPECIAL_LEVEL: very
  SPECIAL_TYPE: charm
```

```
$ kubectl create -f configmap/configmap-multikeys.yaml
$ kubectl get configmap special-config-11 -o yaml
apiVersion: v1
data:
  SPECIAL_LEVEL: very
  SPECIAL_TYPE: charm
kind: ConfigMap
...
```

* pod-configmap-env-var-valueFrom.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod-11
spec:
  containers:
    - name: test-container
      image: k8s.gcr.io/busybox
      command: ["/bin/echo", "$(SPECIAL_LEVEL_KEY)", "$(SPECIAL_TYPE_KEY)"]
      env:
        - name: SPECIAL_LEVEL_KEY
          valueFrom:
            configMapKeyRef:
              name: special-config-11
              key: SPECIAL_LEVEL
        - name: SPECIAL_TYPE_KEY
          valueFrom:
            configMapKeyRef:
              name: special-config-11
              key: SPECIAL_TYPE
  restartPolicy: Never
```

```
$ kubectl create -f pod-configmap-env-var-valueFrom.yaml
$ kubectl logs dapi-test-pod-11
very charm
```

# Reference
https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/
