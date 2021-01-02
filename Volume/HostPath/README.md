# HostPath

HostPath may be useful if, for example...

* Running container that needs access to Docker internals. Use a `hostPath` of `/var/lib/docker`
* Running cAdvisor in a container. Use a `hostPath` of `/sys`
* 

## Types

| Value             | Behavior  |
| ----------------- | --------- |
| (Empty)           | Empty string (default) is for backward compatibility, which means that no checks will be performed before mounting the hostPaht volume. |
| DirectoryOrCreate | If nothing exists at the given path, an empty directory will be created there as needed with permission set to 0755, having the same group and ownership with Kubelete. |
| Directory         | A directory must exist at the given path |
| FileOrCreate      | If nothing exists at the given path, an empty file will be created there as needed with permission set to 0644, having the same group and ownership with Kubelete.|
| File              | A file must exist at the given path |
| Socket            | A UNIX socket must exist at the given path |
| CharDevice        | A character device must exist at the given path |
| BlockDevice       | A block device must exist at the given path |

* hostpath-type-directory.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: test-hostpath-type-directory
spec:
  containers:
  - image: k8s.gcr.io/busybox
    name: test-container
    command: ["/bin/sh", "-c", "tail -f /dev/null"]
    volumeMounts:
    # Directory location on container
    - mountPath: /test-hostpath-type-directory-on-container
      # Name of the volume on host
      name: test-hostpath-type-directory-on-host
  volumes:
  - name: test-hostpath-type-directory-on-host
    hostPath:
      # Directory location on host
      path: /var/tmp/test-hostpath-type-directory-on-host
      # This field is optional
      type: Directory
```

```
$ kubectl apply -f hostpath-type-directory.yaml
```

```
$ kubectl describe pod test-hostpath-type-directory
...
Events:
  Type     Reason       Age                  From               Message
  ----     ------       ----                 ----               -------
......
  Warning  FailedMount  97s (x9 over 3m44s)  kubelet            MountVolume.SetUp failed for volume "test-hostpath-type-directory-on-host" : hostPath type check failed: /test-hostpath-type-directory-on-host is not a directory
```

This instruction has failed because the directory `/test-hostpath-type-directory-in-host` is not existed.
Create it on each host in the kubernetes cluster.

```
$ mkdir /var/tmp/test-hostpath-type-directory-on-host
$ echo "This is a directory on ${HOST}" > /test-hostpath-type-directory-on-host/hostname.txt
$ cat /test-hostpath-type-directory-on-host/hostname.txt
This is a directory on <hostname>
```

Re-create the Pod.

```
$ kubectl delete pod test-hostpath-type-directory
$ kubectl apply -f hostpath-type-directory.yaml
```

```
$ kubectl exec -ti test-hostpath-type-directory sh
# ls -l /test-hostpath-type-directory-on-container/
-rw-r--r--    1 root     root            29 Dec 28 12:41 hostname.txt
# cat /test-hostpath-type-directory-on-container/hostname.txt
This is a directory on <hostname>  # <- This output shows that which host this pod is running on
```

## Type FileOrCreate configuration

* hostpath-type-file-or-create.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: test-hostpath-type-file-or-create
spec:
  containers:
  - image: k8s.gcr.io/busybox
    name: test-container
    command: ["/bin/sh", "-c", "tail -f /dev/null"]
    volumeMounts:
    # Directory location on container
    - mountPath: /test-hostpath-type-file-or-create
      name: mydir
    - mountPath: /test-hostpath-type-file-or-create/test.txt
      name: myfile
  volumes:
  - name: mydir
    hostPath:
      # Ensure the file directory is created
      path: /var/tmp/test-hostpath-type-file-or-create
      type: DirectoryOrCreate
  - name: myfile
    hostPath:
      # This instruction will be failed if you specify a file that is in the non-existence directory.
      # For example /var/tmp/test-hostpath-type-file-or-create2/test.txt
      path: /var/tmp/test-hostpath-type-file-or-create/test.txt
      type: FileOrCreate
```

```
$ kubectl apply -f hostpath-type-file-or-create.yaml
$ kubectl exec -ti test-hostpath-type-file-or-create -- ls -l /test-hostpath-type-file-or-create
total 0
-rw-r--r--    1 root     root             0 Dec 28 13:50 test.txt
```

The directory mounted from the container will be shown one host in the kubernetes cluster.

```
$ ls -l /var/tmp/test-hostpath-type-file-or-create
total 0
-rw-r--r-- 1 root root 0 Dec 28 13:50 test.txt
```

# Reference
https://github.com/kubernetes/examples
https://kubernetes.io/docs/concepts/storage/volumes/
