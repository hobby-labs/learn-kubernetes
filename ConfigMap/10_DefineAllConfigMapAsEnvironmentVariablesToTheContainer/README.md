# Define all ConfigMap as environment variables to the container

* configmap/configmap-multikeys.yaml
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: special-config-10
  namespace: default
data:
  SPECIAL_LEVEL: very
  SPECIAL_TYPE: charm
```

```
$ kubectl create -f configmap/configmap-multikeys.yaml
```

* pod-configmap-envFrom.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod-10
spec:
  containers:
    - name: test-container
      image: k8s.gcr.io/busybox
      command: ["/bin/sh", "-c", "env"]
      envFrom:
      - configMapRef:
          name: special-config-10
  restartPolicy: Never
```

```
$ kubectl create -f pod-configmap-envFrom.yaml
$ kubectl logs dapi-test-pod-10
...
SPECIAL_LEVEL=very
...
SPECIAL_TYPE=charm
...
```

# Reference
https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/
