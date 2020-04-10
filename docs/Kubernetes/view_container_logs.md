# View Logs in Live Environment

1. Execute command `kubectl get pods -n jx-live`
2. Search for the pods that you want to view the logs.
3. Execute command `kubectl logs [pod name] --namespace jx-live --container [container name]` \
   E.g.

```sh
kubectl logs jx-company-employerengagementapi-554b5dc4df-wxll7 --namespace jx-live --container company-employerengagementapi
# or
kubectl logs jx-company-employerengagementapi-554b5dc4df-wxll7 -n jx-live -c company-employerengagementapi
```
