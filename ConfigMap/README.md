# Prepare config files

Create property files.

```
$ mkdir conf
$ cat << EOF > conf/bar.properties
bar.level=2
bar.file=/etc/bar.conf
EOF
$ cat << EOF > conf/foo.properties
foo.level=1
foo.file=/etc/foo.conf
EOF
```

Create config map.

```
$ kubectl create configmap some-config --from-file=conf
configmap/some-config created
```

Show config map.

```
$ kubectl describe configmaps some-config
Name:         some-config
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

Show config map as yaml format.

```
apiVersion: v1
data:
  bar.properties: |
    bar.level=2
    bar.file=/etc/bar.conf
  foo.properties: |
    foo.level=1
    foo.file=/etc/foo.conf
kind: ConfigMap
metadata:
  creationTimestamp: "2020-11-01T02:09:16Z"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:data:
        .: {}
        f:bar.properties: {}
        f:foo.properties: {}
    manager: kubectl-create
    operation: Update
    time: "2020-11-01T02:09:16Z"
  name: some-config
  namespace: default
  resourceVersion: "1225274"
  selfLink: /api/v1/namespaces/default/configmaps/some-config
  uid: 7fb30c93-a168-4601-b9e1-c8ad5893f98e
```

You can create a config map from one file.

```
$ kubectl create configmap foo-config --from-file=conf/foo.properties
$ kubectl describe configmaps foo-config

```

You can also create a config map from multiple file by specifying them with using --from-file option multi times.

```
$ kubectl create configmap foo-bar-config --from-file=conf/foo.properties --from-file=conf/bar.properties
```



