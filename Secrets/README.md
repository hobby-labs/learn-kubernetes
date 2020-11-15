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
  name: service-account-test
spec:
  containers:
  - image: k8s.gcr.io/busybox
    name: service-account-test
    command: ["tail", "-f", "/dev/null"]
  automountServiceAccountToken: false
```

The pod spec takes precedence over the service account if both specify a `automountServiceAccountToken` value.

```
$ kubectl exec -ti service-account-test -- ls -l /var/run/secrets
ls: /var/run/secrets: No such file or directory
command terminated with exit code 1
```

If you did not specify `automountServiceAccountToken: false`, you can see the directory mounted automatically.

```
$ kubectl exec -ti service-account-test -- ls -l /var/run/secrets/kubernetes.io/serviceaccount
lrwxrwxrwx    1 root     root            13 Nov 15 04:23 ca.crt -> ..data/ca.crt
lrwxrwxrwx    1 root     root            16 Nov 15 04:23 namespace -> ..data/namespace
lrwxrwxrwx    1 root     root            12 Nov 15 04:23 token -> ..data/token
```

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

## Editing a Secret

```
$ kubectl edit secrets mysecret

apiVersion: v1
data:
  PASSWORD: MWYyZDFlMmU2N2Rm
  USER_NAME: YWRtaW4=
kind: Secret
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"PASSWORD":"MWYyZDFlMmU2N2Rm","USER_NAME":"YWRtaW4="},"kind":"Secret","metadata":{"annotations":{},"name":"mysecret","namespace":"default"},"type":"Opaque"}
  creationTimestamp: "2020-10-06T14:52:20Z"
  name: mysecret
  namespace: default
  resourceVersion: "921708"
  selfLink: /api/v1/namespaces/default/secrets/mysecret
  uid: 0b174116-ec61-440a-a4c6-b2fc0f0188c8
type: Opaque
```

You can see base64 encoded Secret values in the `data` field.

## Using Secrets

Secrets can be mounted as data volumes or exposed as environment variables to be used by a container.

```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: mysecret
```

```
$ kubectl exec -ti mypod -- ls -l /etc/foo
total 0
lrwxrwxrwx 1 root root 15 Nov 12 15:56 PASSWORD -> ..data/PASSWORD
lrwxrwxrwx 1 root root 16 Nov 12 15:56 USER_NAME -> ..data/USER_NAME
```

If there are multiple containers in the Pod, then each container needs its own `volumeMounts` block, but only one `.spec.volumes` is needed per Secret.

## Projection of Secret keys to specific paths

```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: mysecret
      items:
      - key: USER_NAME
        path: my-group/my-username
```

```
$ kubectl exec -ti mypod -- ls -l /etc/foo/my-group/my-username
-rw-r--r-- 1 root root 5 Nov 14 01:29 /etc/foo/my-group/my-username
```

* `username` secret is stored under `/etc/foo/my-group/my-username` field instead of `/etc/foo/username`
* `password` secret is not projected

If `.spec.volumes[].secret.items` is used, only keys specified in `items` are projected.

## Secret files permissions
If you don't specify any permissions, `0644` is used by default.
You can also set a default mode for the entire Secret volume and override per key if needed.

```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
  volumes:
  - name: foo
    secret:
      secretName: mysecret
      defaultMode: 0400
```

```
$ kubectl exec -ti mypod -- bash -c "cd /etc/foo && ls -l \$(readlink USER_NAME)"
-r-------- 1 root root 5 Nov 14 01:37 ..data/USER_NAME
```

You can also use mapping, sa in the previous example, and specify different permissions for different files like this.

```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
  volumes:
  - name: foo
    secret:
      secretName: mysecret
      items:
      - key: USER_NAME
        path: my-group/my-username
        mode: 0777
```

```
$ kubectl exec -ti mypod -- bash -c "cd /etc/foo && ls -l \$(readlink USER_NAME)"
lrwxrwxrwx 1 root root 15 Nov 14 01:43 my-group -> ..data/my-group
```

# Using Secrets as environment variables

https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-environment-variables

## To use a secret in an environment variable

```
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
  - name: mycontainer
    image: redis
    env:
      - name: SECRET_USERNAME
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: USER_NAME
      - name: SECRET_PASSWORD
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: PASSWORD
  restartPolicy: Never
```

```
$ kubectl exec -ti secret-env-pod -- bash -c "env | grep -P '^SECRET_'"
SECRET_USERNAME=admin
SECRET_PASSWORD=1f2d1e2e67df
```

## Immutable Secrets

Immutable Secrets are

* Protects you from accidental updates that could cause applications outages
* Improves performance of your cluster by significantly reducing load on kube-apiserver, closing watches for secrets marked as immutable

```
apiVersion: v1
kind: Secret
metadata:
  ...
data:
  ...
immutable: true
```

## Using imagePullSecrets
You can use an `imagePullSecrets` to pass a secret that contains a Docker (or other) image registry password to the kubelet.

## Manually specifying an imagePullSecret

https://kubernetes.io/docs/concepts/containers/images/#specifying-imagepullsecrets-on-a-pod

## Arranging for imagePullSecrets to be automatically attached
Any Pods created with that ServiceAccount or created with that ServiceAccount by default, will get their `imagePullSecrets` field set to that of the service account.

https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#add-imagepullsecrets-to-a-service-account

## Automatic mounting of manually created Secrets

Manyally created secrets (for example, one containing a token for accessing a GitHub account) can be automatically attached to pods based on their service account.

https://kubernetes.io/docs/tasks/inject-data-application/podpreset/

## Pod with ssh keys

```
$ ssh-keygen -f k8skey
$ kubectl create secret generic ssh-key-secret --from-file=ssh-privatekey=${PWD}/k8skey --from-file=ssh-publickey=${PWD}/k8skey.pub
```

```
apiVersion: v1
kind: Pod
metadata:
  name: secret-test-pod
  labels:
    name: secret-test
spec:
  volumes:
  - name: secret-volume
    secret:
      secretName: ssh-key-secret
  containers:
  - name: ssh-test-container
    image: k8s.gcr.io/busybox
    command: ["ls", "-l", "/etc/secret-volume"]
    volumeMounts:
    - name: secret-volume
      readOnly: true
      mountPath: "/etc/secret-volume"
  restartPolicy: Never
```

```
$ kubectl logs secret-test-pod
total 0
lrwxrwxrwx    1 root     root            21 Nov 14 03:34 ssh-privatekey -> ..data/ssh-privatekey
lrwxrwxrwx    1 root     root            20 Nov 14 03:34 ssh-publickey -> ..data/ssh-publickey
```

## Pods with prod / test credentials

```
$ kubectl create secret generic prod-db-secret --from-literal=username=produser --from-literal=password=prod-secret
$ kubectl create secret generic test-db-secret --from-literal=username=testuser --from-literal=password=test-secret
```

```
apiVersion: v1
kind: List
items:
- kind: Pod
  apiVersion: v1
  metadata:
    name: prod-db-client-pod
    labels:
      name: prod-db-client
  spec:
    volumes:
    - name: secret-volume
      secret:
        secretName: prod-db-secret
    containers:
    - name: db-client-container
      image: k8s.gcr.io/busybox
      command: ["tail", "-f", "/dev/null"]
      volumeMounts:
      - name: secret-volume
        readOnly: true
        mountPath: "/etc/secret-volume"
- kind: Pod
  apiVersion: v1
  metadata:
    name: test-db-client-pod
    labels:
      name: test-db-client-pod
  spec:
    volumes:
    - name: secret-volume
      secret:
        secretName: test-db-secret
    containers:
    - name: db-client-container
      image: k8s.gcr.io/busybox
      command: ["tail", "-f", "/dev/null"]
      volumeMounts:
      - name: secret-volume
        readOnly: true
        mountPath: "/etc/secret-volume"

```

```
$ kubectl exec -ti prod-db-client-pod -- sh -c "cd /etc/secret-volume; ls -l; cat username; echo; cat password"
$ kubectl exec -ti test-db-client-pod -- sh -c "cd /etc/secret-volume; ls -l; cat username; echo; cat password"
```

You could further simplify the base Pod specification by using two service accounts

```
apiVersion: v1
kind: Pod
metadata:
  name: prod-db-client-pod
  labels:
    name: prod-db-client
spec:
  serviceAccount: prod-db-client
  containers:
  - name: db-client-container
    image: k8s.gcr.io/busybox
```



# Reference

* Secrets
https://kubernetes.io/docs/concepts/configuration/secret/
* Managing Secret using kubectl
https://kubernetes.io/docs/tasks/configmap-secret/managing-secret-using-kubectl/
