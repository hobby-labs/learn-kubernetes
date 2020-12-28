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
  - image: k8s.gcr.io/test-webserver
    name: test-container
    volumeMounts:
    # Directory location on container
    - mountPath: /test-hostpath-type-directory-on-container
      # Name of the volume on host
      name: test-hostpath-type-directory-on-host
  volumes:
  - name: test-hostpath-type-directory-on-host
    hostPath:
      # Directory location on host
      path: /test-hostpath-type-directory-on-host
      # This field is optional
      type: Directory
```

Create the directory and file on each host.

```
$ mkdir /test-hostpath-type-directory-on-host
$ echo "This is a directory on ${HOST}" > /test-hostpath-type-directory-on-host/hostname.txt
```



# Reference
https://github.com/kubernetes/examples
