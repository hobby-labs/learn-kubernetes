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

https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/

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

Every namespace has a default service account resource called `default`.

```
$ kubectl get serviceaccounts
NAME      SECRETS   AGE
default   1         42d
```

You can create additional Serviceaccount objects.
The name of a ServiceAccount

```
$ kubectl apply -f - << EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: build-robot
EOF
```

```
$ kubectl get serviceaccounts/build-robot -o yaml
  ......
  name: build-robot
  namespace: default
  resourceVersion: "2374275"
  selfLink: /api/v1/namespaces/default/serviceaccounts/build-robot
  uid: e87c6902-e2e2-4730-a94f-b2e9503343ce
secrets:
- name: build-robot-token-h8mp8
```

You can see the token(secrets) that has automatically been created.

To use a non-default service accunt on the Pod, you can specify it with `spec.serviceAccountName` filed in the manifest of the Pod.

## Delete ServiceAccount

```
$ kubectl delete serviceaccount/build-robot
```

## Manually create a service account API token

You can create secret and specify its name.

```
$ kubectl apply -f - << EOF
apiVersion: v1
kind: Secret
metadata:
  name: build-robot-secret
  annotations:
    kubernetes.io/service-account.name: build-robot
type: kubernetes.io/service-account-token
EOF
```

This command create secret on the service-account `build-robot`.

```
$ kubectl describe secrets/build-robot-secret
Name:         build-robot-secret
Namespace:    default
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: build-robot
              kubernetes.io/service-account.uid: 25a38222-3f8d-415c-9429-ffccd8c1ea35

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1066 bytes
namespace:  7 bytes
token:      eyJhbGc......
```

# Add ImagePullSecrets to a service account

https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#add-imagepullsecrets-to-a-service-account

## Create an imagePullSecret

```
$ kubectl create secret docker-registry myregistrykey --docker-server=DUMMY_SERVER \
    --docker-username=dummy_username --docker-password=dummy_docker_secret --docker-email=dummy_docker_email
secret/myregistrykey created
```

```
$ kubectl get secrets myregistrykey
NAME            TYPE                             DATA   AGE
myregistrykey   kubernetes.io/dockerconfigjson   1      15m

```

## Add image pull secret to service account

```
$ kubectl patch serviceaccount default -p '{"imagePullSecrets": [{"name": "myregistrykey"}]}'
```

## Add image pull secret to service account with `kubectl edit` or yaml

You can add image pull secret to service account by `kubectl edit` or yaml instead.
If you will add it from yaml, you can do like below.

```
$ kubectl get serviceaccounts default -o yaml > ./sa.yaml
$ vim sa.yaml
```

```
apiVersion: v1
imagePullSecrets:
- name: myregistrykey
kind: ServiceAccount
metadata:
  creationTimestamp: "2020-09-26T11:09:08Z"
  name: default
  namespace: default
  resourceVersion: "2572666"
  selfLink: /api/v1/namespaces/default/serviceaccounts/default
  uid: 796bdaa7-fbfc-4fc4-b02b-70a201f4336e
secrets:
- name: default-token-5p6x2
```

delete line with key `resourceVersion`, add lines with `imagePullSecrets:` and save

```
 apiVersion: v1
 imagePullSecrets:
 - name: myregistrykey
 kind: ServiceAccount
 metadata:
   creationTimestamp: "2020-09-26T11:09:08Z"
   name: default
   namespace: default
-  resourceVersion: "2572666"
   selfLink: /api/v1/namespaces/default/serviceaccounts/default
   uid: 796bdaa7-fbfc-4fc4-b02b-70a201f4336e
 secrets:
 - name: default-token-5p6x2
+imagePullSecrets:
+- name: myregistrykey
```

```
$ kubectl replace serviceaccount default -f ./sa.yaml
serviceaccount/default replaced
```

## Verify image Pull
Verify imagePullSecrets was added to pod spec.

```
$ kubectl run nginx --image=nginx --restart=Never
$ kubectl get pod nginx -o=jsonpath='{.spec.imagePullSecrets[0].name}{"\n"}'
myregistrykey
```

## About imagePullSecrets
https://kubernetes.io/docs/concepts/containers/images/#specifying-imagepullsecrets-on-a-pod

## Service Account Token Volume Projection(Kubernetes v1.12 [beta])
The kubelet can also project a service account token into a Pod.

* pod-projected-svc-token.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    volumeMounts:
    - mountPath: /var/run/secrets/tokens
      name: vault-token
  serviceAccountName: build-robot
  volumes:
  - name: vault-token
    projected:
      sources:
      - serviceAccountToken:
          path: vault-token
          expirationSeconds: 7200
          audience: vault
```

```
$ kubectl create -f ./pod-projected-svc-token.yaml
```

## Service Account Issuer Discovery(Kubernetes v1.18 [alpha])
The Service Account Issuer Discovery feature is enabled by enabling the `ServiceAccountIssuerDiscovery`.

See  
https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#service-account-issuer-discovery

# Creating a Secret

https://kubernetes.io/docs/concepts/configuration/secret/#creating-a-secret

## 

# Reference
* Secrets
https://kubernetes.io/docs/concepts/configuration/secret/
* Managing Secret using kubectl
https://kubernetes.io/docs/tasks/configmap-secret/managing-secret-using-kubectl/