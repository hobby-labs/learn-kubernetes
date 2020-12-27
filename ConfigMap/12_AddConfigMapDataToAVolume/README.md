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

* pod-configmap-volume.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod-12
spec:
  containers:
    - name: test-container
      image: k8s.gcr.io/busybox
      command: ["/bin/sh", "-c", "ls /etc/config/; cat /etc/config/SPECIAL_LEVEL;echo ; cat /etc/config/SPECIAL_TYPE; echo"]
      volumeMounts:
      - name: config-volume
        mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        # Provide the name of the ConfigMap containing the files you want to add to the container
        name: special-config-12
  restartPolicy: Never
```

```
$ kubectl create -f pod-configmap-volume.yaml
$ kubectl logs dapi-test-pod-12
SPECIAL_LEVEL
SPECIAL_TYPE
very
charm
```

# Reference
https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/
