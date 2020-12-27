# Use ConfigMap-defined environment variables in Pod commands

You can use ConfigMap-defined environment variables in the `command` and `args` of a container using the `$(VAR_NAME)` Kubernetes substitution syntax.

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
              name: special-config
              key: SPECIAL_LEVEL
        - name: SPECIAL_TYPE_KEY
          valueFrom:
            configMapKeyRef:
              name: special-config
              key: SPECIAL_TYPE
  restartPolich: Never
```

# Reference
https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/
