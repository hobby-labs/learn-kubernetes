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

## Reference
https://kubernetes.io/docs/concepts/configuration/secret/
