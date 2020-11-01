# Prepare config files

## Create configmap from directories
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

## Create configmap from files
You can create a config map from one file.

```
$ kubectl create configmap foo-config --from-file=conf/foo.properties
$ kubectl describe configmaps foo-config

```

You can also create a config map from multiple file by specifying them with using --from-file option multi times.

```
$ kubectl create configmap foo-bar-config --from-file=conf/foo.properties --from-file=conf/bar.properties
$ kubectl describe configmaps foo-bar-config
```

## Create configmap as environment variable
Create configmap as environment variable from foo.properties.

```
$ cat conf/foo.properties
foo.level=1
foo.file=/etc/foo.conf
```

Apply config with the option `--from-env-file`.

```
$ kubectl create configmap foo-env --from-env-file=conf/foo.properties
$ kubectl get configmap foo-env -o yaml
apiVersion: v1
data:
  foo.file: /etc/foo.conf
  foo.level: "1"
kind: ConfigMap
metadata:
  creationTimestamp: "2020-11-01T03:02:19Z"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:data:
        .: {}
        f:foo.file: {}
        f:foo.level: {}
    manager: kubectl-create
    operation: Update
    time: "2020-11-01T03:02:19Z"
  name: some-env
  namespace: default
  resourceVersion: "1232974"
  selfLink: /api/v1/namespaces/default/configmaps/some-env
  uid: 88769c22-16e3-4efe-b2d2-06756055cced
```

Difference configmap and env are like below.

```
# config map
...
data:
  foo.properties: |
    foo.level=1
    foo.file=/etc/foo.conf
...
# env
data:
  foo.file: /etc/foo.conf
  foo.level: "1"
```

You can specify multiple files but not specify directory.

```
$ kubectl create configmap some-env --from-env-file=conf/foo.properties --from-env-file=conf/foo.properties
```

This variable will be loaded as environment variables on the Pod.

## Define the key to use when creating a ConfigMap from a file
Create a config map with specifying key name.

```
$ cat conf/foo.properties
foo.level=1
foo.file=/etc/foo.conf
```

```
$ kubectl create configmap foo-key-config --from-file=foo-key.properties=conf/foo.properties
$ kybectl get configmap foo-key-config -o yaml
apiVersion: v1
data:
  foo-key.properties: |
    foo.level=1
    foo.file=/etc/foo.conf
kind: ConfigMap
metadata:
  creationTimestamp: "2020-11-01T03:18:08Z"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:data:
        .: {}
        f:foo-key.properties: {}
    manager: kubectl-create
    operation: Update
    time: "2020-11-01T03:18:08Z"
  name: foo-key-config
  namespace: default
  resourceVersion: "1235268"
  selfLink: /api/v1/namespaces/default/configmaps/foo-key-config
  uid: 6ad55ca5-573a-4e4a-a947-21516c5ac9c0
```

You can also specify multi files like below but not directory.

```
$ kubectl create configmap foo-bar-key-config --from-file=foo-key.properties=conf/foo.properties --from-file=bar-key.properties=conf/bar.properties
```

## Create ConfigMaps from literal values

```
$ kubectl create configmap foo-literal-config --from-literal=foo.level=10 --from-literal=foo.file=/etc/foo-literal.conf
$ kubectl get configmap foo-literal-config -o yaml
apiVersion: v1
data:
  foo.file: /etc/foo-literal.conf
  foo.level: "10"
kind: ConfigMap
......
```

# Create ConfigMap from generator
Create ConfigMap from kustomization.yaml

```
cat << EOF > ./kustomization.yaml
configMapGenerator:
- name: foo-generator-config
  files:
  - conf/foo.properties
EOF
```

Create ConfigMap from `kustomization.yaml`.
You can specify a directory not a file with option `-k`.

```
$ kubectl apply -k .
$ kubectl get configmap
NAME                              DATA   AGE
...
foo-generator-config-5hbd74hfk8   1      2m27s
...
$ kubectl get configmaps/foo-generator-config-5hbd74hfk8 -o yaml
apiVersion: v1
data:
  foo.properties: |
    foo.level=1
    foo.file=/etc/foo.conf
kind: ConfigMap
...
```

Suffix of the config name is generated with its content.
The suffix will be changed when you create new ConfigMap with different content.

## Create ConfigMap with defined key from generator

```
$ cat << EOF > ./kustomization.yaml
configMapGenerator:
- name: foo-generator-key-config
  files:
  - foo-generator-key=conf/foo.properties
EOF
```

```
$ kubectl apply -k .
$ kubectl get configmap
NAME                                  DATA   AGE
...
foo-generator-key-config-cmcb78kk7k   1      2m25s
...
$ kubectl get configmap/foo-generator-key-config-cmcb78kk7k -o yaml
apiVersion: v1
data:
  foo-generator-key: |
    foo.level=1
    foo.file=/etc/foo.conf
kind: ConfigMap
...
```

## Create ConfigMap with literals from generator

```
$ cat << EOF > ./kustomization.yaml
configMapGenerator:
- name: foo-generator-literal-config
  literals:
  - foo.level=10
  - foo.file=/etc/foo-generator-literal-config.conf
EOF
```

```
$ kubectl apply -k .
$ kubectl get configmap
NAME                                      DATA   AGE
...
foo-generator-literal-config-m8c5949672   2      33m
...
```

# Define container environment variables using ConfigMap data
## Define a container environment variable with data from a single ConfigMap

```
$ kubectl create configmap foo-special-config --from-literal=foo.level=11 --from-literal=foo.file=/etc/foo-special-config.conf
```

If you want to assign `foo.level` to the value `SPECIAL_FOO_LEVEL`, create manifest like below.

* pod-single-configmap-env-variable.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: foo-special-config-pod
spec:
  containers:
    - name: test-container
      image: k8s.gcr.io/busybox
      command: ["/bin/sh", "-c", "env"]
      env:
        - name: SPECIAL_FOO_LEVEL
          valueFrom:
            configMapKeyRef:
              name: foo-special-config
              key: foo.level
        - name: SPECIAL_FOO_FILE
          valueFrom:
            configMapKeyRef:
              name: foo-special-config
              key: foo.file
  restartPolicy: Never
```

Create the Pod then you can see the values like below.

```
$ kubectl create -f ./pod-single-configmap-env-variable.yaml
$ # After few seconds...
$ kubectl logs foo-special-config-pod
...
SPECIAL_FOO_FILE=/etc/foo-special-config.conf
SPECIAL_FOO_LEVEL=11
...
```

## Define container environment variables with data from multiple ConfigMaps

* multiple_configmaps.yaml
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: special-foo-level-config
  namespace: default
data:
  foo.level: 12
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: special-foo-file-config
  namespace: default
data:
  foo.file: /etc/special-foo-file-config
```

Then create a ConfigMap from it.

```
$ kubectl create -f multiple_configmaps.yaml
```

Define environment variables.

* pod-multiple-configmap-env-variable.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: special-foo-multiple-configmaps-pod
spec:
  containers:
    - name: special-foo-multiple-configmaps-container
      image: k8s.gcr.io/busybox
      command: ["/bin/sh", "-c", "env"]
      env:
        - name: SPECIAL_FOO_LEVEL
          valueFrom:
            configMapKeyRef:
              name: special-foo-level-config
              key: foo.level
        - name: SPECIAL_FOO_FILE
          valueFrom:
            configMapKeyRef:
              name: special-foo-file-config
              key: foo.file
  restartPolicy: Never
```

Create the Pod.

```
$ kubectl create -f pod-multiple-configmap-env-variable.yaml
```

# Reference
https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/

