# Assign Namespace to Node

```
apiVersion: v1
kind: Namespace
metadata:
  name:  jx
  annotations:
    scheduler.alpha.kubernetes.io/node-selector: <<your node label>>
spec: {}
status: {}
```
