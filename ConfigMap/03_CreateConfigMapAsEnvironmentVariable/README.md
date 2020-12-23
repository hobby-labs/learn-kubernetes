# Create configmap as environment variable from env-file

* configmap/game-env-file.properties
```
enemies=aliens
lives=3
allowed="true"
```

```
$ kubectl create configmap game-config-env-file --from-env-file=configmap/game-env-file.properties
$ kubectl get configmap game-config-env-file -o yaml
apiVersion: v1
data:
  allowed: '"true"'
  enemies: aliens
  lives: "3"
kind: ConfigMap
metadata:
  creationTimestamp: "2020-12-23T15:59:58Z"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:data:
        .: {}
        f:allowed: {}
        f:enemies: {}
        f:lives: {}
    manager: kubectl-create
    operation: Update
    time: "2020-12-23T15:59:58Z"
  name: game-config-env-file
  namespace: default
  resourceVersion: "11687094"
  selfLink: /api/v1/namespaces/default/configmaps/game-config-env-file
  uid: 7d0221f5-721a-4503-bd5d-d26a379ba515
```

You can also specify multi env file.

* configmap/ui-env-file.properties
```
width=200
height=180
```

```
$ kubectl create configmap config-multi-env-files \
    --from-env-file=configmap/game-env-file.properties \
    --from-env-file=configmap/ui-env-file.properties

$ kubectl get configmap config-multi-env-files -o yaml
apiVersion: v1
data:
  height: "180"
  width: "200"
kind: ConfigMap
metadata:
  creationTimestamp: "2020-12-23T16:04:00Z"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:data:
        .: {}
        f:height: {}
        f:width: {}
    manager: kubectl-create
    operation: Update
    time: "2020-12-23T16:04:00Z"
  name: config-multi-env-files
  namespace: default
  resourceVersion: "11687682"
  selfLink: /api/v1/namespaces/default/configmaps/config-multi-env-files
  uid: e48d3866-deda-4649-a555-4f0ae457f2a0
```


