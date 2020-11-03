# Prepare config files

## Create configmap from directories
Create property files.

```
$ mkdir someconf
$ cat << EOF > conf/bar.properties
bar.level=2
bar.file=/etc/bar.conf
EOF
$ cat << EOF > someconf/foo.properties
foo.level=1
foo.file=/etc/foo.conf
EOF
```

Create config map.

```
$ kubectl create configmap some-config --from-file=someconf
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
$ kubectl create configmap foo-config --from-file=someconf/foo.properties
$ kubectl describe configmaps foo-config
```

You can also create a config map from multiple file by specifying them with using --from-file option multi times.

```
$ kubectl create configmap foo-bar-config --from-file=someconf/foo.properties --from-file=someconf/bar.properties
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
$ kubectl create configmap foo-env --from-env-file=someconf/foo.properties
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
$ kubectl create configmap some-env --from-env-file=someconf/foo.properties --from-env-file=someconf/foo.properties
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
$ kubectl create configmap foo-key-config --from-file=foo-key.properties=someconf/foo.properties
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
$ kubectl create configmap foo-bar-key-config --from-file=foo-key.properties=someconf/foo.properties --from-file=bar-key.properties=someconf/bar.properties
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
  - foo-generator-key=someconf/foo.properties
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

* conf/multiple_configmaps.yaml
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: special-foo-level-config
  namespace: default
data:
  foo.level: "12"
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: special-foo-file-config
  namespace: default
data:
  foo.file: "/etc/special-foo-file-config"
```

Then create a ConfigMap from it.

```
$ kubectl create -f conf/multiple_configmaps.yaml
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
$ kubectl logs special-foo-multiple-configmaps-pod
...
SPECIAL_FOO_FILE=/etc/special-foo-file-config
SPECIAL_FOO_LEVEL=12
...
```

## Configure all key-value pairs in a ConfigMap as container environment variables

* conf/configmap-all-key-value-pairs.yaml
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: all-key-value-config
  namespace: default
data:
  SPECIAL_FOO_LEVEL: "13"
  SPECIAL_FOO_FILE: "/etc/special_all_key_value_config.conf"
```

```
$ kubectl create -f conf/configmap-all-key-value-pairs.yaml
```

Use `envFrom` to define all of the ConfigMap's data as container environment variables.

* pod-all-key-value-pairs.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: all-key-value-pairs-pod
spec:
  containers:
    - name: all-key-value-pairs-container
      image: k8s.gcr.io/busybox
      command: ["/bin/sh", "-c", "env"]
      envFrom:
      - configMapRef:
          name: all-key-value-config
  restartPolicy: Never
```

```
$ kubectl create -f pod-all-key-value-pairs.yaml
$ kubectl logs all-key-value-pairs-pod
...
SPECIAL_FOO_FILE=/etc/special_all_key_value_config.conf
SPECIAL_FOO_LEVEL=13
...
```

## Use ConfigMap-defined environment variables in Pod commands

We can use ConfigMap-defined environment variable `$(VAR_NAME)` in `command` section.

* configmap-defined-env-var-pod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: configmap-defined-env-var-pod
spec:
  containers:
    - name: configmap-defined-env-var-container
      image: k8s.gcr.io/busybox
      command: ["/bin/sh", "-c", "echo SPECIAL_FOO_LEVEL_KEY=$(SPECIAL_FOO_LEVEL_KEY) SPECIAL_FOO_FILE_KEY=$(SPECIAL_FOO_FILE_KEY)"]
      env:
        - name: SPECIAL_FOO_LEVEL_KEY
          valueFrom:
            configMapKeyRef:
              name: all-key-value-config
              key: SPECIAL_FOO_LEVEL
        - name: SPECIAL_FOO_FILE_KEY
          valueFrom:
            configMapKeyRef:
              name: all-key-value-config
              key: SPECIAL_FOO_FILE
  restartPolicy: Never
```

```
$ kubectl create -f configmap-defined-env-var-pod.yaml
$ kubectl logs configmap-defined-env-var-pod
```

## Populate a Volume with data stored in a ConfigMap

* populate-volume-with-data-stored-in-a-configmap.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: populate-volume-with-data-stored-in-a-configmap-pod
spec:
  containers:
    - name: populate-volume-with-data-stored-in-a-configmap-container
      image: k8s.gcr.io/busybox
      command: ["/bin/sh", "-c", "ls /etc/config/"]
      volumeMounts:
      - name: config-volume
        mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: all-key-value-config
  restartPolicy: Never
```

```
$ kubectl create -f populate-volume-with-data-stored-in-a-configmap.yaml
$ kubectl logs populate-volume-with-data-stored-in-a-configmap-pod
SPECIAL_FOO_LEVEL
SPECIAL_FOO_FILE
```

If a directory `/etc/config` is already exists on the container, it will be removed.
Change the command in `populate-volume-with-data-stored-in-a-configmap.yaml` then create and see logs, you can see the value of the values.

## Add ConfigMap data to a specific path in the Volume

* add-configmap-data-to-a-specific-path.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: add-configmap-data-to-a-specific-path-pod
spec:
  containers:
    - name: add-configmap-data-to-a-specific-path-container
      image: k8s.gcr.io/busybox
      command: ["/bin/sh", "-c", "cat /etc/config/keys"]
      volumeMounts:
      - name: config-volume
        mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: all-key-value-config
        items:
          - key: SPECIAL_FOO_LEVEL
            path: keys
  restartPolicy: Never
```

```
$ kubectl create -f add-configmap-data-to-a-specific-path.yaml
$ kubectl logs add-configmap-data-to-a-specific-path-pod
```

You CAN NOT declare multi items as a same key at one time like below.
It will only apply `SPECIAL_FOO_FILE`.

```
...
  volumes:
    - name: config-volume
      configMap:
        items:
          - key: SPECIAL_FOO_LEVEL
            path: keys
          - key: SPECIAL_FOO_FILE
            path: keys
...
```

### Other features
* Mounted ConfigMaps are updated automatically if its ConfigMap was updated
* kubelet monitors every 1 minutes (by default) to update it
* ConfigMap as a subPath volume will not receive ConfigMap updates.
* Pod will not start if keys that don't exists.
* Invalid keys will be skipped if you use `envFrom`. But the invalid names will be recorded in the event log `InvalidVariableNames`.

# Reference
https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/

