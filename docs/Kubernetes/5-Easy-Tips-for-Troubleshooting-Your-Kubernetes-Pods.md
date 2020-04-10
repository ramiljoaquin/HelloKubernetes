In many cases, you may find that an application in Kubernetes has not been deployed correctly or is not working as it should. This post offers troubleshooting tips to resolve such situations swiftly.


After reading this piece, you’ll also gain insights into the internals of Kubernetes, and as a bonus, I’ll share helpful tips on navigating K8S on your own.


So, let’s begin!.

First, there are two reasons why your Pods can fail.

1. Errors in the configuration of your Kubernetes resources like deployment and services.
2. Problems in your code.


In the former case, containers do not start. In the latter instance, the application code fails after the container starts up. We’ll address each of these situations systematically.


The kubectl command-line utility will be used to interact with K8S throughout this exercise.


# Tip 1: Observe Pods
Verify that the Pods are in status Running or Ready.
```bash
kubectl get pods
```

![get pods](https://miro.medium.com/max/1976/1*_Tig20dggByhH8idr6bu0A.png)


One Pod is in the status Pending for nine hours, that cannot be good! The container did not start, and we’ll investigate this with the describe command in tip number two.


Meanwhile, here are other error codes that occur when a container fails to start.


* ImagePullBackoff— Docker image registry not accessible, image name/version specified in deployment incorrect. Make sure the image name is correct, and the registry is accessible and authenticated (docker login…).
* RunContainerError— One possible cause: ConfigMap/Secrets missing.
* ContainerCreating— Something not available immediately, persistent volume?


Before we investigate other errors, let’s experiment with starting a Pod using an incorrect image name.
```bash
# start Pod from image "ngin". 
# 'web' can be any name, is the name of resulting K8S deployment
kubectl run web --image=ngin --replicas=1
```
![kubectl run web --image=ngin --replicas=1](https://miro.medium.com/max/2256/1*GMjCfJc9XNgWVo5V9kM4yg.png)


Sure enough, the use of the non-existent image “ngin” caused an ImagePullBackoff error. Let’s fix that by using the correct image name “nginx”.

```bash
kubectl run temp --image=nginx --replicas=1
kubectl get pods
```

![kubectl get pods](https://miro.medium.com/max/2092/1*QtLl_SjCcHFzQ2fc1BRRyg.png)

Pod is up


It’s all good now: see above!


Next, here are a few errors that can occur after the container has started.


* CrashLoopBackOff — Pod liveness check has failed or the Docker image is faulty. E.g., The Docker CMD is exiting immediately. See tip number three to check logs. Note: The RESTARTS column in the screenshot shows the number of restarts. In this case, you should expect to see some restarts because K8S attempts to start Pods repeatedly when errors occur.
* If the Pod is in status Running and your app is still not working correctly, proceed to tips three and four.


# Tip 2: Check Events Related to Pods

If you see one of the error codes on the Pod status, you can get more information with the describe command. This is helpful in situations where the container itself did not start.

```bash
kubectl describe frontend-65c58c957d-f4cqn
```

![kubectl describe frontend-65c58c957d-f4cqn](https://miro.medium.com/max/3776/1*mhjICLD23Lwwbd_lbQ1gsw.png)



The last line of the screenshot indicates that the Pod has not started due to the lack of CPU resources, see the message at the bottom. You can increase the Pod’s CPU share and redeploy the application.


# Tip 3: Check Your Logs

Now that the container has started, see if the application is functioning correctly by checking logs. E.g. for Pod frontend-65c58c957d-bzbg2:

```bash
kubectl logs --tail=10 frontend-65c58c957d-bzbg2
```
![kubectl logs --tail=10 frontend-65c58c957d-bzbg2](https://miro.medium.com/max/3184/1*sUZpBkjGx1uQ3G_4ABWJ6A.png)

Steam logs of a running application.

```bash
kubectl logs -f frontend-65c58c957d-bzbg2
```

If the log’s command produces no output, it is possible that get pod was showing a newly-restarted Pod, so check the previous dead Pod.

```bash
kubectl logs frontend-65c58c957d-bzbg2 --previous
```

# Tip 4: Run “sh”, “bash”, or “ash” Directly in the Pods


You can get inside of the Pods and run commands to troubleshoot your application (hit exit to get out of it).
```bash
kubectl exec -it frontend-65c58c957d-bzbg2 /bin/sh
```

# Tip 5: Show Cluster-Level Events

K8S fires events whenever the state of the resources it manages changes (Normal, Warning, etc). They help us understand what happened behind the scenes. The get events command provides an aggregate perspective of events.
```bash
# all events sorted by time. 
kubectl get events --sort-by=.metadata.creationTimestamp
# warnings only
kubectl get events --field-selector type=Warning
# events related to Nodes 
kubectl get events --field-selector involvedObject.kind=Node
```

# A Bonus Tip
This is my favorite tip! Familiarity with commands gives you confidence navigating matters in your K8S cluster.
First, type kubectl to a list of commands.


Next, try the command shown here to grep debug commands.

```bash
kubectl | grep -i -A 10 debugging
```
![kubectl | grep -i -A 10 debugging](https://miro.medium.com/max/2736/1*jQZVTllyKQqBLweMArXPtg.png)

Lists basic commands you can run on K8S resources.
```bash
kubectl | grep -i -A 5 Basic
```
![kubectl | grep -i -A 5 Basic](https://miro.medium.com/max/3788/1*QLtUfskmsNbu_GJQDYL7qw.png)

Next, list Kubernetes resources on which you can operate.
```bash
kubectl api-resources
```
![kubectl api-resources](https://miro.medium.com/max/3556/1*LD6xJpO1NGL4XB9YF8VwTQ.png)
Now make your own command! You can pick a command (get, describe, explain) and run it on one of these resources! E.g., get nodes. Try others!


Although some combinations are not meaningful, other than that, the command system is quite intuitive and consistent; you can easily compose commands and explore.


Just be careful not to delete or modify objects you do not want to touch.
List K8S namespaces:

```bash
kubectl get ns
```
![kubectl ns](https://miro.medium.com/max/1088/1*RHvkr8zKs4KPWOtLBdY8Pg.png)

You can dig down into specific commands to understand your options and examples.
```bash
kubectl get --help
# see K8S system pods in 'kube-system' namespace!
kubectl -n kube-system get pods
```
![kubectl -n kube-system get pods](https://miro.medium.com/max/1572/1*w119d2Brkv_24NiWvflGzg.png)
As you can see, the K8S command system is quite easy to understand and you can learn a lot simply by experimenting with these commands.


# Conclusion
With this, I hope you’ll find yourself comfortable working in a Kubernetes cluster, locating and fixing errors in K8S resources and code. I am hoping to cover debugging of K8S services and networking next.
If you have come this far, I want to thank you for your persistence and dedication to learning Kubernetes. Please leave comments below, let me know if you would like me to cover any specific topics going forward.
Finally, read my last piece if you want to deploy a real-world application in K8S. You can then tinker with it using commands and troubleshooting facilities.
# Further Reading
[Kubernetes debug application](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-application/)
[Learn K8S — Troubleshooting deployments](https://learnk8s.io/troubleshooting-deployments)

WRITTEN BY

[Ram Rai](https://medium.com/@dancerincloud?source=follow_footer--------------------------follow_footer-)

