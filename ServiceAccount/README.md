# Configure Service Accounts for Pods

https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/

## Use the Default Service Account to access the API server
If you do not specify any account name, Pod will assigned the `default` service account.

```
apiVersion: v1
kind: Pod
metadata:
  name: service-account-test
spec:
  containers:
  - image: k8s.gcr.io/busybox
    name: service-account-test
    command: ["tail", "-f", "/dev/null"]
```

```
$ kubectl exec -ti service-account-test -- bash -c "cd /var/run/secrets/kubernetes.io/serviceaccount && ls -l"
lrwxrwxrwx    1 root     root            13 Nov 15 03:10 ca.crt -> ..data/ca.crt
lrwxrwxrwx    1 root     root            16 Nov 15 03:10 namespace -> ..data/namespace
lrwxrwxrwx    1 root     root            12 Nov 15 03:10 token -> ..data/token
```

You can also specify not to mount the secret volume with `automountServiceAccountToken: false`.

```
apiVersion: v1
kind: Pod
metadata:
  name: service-account-test
spec:
  containers:
  - image: k8s.gcr.io/busybox
    name: service-account-test
    command: ["tail", "-f", "/dev/null"]
  automountServiceAccountToken: false
```

Then kubectl apply it, `/var/run/secrets` will not be mounted on the Pod

```
$ kubectl exec -ti service-account-test -- ls -l /var/run/secrets
ls: /var/run/secrets: No such file or directory
command terminated with exit code 1
```

TODO:

