# emptyDir

emptyDir is a directory that exists as long as that Pod is running on that node.
As the name says, the `emptyDir` volumes is initially empty.

* pod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: test-emptydir-pod
spec:
  containers:
  - image: k8s.gcr.io/busybox
    name: test-emptydir-container
    command: ["/bin/sh", "-c", "tail -f /dev/null"]
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir: {}
```

## Restart a Pod

Check a node that the Pod is running and restart the Pod.

```
# kubectl describe pod test-emptydir-pod | grep -E '^Node: .*'
Node:         nuc03/192.168.1.23
# kubectl exec -ti test-emptydir-pod -- /bin/sh
pod ~# echo "Test message" > /cache/test.txt
pod ~# cat /cache/test.txt
Test message
pod ~# killall tail
command terminated with exit code 137
```

Re-check a node.

```
# kubectl describe pod test-emptydir-pod | grep -E '^Node: .*'
Node:         nuc03/192.168.1.23
```

If the Pod was restarted for some error, node that the Pod was running will not be changed.
According to this behavior, a file that located in emptyDir will be remaining.

```
# kubectl exec -ti test-emptydir-pod -- /bin/sh
# cat /cache/test.txt
Test message
```

## Update the image

* pod-image-update.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: test-emptydir-pod
spec:
  containers:
  - image: ubuntu
    name: test-emptydir-container
    command: ["/bin/sh", "-c", "tail -f /dev/null"]
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir: {}
```

`.spec.containers[0].image` was changed to `ubuntu` and re-apply.

```
# kubectl apply -f pod-image-update.yaml
```

Re-check the condition then you can find the pod is running on the same host.
And you can find the file that created before on the pod.

```
# kubectl describe pod test-emptydir-pod | grep -E '^Node: .*'
Node:         nuc03/192.168.1.23

# kubectl exec -ti test-emptydir-pod -- /bin/sh
# cat /cache/test.txt
Test message
```

## Use emptyDir on deployment
Deploy 5 pod on the k8s cluster that has 3 node on it.

```

```

```
# kubectl get pod | grep test-emptydir-deployment | awk '{print $1}' | xargs -I {} bash -c "echo -n '{}    '; kubectl describe pod {} | grep -E '^Node:' | awk '{print \$2}'"
test-emptydir-deployment-cdc6bd54-2fpfj    nuc03/192.168.1.23
test-emptydir-deployment-cdc6bd54-7fc2m    nuc02/192.168.1.22
test-emptydir-deployment-cdc6bd54-88zl8    nuc02/192.168.1.22
test-emptydir-deployment-cdc6bd54-tws2m    nuc03/192.168.1.23
test-emptydir-deployment-cdc6bd54-zsvrf    nuc02/192.168.1.22

# while read line; do
     kubectl exec $line -- /bin/bash -c 'echo $(hostname) > /cache/$(hostname).txt'
 done < <(kubectl get pod | grep test-emptydir-deployment | awk '{print $1}')

# while read line; do
     kubectl exec $line -- /bin/bash -c 'echo ">>> $(hostname)"; ls -l /cache; cat /cache/*.txt'
 done< <(kubectl get pod | grep test-emptydir-deployment | awk '{print $1}')

# while read line; do
     kubectl exec $line -- /bin/bash -c 'echo ">>> $(hostname)"; ls -l /cache; cat /cache/*.txt'
 done< <(kubectl get pod | grep test-emptydir-deployment | awk '{print $1}')
>>> test-emptydir-deployment-cdc6bd54-2fpfj
total 4
-rw-r--r-- 1 root root 40 Jan  2 13:34 test-emptydir-deployment-cdc6bd54-2fpfj.txt
test-emptydir-deployment-cdc6bd54-2fpfj
>>> test-emptydir-deployment-cdc6bd54-7fc2m
total 4
-rw-r--r-- 1 root root 40 Jan  2 13:34 test-emptydir-deployment-cdc6bd54-7fc2m.txt
test-emptydir-deployment-cdc6bd54-7fc2m
>>> test-emptydir-deployment-cdc6bd54-88zl8
total 4
-rw-r--r-- 1 root root 40 Jan  2 13:34 test-emptydir-deployment-cdc6bd54-88zl8.txt
test-emptydir-deployment-cdc6bd54-88zl8
>>> test-emptydir-deployment-cdc6bd54-tws2m
total 4
-rw-r--r-- 1 root root 40 Jan  2 13:34 test-emptydir-deployment-cdc6bd54-tws2m.txt
test-emptydir-deployment-cdc6bd54-tws2m
>>> test-emptydir-deployment-cdc6bd54-zsvrf
total 4
-rw-r--r-- 1 root root 40 Jan  2 13:34 test-emptydir-deployment-cdc6bd54-zsvrf.txt
test-emptydir-deployment-cdc6bd54-zsvrf

# # // Each emptyDir are unique on each Pod even if the Pods are running on the same Node.
```

The Pod will be restarted on the same Pod if it was down for some error.

```
# kubectl exec -ti test-emptydir-deployment-cdc6bd54-7fc2m -- pkill tail
# kubectl describe pod test-emptydir-deployment-cdc6bd54-7fc2m | grep Node:
Node:         nuc02/192.168.1.22    # <- Same node before the Pod was down with some error
# kubectl exec -ti test-emptydir-deployment-cdc6bd54-7fc2m -- /bin/bash -c 'echo ">>> $(hostname)"; ls -l /cache; cat /cache/*.txt'
```

## tmpDir is shared during the containers in a Pod

https://www.alibabacloud.com/blog/kubernetes-volume-basics-emptydir-and-persistentvolume_594834

# Reference
https://kubernetes.io/docs/concepts/storage/volumes/

