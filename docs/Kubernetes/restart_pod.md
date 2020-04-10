# Restarting / Recreating Kubernetes Pods

1. Display the deployments where the pod is declared that you want to restart.
```console
# kubectl get deployments --namespace jx-live
NAME                                          READY   UP-TO-DATE   AVAILABLE   AGE
jx-ubidy-agencyapi                            1/1     1            1           49d
jx-ubidy-agencycandidatecenterapi             1/1     1            1           49d
jx-ubidy-agencyemailpreferenceapi             1/1     1            1           49d
jx-ubidy-agencyengagementapi                  1/1     1            1           49d
jx-ubidy-agencyengagementboardsapi            1/1     1            1           49d
jx-ubidy-agencynotificationapi                1/1     1            1           49d
jx-ubidy-applicationapi                       1/1     1            1           49d
jx-ubidy-applicationlookupapi                 1/1     1            1           49d
jx-ubidy-backofficeapi                        1/1     1            1           21d
jx-ubidy-commentssocketapi                    1/1     1            1           49d
jx-ubidy-documentapi                          1/1     1            1           48d
jx-ubidy-employerapi                          1/1     1            1           48d
jx-ubidy-employercandidatecenterapi           1/1     1            1           39d
jx-ubidy-employeremailpreferenceapi           1/1     1            1           47d
jx-ubidy-employerengagementapi                1/1     1            1           47d
jx-ubidy-employerengagementboardsapi          1/1     1            1           47d
jx-ubidy-employernotificationapi              1/1     1            1           47d
jx-ubidy-messagequeueapi                      1/1     1            1           46d
jx-ubidy-messengerapi                         1/1     1            1           46d
jx-ubidy-opportunitiesapi                     1/1     1            1           46d
```
2. Change the replicaset to 0 so that the current pod will terminate.
```console
kubectl scale deploy jx-ubidy-agencyapi --namespace jx-live --replicas=0
```
3. Change the replicaset to 1 so that the deployment will create a pod.
```console
kubectl scale deploy jx-ubidy-agencyapi --namespace jx-live --replicas=1
```
