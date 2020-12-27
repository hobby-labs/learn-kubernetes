# Prepare config files

This is a directory that contains examples of ConfigMap.

### Other features
* Mounted ConfigMaps are updated automatically if its ConfigMap was updated
* kubelet monitors every 1 minutes (by default) to update it
* ConfigMap as a subPath volume will not receive ConfigMap updates.
* Pod will not start if keys that don't exists.
* Invalid keys will be skipped if you use `envFrom`. But the invalid names will be recorded in the event log `InvalidVariableNames`.

# Reference
https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/

