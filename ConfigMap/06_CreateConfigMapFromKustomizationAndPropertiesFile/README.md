# Create a ConfigMap from generator(kustomization.yaml) and properties file

* kustomization.yaml
```
configMapGenerator:
- name: game-config-4
  files:
  - configmap/game.properties
```

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
$ kubectl apply -k .
$ kubectl get configmap
NAME                       DATA   AGE
...
game-config-4-k87d9g8m72   1      7s
...
```

