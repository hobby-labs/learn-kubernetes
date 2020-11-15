# Encrypting Secret Data at Rest

https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/

## Encrypting your data

encryption.yaml
```
kind: EncryptionConfiguration
resources:
  - resources:
    - secrets
    providers:
    - aescbc:
        keys:
        - name: key1
          secret: r4rGb8CJxIy5zh80kKDWxA6wQOPR7zbBlEmLVSsohIo=
    - identity: {}
```

Put `encryption.yaml` to `/etc/kubernetes/pki/encryption.yaml` that is contained `/etc/kubernetes/manifests`.

```
$ mv encryption.yaml /etc/kubernetes/pki/encryption.yaml
```

Add a line like below in `/etc/kubernetes/manifests/kube-apiserver.yaml`.

```
  - --encryption-provider-config=/etc/kubernetes/encryption.yaml
```

Wait to restart kubelet.
Then you can check the status of the Pods.

```
$ kubectl get pods
-> error
```

TODO: 

# Reference

https://stackoverflow.com/questions/59817491/how-to-add-encryption-provider-config-option-to-kube-apiserver

https://stackoverflow.com/questions/59817491/how-to-add-encryption-provider-config-option-to-kube-apiserver

