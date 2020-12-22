# Create ConfigMap from specific files

* configmap/foo.properties
```
foo.level=1
foo.file=/etc/foo.conf
```

* configmap/bar.properties
```
bar.level=2
bar.file=/etc/bar.conf
```

## Apply one config map
```
$ kubectl create configmap foo-config --from-file=configmap/foo.properties
$ kubectl describe configmaps foo-config
Name:         foo-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
foo.properties:
----
foo.level=1
foo.file=/etc/foo.conf

Events:  <none>

```

## Apply two config maps
```
$ kubectl create configmap foo-bar-config --from-file=configmap/foo.properties --from-file=configmap/bar.properties
$ kubectl describe configmaps foo-bar-config
Name:         foo-bar-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
bar.properties:
----
bar.level=2
bar.file=/etc/bar.conf

foo.properties:
----
foo.level=1
foo.file=/etc/foo.conf

Events:  <none>
```

# Reference
https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/
