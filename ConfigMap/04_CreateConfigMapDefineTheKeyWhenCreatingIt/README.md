# Create configmap define the key to use when creating a ConfigMap from a file

Syntax

```
$ kubectl create configmap game-config-3 --from-file=${my_key_name}=${path_to_file}
```

Prepare a properties file.

* configmap/game.properties
```
enemies=aliens
lives=3
enemies.cheat=true
enemies.cheat.level=noGoodRotten
secret.code.passphrase=UUDDLRLRBABAS
secret.code.allowed=true
secret.code.lives=30
```

```
$ kubectl create configmap game-config-3 --from-file=game-special-key=configmap/game.properties
```

# Reference
https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/

