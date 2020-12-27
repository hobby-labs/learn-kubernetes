# Add ConfigMap data to a specific path in the Volume

* configmap/configmap-multikeys.yaml
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: special-config-13
  namespace: default
data:
  SPECIAL_LEVEL: very
  SPECIAL_TYPE: charm
```

```
$ kubectl create -f configmap/configmap-multikeys.yaml
```

* pod-configmap-volume-specific-key.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod-13
spec:
  containers:
    - name: test-container
      image: k8s.gcr.io/busybox
      command: ["/bin/sh", "-c", "cat /etc/config/keys"]
      volumeMounts:
      - name: config-volume
        mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: special-config-13
        items:
        - key: SPECIAL_LEVEL
          path: keys
  restartPolicy: Never
```

```
$ kubectl create -f pod-configmap-volume-specific-key.yaml
$ kubectl logs dapi-test-pod-13
very
```

# Reference
https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/
