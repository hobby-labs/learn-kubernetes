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

## Opaque

You can create an empty Secret like below.
```
$ kubectl create secret generic empty-secret
$ kubectl get secret empty-secret
NAME          TYPE    DATA  AGE
empty-secret  Opaque  0     50s
```

You can also create a Secret from files like below.

```
$ 
```


https://kubernetes.io/docs/tasks/configmap-secret/managing-secret-using-kubectl/

## Reference
https://kubernetes.io/docs/concepts/configuration/secret/
