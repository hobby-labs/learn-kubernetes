# Devine ConfigMap as environment variables to the container

```
$ kubectl create configmap special-config-08 --from-literal=special.how=very
```

Assign the special.how value defined in the ConfigMap to the `SPECIAL_LEVEL_KEY` environment variable in the Pod specification.

* pod-single-configmap-env-variable.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod-08
spec:
  containers:
    - name: test-container
      image: k8s.gcr.io/busybox
      command: ["/bin/sh", "-c", "env"]
      env:
        # Define the environment variable
        - name: SPECIAL_LEVEL_KEY
          valueFrom:
            configMapKeyRef:
              # The ConfigMap containing the value you want to assign to SPECIAL_LEVEL_KEY
              name: special-config-08
              # Specify the key associated with the value
              key: special.how
  restartPolicy: Never
```

```
$ kubectl create -f pod-single-configmap-env-variable.yaml
$ kubectl logs dapi-test-pod-08
...
SPECIAL_LEVEL_KEY=very
...
```

# Reference
https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/

