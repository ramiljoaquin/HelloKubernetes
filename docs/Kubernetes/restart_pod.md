# Restarting / Recreating Kubernetes Pods

1. Display the deployments where the pod is declared that you want to restart.

```console
# kubectl get deployments --namespace jx-live
NAME                                          READY   UP-TO-DATE   AVAILABLE   AGE
jx-company-agencyapi                            1/1     1            1           49d
jx-company-agencycandidatecenterapi             1/1     1            1           49d
jx-company-agencyemailpreferenceapi             1/1     1            1           49d
jx-company-agencyengagementapi                  1/1     1            1           49d
jx-company-agencyengagementboardsapi            1/1     1            1           49d
jx-company-agencynotificationapi                1/1     1            1           49d
jx-company-applicationapi                       1/1     1            1           49d
jx-company-applicationlookupapi                 1/1     1            1           49d
jx-company-backofficeapi                        1/1     1            1           21d
jx-company-commentssocketapi                    1/1     1            1           49d
jx-company-documentapi                          1/1     1            1           48d
jx-company-employerapi                          1/1     1            1           48d
jx-company-employercandidatecenterapi           1/1     1            1           39d
jx-company-employeremailpreferenceapi           1/1     1            1           47d
jx-company-employerengagementapi                1/1     1            1           47d
jx-company-employerengagementboardsapi          1/1     1            1           47d
jx-company-employernotificationapi              1/1     1            1           47d
jx-company-messagequeueapi                      1/1     1            1           46d
jx-company-messengerapi                         1/1     1            1           46d
jx-company-opportunitiesapi                     1/1     1            1           46d
```

2. Change the replicaset to 0 so that the current pod will terminate.

```console
kubectl scale deploy jx-company-agencyapi --namespace jx-live --replicas=0
```

3. Change the replicaset to 1 so that the deployment will create a pod.

```console
kubectl scale deploy jx-company-agencyapi --namespace jx-live --replicas=1
```
