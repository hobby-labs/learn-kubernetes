# Secrets

## A Secret can be used with a Pod in 3 ways

* As files in a volume mounted on one or more of its containers
* As container environment variable
* By the kubelet when pulling images for the Pod

## Restrictions

* The name of a Secret object must be a valid DNS subdomain name
* You can specify data and/or the stringData field
* The values for all keys in the `data` field have to be base64 encoded strings
* All key-value pairs in the `stringData` field are internally merged into the data field
* If a key appears in both the `data` and the `stringData` field, the value specified in the `strintDatal` field takes precedence

## Types of Secret
| Buildin Type | Usage |
| ---- | ---- |
| Opaque | arbitrary user-defined data |
| kubernetes.io/service-account-token | service account token |
| kubernetes.io/dockercfg | serialized `~/.dockercfg` file |
| kubernetes.io/dockerconfigjson | serialized `~/.docker/config.json` file |
| kubernetes.io/basic-auth | credentials for basic authentication |
| kubernetes.io/ssh-auth | credentials for SSH authentication |
| kubernetes.io/tls | data for a TLS client or server |
| bootstrap.kubernetes.io/token | bootstrap token data |

# Opaque

## Opaque (empty) Secret
You can create an empty Secret like below.
```
$ kubectl create secret generic empty-secret
$ kubectl get secret empty-secret
NAME          TYPE    DATA  AGE
empty-secret  Opaque  0     50s
```

## Create a Secret from file

You can also create a Secret from files like below.
You have to specify `-n` flag to `echo` because kubectl read content from file then encode them to base64.
This will contains new line character if existed.

```
$ mkdir secrets_from_file && cd secrets_from_file
$ echo -n 'admin' > ./username.txt
$ echo -n 'secret' > ./password.txt
$ kubectl create secret generic db-user-pass --from-file=./username.txt --from-file=./password.txt
secret/db-user-pass created
```

## Create a Secret from file with specifying key name
```
$ kubectl create secret generic db-user-pass --from-file=username=./username.txt --from-file=password=password.txt
```

## Create a Secret from literal
```
$ kubectl create secret generic dev-db-secret --from-literal=username=devuser --from-literal=password='secret'
```

## Verify the Secret
```
$ kubectl get secrets
......
$ kubectl describe secrets/db-user-pass
$ kubectl describe secrets/dev-db-secret
```

## Decoding the Secret
```
$ kubectl get secret db-user-pass -o jsonpath='{.data}'
{"password.txt":"c2VjcmV0","username.txt":"YWRtaW4="}
$ base64 -d -i <<< 'c2VjcmV0'
secret
$ base64 -d -i <<< 'YWRtaW4='
admin
```

## Crean up a Secret

```
$ kubectl delete secret db-user-pass
$ kubectl delete secret dev-db-secret
```

# Use the Default Service Account to access the API server

## Auto mounting

In version 1.6+ you can oput out of automounting API by setting `automountServiceAccountToken: false`

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: build-robot
automountServiceAccountToken: false
```

You can also mount API credentials for a particular podl

```
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  serviceAccountName: build-robot
  automountServiceAccountToken: false
  ...
```

The pod spec takes precedence over the service account if both specify a `automountServiceAccountToken` value.

## Use Multiple Service Accounts




# Reference
https://kubernetes.io/docs/concepts/configuration/secret/
https://kubernetes.io/docs/tasks/configmap-secret/managing-secret-using-kubectl/
